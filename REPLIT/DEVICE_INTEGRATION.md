# PlantGuard Device Integration Guide

This guide explains how to integrate ESP32 microcontrollers with LoRa communication to your PlantGuard monitoring system.

## System Architecture

**Sensor Nodes → LoRa → Gateway → Internet → PlantGuard App**

1. **ESP32 Sensor Nodes**: Remote plant monitoring with LoRa transmission
2. **Local Gateway**: Raspberry Pi or ESP32 that receives LoRa data and forwards to PlantGuard
3. **PlantGuard App**: Web dashboard for real-time monitoring and alerts

## Hardware Requirements

### ESP32 Sensor Node (Per Plant)
- **Microcontroller**: ESP32-S3 or ESP32-C3 ($5-8)
- **LoRa Module**: SX1276/SX1278 (915MHz US / 868MHz EU) ($8-12)
- **Power**: 18650 Li-ion battery (3000-5000mAh) ($5-8)
- **Sensors**: pH, NPK, Light (LDR), Soil Moisture, DHT22 (Humidity/Temperature) ($10-15)
- **Case**: Weatherproof enclosure for outdoor use ($5)
- **Total per node**: ~$35-50

### Local Gateway Hub
- **Option 1**: Raspberry Pi 4 + LoRa Hat/Module (~$50-70)
- **Option 2**: ESP32 + Ethernet + LoRa Module (~$25-35)
- **Internet**: Ethernet or WiFi connection
- **Power**: Wall adapter (continuous operation)

### Communication Range
- **LoRa Range**: 2km+ rural areas, 200m+ urban environments
- **Power Consumption**: µA sleep mode for months of battery life
- **Frequency**: 915MHz (US), 868MHz (EU), 433MHz (Asia)

## API Endpoints

### Register a New Sensor Node

**POST** `/api/sensor-nodes`

Request body:
```json
{
  "name": "ESP32-Mint-01",
  "plantType": "Peppermint", 
  "location": "Greenhouse A - Section 1",
  "connectivity": "LoRa",
  "isActive": true,
  "updateInterval": 300
}
```

Response:
```json
{
  "id": "uuid-string",
  "name": "ESP32-Mint-01",
  "plantType": "Peppermint",
  "location": "Greenhouse A - Section 1",
  "connectivity": "LoRa",
  "isActive": true,
  "batteryLevel": 100,
  "updateInterval": 300,
  "lastSeen": "2024-01-15T10:30:00Z",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

### Send Sensor Data (Gateway to App)

**POST** `/api/sensor-readings`

Request body:
```json
{
  "nodeId": "your-sensor-node-id",
  "pH": 6.8,
  "npk": 150.5,
  "lightIntensity": 850.2,
  "soilMoisture": 65.3,
  "humidity": 58.7,
  "temperature": 24.1,
  "batteryLevel": 89
}
```

## ESP32 Sensor Node Code

### Libraries Required
```cpp
#include <WiFi.h>          // For WiFi backup
#include <SPI.h>           // For LoRa communication
#include <LoRa.h>          // LoRa library
#include <ArduinoJson.h>   // JSON formatting
#include <DHT.h>           // Temperature/Humidity sensor
```

### Complete ESP32 Sensor Node Code
```cpp
#include <SPI.h>
#include <LoRa.h>
#include <ArduinoJson.h>
#include <DHT.h>
#include <esp_sleep.h>

// LoRa pins for ESP32
#define SCK 5
#define MISO 19
#define MOSI 27
#define SS 18
#define RST 14
#define DIO0 26

// Sensor pins
#define PH_PIN A0
#define NPK_PIN A1
#define LIGHT_PIN A2
#define MOISTURE_PIN A3
#define BATTERY_PIN A4
#define DHT_PIN 4
#define DHT_TYPE DHT22

// Configuration
const char* nodeId = "ESP32-Mint-01";
int updateInterval = 300; // 5 minutes default
DHT dht(DHT_PIN, DHT_TYPE);

void setup() {
  Serial.begin(115200);
  
  // Initialize LoRa
  SPI.begin(SCK, MISO, MOSI, SS);
  LoRa.setPins(SS, RST, DIO0);
  
  if (!LoRa.begin(915E6)) { // 915MHz for US
    Serial.println("Starting LoRa failed!");
    while (1);
  }
  
  LoRa.setSpreadingFactor(7);
  LoRa.setSignalBandwidth(125E3);
  LoRa.setCodingRate4(5);
  
  dht.begin();
  Serial.println("Sensor node started");
}

void loop() {
  // Read all sensors
  float ph = readPH();
  float npk = readNPK();
  float light = readLight();
  float moisture = readSoilMoisture();
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();
  int battery = readBatteryLevel();
  
  // Create JSON payload
  DynamicJsonDocument doc(512);
  doc["nodeId"] = nodeId;
  doc["pH"] = ph;
  doc["npk"] = npk;
  doc["lightIntensity"] = light;
  doc["soilMoisture"] = moisture;
  doc["humidity"] = humidity;
  doc["temperature"] = temperature;
  doc["batteryLevel"] = battery;
  
  String jsonString;
  serializeJson(doc, jsonString);
  
  // Send via LoRa
  LoRa.beginPacket();
  LoRa.print(jsonString);
  LoRa.endPacket();
  
  Serial.println("✓ Data sent via LoRa: " + jsonString);
  
  // Enter deep sleep to save battery
  Serial.println("Sleeping for " + String(updateInterval) + " seconds...");
  esp_sleep_enable_timer_wakeup(updateInterval * 1000000); // microseconds
  esp_deep_sleep_start();
}

