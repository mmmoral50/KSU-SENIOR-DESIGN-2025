# ESP32 PlantGuard MASTER Node Setup Guide

## Overview
The MASTER node acts as a WiFi/MQTT gateway that receives LoRa packets from SLAVE nodes and forwards them to your PlantGuard dashboard.

## Hardware Required
- **Heltec WiFi LoRa 32 V3.2** board
- **USB-C cable** for programming and power
- **Battery** (optional) - 3.7V LiPo for portable operation
- **RS485 Sensor** (optional) - COMWIN 7-in-1 soil sensor for local monitoring

## Firmware: ESP32_PlantGuard_MASTER.ino

### Key Features
✅ **WiFi Manager** - Easy network configuration via captive portal  
✅ **MQTT Gateway** - Automatic packet forwarding to dashboard  
✅ **LoRa Receiver** - Listens for SLAVE node transmissions  
✅ **Optional Local Sensor** - Can monitor its own location too  
✅ **4-Page OLED Display** - Auto-rotating status screens  
✅ **Signal Quality** - RSSI/SNR monitoring  
✅ **Power Management** - OLED auto-sleep, configurable timeouts  

---

## Step 1: Configure the Firmware

### Open ESP32_PlantGuard_MASTER.ino

Find these settings near the top of the file:

```cpp
/*********************** Configuration ************************/
const char* MQTT_SERVER = "test.mosquitto.org";    // ← Your MQTT broker
const int MQTT_PORT = 1883;
const char* MQTT_USER = "";                        // ← MQTT username (if needed)
const char* MQTT_PASSWORD = "";                    // ← MQTT password (if needed)
const char* NODE_ID = "MASTER_001";                // ← CHANGE THIS for each MASTER

// LoRa Configuration (must match SLAVE nodes)
#define LORA_FREQUENCY 868.0    // EU: 868.0, US: 915.0, Asia: 433.0
#define LORA_BANDWIDTH 125.0
#define LORA_SPREADING_FACTOR 7
```

### Important Settings:
1. **NODE_ID**: Set to `"MASTER_001"` (or `"MASTER_002"`, etc. if you have multiple)
2. **LORA_FREQUENCY**: 
   - Europe: `868.0`
   - USA: `915.0`
   - Asia: `433.0`
3. **LoRa settings MUST match your SLAVE nodes** (frequency, bandwidth, spreading factor)

---

## Step 2: Upload Firmware

### Using Arduino IDE:
1. **Install Libraries** (if not already installed):
   - Heltec ESP32 LoRa v3 (by ropg)
   - WiFiManager (by tzapu)
   - PubSubClient
   - ArduinoJson

2. **Board Settings**:
   - Board: "Heltec WiFi LoRa 32(V3) / Wireless shell(V3) / Wireless stick lite (V3)"
   - Upload Speed: 921600
   - CPU Frequency: 240MHz
   - Flash Frequency: 80MHz
   - Partition Scheme: "Default 4MB with spiffs"

3. **Upload**:
   - Connect via USB-C
   - Select correct COM port
   - Click Upload

---

## Step 3: First-Time WiFi Setup

### On First Boot:
1. **OLED displays**: "WiFi Config Mode" with AP name
2. The MASTER creates a WiFi access point named: **PlantGuard-MASTER_001**

### Connect from Phone/Computer:
1. Look for WiFi network: `PlantGuard-MASTER_001`
2. Connect to it (no password needed)
3. **Captive portal should auto-open**
   - If not, browse to: `http://192.168.4.1`
4. Select your WiFi network from the list
5. Enter password
6. Click "Save"
7. MASTER will restart and connect to your WiFi

---

## Step 4: Verify Operation

### Serial Monitor Output (115200 baud):
```
🌱 PlantGuard MASTER Node
============================
Initializing LoRa...
LoRa init success!
Frequency: 868.0 MHz, SF: 7
Starting WiFi Manager...
WiFi connected!
IP: 192.168.1.123
MQTT connecting...connected!
Subscribed to: esp32/MASTER_001/config
```

### OLED Display Pages (auto-rotate every 15 seconds):

**Page 1: MASTER Status**
```
MASTER: MASTER_001
WiFi: YourNetworkName
MQTT: ✓
Bat: 85% (3.95V)
Fwd: 0 pkts
```

**Page 2: Gateway Statistics**
```
LoRa Gateway Stats
RX: 0 pkts
Fwd: 0
Fail: 0
Last: 
```

**Page 3: Signal Quality**
```
Last Signal Quality
From: SLAVE_001
RSSI: -65 dBm
SNR: 8.5 dB
Quality: Good
```

**Page 4: Local Sensor** (if RS485 connected)
```
Local Sensor Data
Temp: 21.5°C
Moist: 45.2%
pH: 6.8
EC: 320 µS/cm
```