// Sensor reading functions
float readPH() {
  int sensorValue = analogRead(PH_PIN);
  return map(sensorValue, 0, 4095, 0, 14); // Convert to pH scale
}

float readNPK() {
  int sensorValue = analogRead(NPK_PIN);
  return map(sensorValue, 0, 4095, 0, 300); // Convert to NPK ppm
}

float readLight() {
  int sensorValue = analogRead(LIGHT_PIN);
  return map(sensorValue, 0, 4095, 0, 2000); // Convert to lux
}

float readSoilMoisture() {
  int sensorValue = analogRead(MOISTURE_PIN);
  return map(sensorValue, 0, 4095, 0, 100); // Convert to percentage
}

int readBatteryLevel() {
  int sensorValue = analogRead(BATTERY_PIN);
  float voltage = (sensorValue / 4095.0) * 3.3 * 2; // Voltage divider
  return map(voltage * 100, 300, 420, 0, 100); // 3.0V-4.2V to 0-100%
}
```

## Local Gateway Code (Raspberry Pi)

### Python Gateway Script
```python
import json
import time
import requests
import serial
from LoRa import LoRa

class LoRaGateway:
    def __init__(self):
        self.lora = LoRa()
        self.server_url = "http://your-plantguard-app.com/api/sensor-readings"
        
    def setup_lora(self):
        self.lora.set_mode(1)  # LoRa mode
        self.lora.set_pa_config(pa_select=1)
        self.lora.set_spreading_factor(7)
        self.lora.set_bandwidth(7)  # 125kHz
        self.lora.set_coding_rate(1)  # 4/5
        
    def listen_for_data(self):
        while True:
            if self.lora.received_packet():
                payload = self.lora.read_payload()
                try:
                    # Parse JSON from LoRa
                    sensor_data = json.loads(payload.decode())
                    print(f"Received: {sensor_data}")
                    
                    # Forward to PlantGuard app
                    response = requests.post(
                        self.server_url,
                        json=sensor_data,
                        headers={'Content-Type': 'application/json'}
                    )
                    
                    if response.status_code == 200:
                        print("✓ Data forwarded to PlantGuard")
                    else:
                        print(f"✗ Failed to forward: {response.status_code}")
                        
                except Exception as e:
                    print(f"Error processing data: {e}")
                    
            time.sleep(0.1)

if __name__ == "__main__":
    gateway = LoRaGateway()
    gateway.setup_lora()
    print("LoRa Gateway started - listening for sensor data...")
    gateway.listen_for_data()
```

## GPIO Pin Connections

### ESP32 Sensor Node Wiring:
```
LoRa SX1276/SX1278:
- VCC → 3.3V
- GND → GND
- SCK → GPIO5
- MISO → GPIO19
- MOSI → GPIO27
- NSS → GPIO18
- RST → GPIO14
- DIO0 → GPIO26

Sensors:
- pH Sensor → A0 (GPIO36)
- NPK Sensor → A1 (GPIO39)
- Light Sensor (LDR) → A2 (GPIO34)
- Soil Moisture → A3 (GPIO35)
- Battery Monitor → A4 (GPIO32)
- DHT22 Data → GPIO4
- DHT22 VCC → 3.3V
- DHT22 GND → GND
```

## Update Intervals & Battery Life

### Available Intervals in PlantGuard:
- **1 second**: Real-time demos (hours of battery)
- **15 seconds**: Live monitoring (1-2 days)
- **1 minute**: Frequent updates (1 week)
- **5 minutes**: Regular monitoring (1 month)
- **30 minutes**: Periodic checks (3 months)
- **1 hour**: Long-term monitoring (6 months)
- **12 hours**: Daily checks (1+ years)
- **24 hours**: Weekly monitoring (2+ years)

### Power Optimization Tips:
1. **Deep Sleep**: ESP32 uses only 10µA in deep sleep
2. **LoRa Efficiency**: Much lower power than WiFi
3. **Sensor Power**: Turn off sensors between readings
4. **Battery Choice**: 18650 Li-ion for longest life
5. **Solar Option**: Small solar panel for infinite operation

## Sensor Value Ranges for Optimal Plant Health

### Peppermint/Spearmint Plants:
- **pH**: 6.0-7.5 (optimal), 5.0-8.5 (acceptable)
- **NPK**: 120-200 ppm (optimal), 80-300 ppm (acceptable)
- **Light**: 600-1200 lux (optimal), 300-2000 lux (acceptable)
- **Soil Moisture**: 40-80% (optimal), 20-90% (acceptable)
- **Humidity**: 40-70% (optimal), 30-80% (acceptable)
- **Temperature**: 18-30°C (optimal), 10-40°C (acceptable)
- **Battery**: 30%+ (warning), 15%+ (critical)

## Getting Started

1. **Register your first sensor** in the PlantGuard dashboard
2. **Build your ESP32 sensor node** with the code above
3. **Set up a local gateway** (Raspberry Pi recommended)
4. **Configure update intervals** based on your battery life needs
5. **Monitor your plants** in real-time through the web dashboard

Your PlantGuard system will automatically detect when sensors go offline, generate alerts for critical conditions, and provide Excel exports of all your plant data for analysis!