---

## Step 5: Test with SLAVE Nodes

Once MASTER is running:
1. Power on a SLAVE node
2. SLAVE will transmit sensor data via LoRa
3. MASTER receives the packet
4. **Serial Monitor shows**:
   ```
   ═════════════════════════════
   📡 LoRa RX: {"nodeId":"SLAVE_001","temperature":21.5,...}
   📊 RSSI: -68 dBm, SNR: 7.2 dB
   📦 Total received: 1
   ✓ Forwarded to MQTT (total: 1)
   ═════════════════════════════
   ```
5. **OLED updates** with packet count and signal quality
6. **Dashboard shows data** from SLAVE_001

---

## Button Controls

### PROGRAM/BOOT Button (GPIO 0):
- **Quick press (<1s)**: Wake OLED + scroll to next page
- **Hold 3 seconds**: Reset WiFi settings (reconfigure)
- **Hold 10 seconds**: Deep sleep shutdown (power save)

---

## Optional: Connect Local RS485 Sensor

If you want MASTER to also monitor its own location:

### Wiring:
```
COMWIN Sensor → MASTER Board
─────────────────────────────
Red    (VCC)  → 5V
Black  (GND)  → GND
Yellow (A)    → GPIO 35 (RS485_TX)
Blue   (B)    → GPIO 33 (RS485_RX)
```

- **GPIO 38** is RS485_EN (direction control)
- MASTER will read sensor every 5 seconds (configurable)
- Local data published to: `esp32/MASTER_001/sensors`

---

## Dashboard Integration

### The MASTER automatically:
1. **Receives** LoRa packets from SLAVE nodes
2. **Parses** JSON sensor data
3. **Publishes** to MQTT topic: `esp32/{SLAVE_ID}/sensors`
4. **Dashboard** displays data from all nodes

### In Dashboard:
- MASTER appears as: **MASTER_001**
- SLAVE nodes appear as: **SLAVE_001**, **SLAVE_002**, etc.
- Signal quality visible in logs
- Battery levels tracked for all nodes

---

## Power Management

### Configurable from Dashboard:
- **Update Interval**: How often to poll local sensor (default: 5 seconds)
- **OLED Sleep Timeout**: Auto-sleep display (default: 5 minutes)

### MQTT Configuration:
Dashboard sends config to: `esp32/MASTER_001/config`
```json
{
  "updateInterval": 10,
  "oledSleepTimeout": 300
}
```

---

## Troubleshooting

### WiFi Won't Connect:
- Hold button 3 seconds to reset WiFi
- Reconfigure via captive portal
- Check WiFi signal strength

### MQTT Not Connected:
- Verify MQTT_SERVER address
- Check internet connection
- Test with MQTT client (MQTT Explorer)

### No LoRa Packets Received:
- **Check LoRa frequency** matches SLAVE nodes
- **Verify spreading factor** (must match)
- **Check antenna** is connected
- **Reduce distance** between nodes for testing

### Display Blank:
- Press button to wake OLED
- Check OLED timeout setting
- Verify Vext power control

---

## Next Steps

1. **Deploy MASTER** in central location with WiFi coverage
2. **Deploy SLAVE nodes** in field/greenhouse
3. **Monitor dashboard** for incoming data
4. **Check signal quality** (RSSI/SNR) on Page 3
5. **Optimize placement** for best signal strength

---

## LoRa Range Tips

✅ **Good**: RSSI > -80 dBm, SNR > 5 dB  
⚠️ **Weak**: RSSI -80 to -100 dBm, SNR 0-5 dB  
❌ **Poor**: RSSI < -100 dBm, SNR < 0 dB  

**To improve range:**
- Mount MASTER antenna vertically
- Keep antennas away from metal objects
- Increase spreading factor (7→9 for longer range, slower)
- Reduce bandwidth (125→62.5 for longer range)
- Position MASTER at highest point

---

## Example Network Setup

```
              Internet
                  ↓
           ╔══════════════╗
           ║  MASTER_001  ║  ← WiFi + MQTT Gateway
           ║  (Greenhouse)║     Receives & Forwards
           ╚══════════════╝
                  ↑
           LoRa (868 MHz)
        ↗         ↑         ↖
       ↙          |          ↘
SLAVE_001    SLAVE_002    SLAVE_003
(Field A)    (Field B)    (Field C)
   📡           📡           📡
  RS485        RS485        RS485
```

Each SLAVE transmits via LoRa → MASTER forwards to MQTT → Dashboard displays all data

---

## Support

For issues or questions:
- Check Serial Monitor output (115200 baud)
- Verify LoRa settings match between nodes
- Test MQTT connection separately
- Check dashboard logs for received data

**Happy Monitoring!** 🌱📡
