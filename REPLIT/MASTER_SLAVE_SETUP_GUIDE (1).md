# PlantGuard MASTER ↔ SLAVE Communication Setup Guide

## 📡 Overview

Your PlantGuard system now supports **two-way LoRa communication** between nodes:

- **MASTER Node:** WiFi/MQTT gateway that receives LoRa packets and forwards to cloud
- **SLAVE Node:** Battery-powered sensor that reads RS485 data and transmits via LoRa

---

## 🎯 Three Firmware Options

### Option 1: Separate Firmwares (Recommended for beginners)
- `ESP32_PlantGuard_MASTER.ino` - For stand-alone gateway node
- `ESP32_PlantGuard_SLAVE.ino` - For sensor node with RS485

**Pros:** Clear, dedicated purpose for each node
**Cons:** Need to upload different firmware to switch roles

### Option 2: Unified Firmware (Flexible)
- `ESP32_PlantGuard_UNIFIED.ino` - Single firmware, configure role at top

**Pros:** One firmware for all nodes, easy to switch roles
**Cons:** Slightly larger code size

### Option 3: Test First (Diagnostic)
- `ESP32_MASTER_DisplayTest.ino` - Simple display test
- Upload this first to verify hardware works

---

## 🔧 Setup Instructions

### Step 1: Choose Your Nodes

**Your Setup (as discussed):**
- **Node 1:** Stand-alone ESP32 → **MASTER** (WiFi gateway)
- **Node 2:** ESP32 with RS485 sensor → **SLAVE** (field sensor)

---

### Step 2A: Setup MASTER Node (Gateway)

**Using dedicated firmware:**

1. **Open:** `ESP32_PlantGuard_MASTER.ino` in Arduino IDE

2. **Configure at top of file:**
   ```cpp
   const char* NODE_ID = "MASTER_001";  // ← Your gateway ID
   #define LORA_FREQUENCY 915.0          // US: 915, EU: 868, Asia: 433
   ```

3. **Upload to ESP32**
   - Select Board: "WiFi LoRa 32(V3)"
   - Upload

4. **First Boot:**
   - Display shows: "WiFi Config Mode"
   - Connect phone to WiFi: "PlantGuard-MASTER_001"
   - Browser opens → Select your WiFi network
   - Enter password and Save

5. **After WiFi connects:**
   - Display rotates through 4 pages:
     - Page 1: MASTER status (WiFi, MQTT, battery, packets)
     - Page 2: LoRa gateway stats (RX/Forwarded/Failed)
     - Page 3: Signal quality (RSSI/SNR from last packet)
     - Page 4: Local sensor data (if RS485 connected)

6. **Button Controls:**
   - Quick press: Wake display, scroll pages
   - Hold 3 sec: Reset WiFi
   - Hold 10 sec: Shutdown

---

### Step 2B: Setup SLAVE Node (Sensor)

**Using dedicated firmware:**

1. **Open:** `ESP32_PlantGuard_SLAVE.ino` in Arduino IDE

2. **Configure at top of file:**
   ```cpp
   const char* NODE_ID = "SLAVE_001";    // ← Your sensor ID
   #define LORA_FREQUENCY 915.0           // MUST MATCH MASTER!
   ```

3. **Upload to ESP32 with RS485 sensor connected**

4. **First Boot - WiFi Configuration:**
   - Display shows: "WiFi Config Mode"
   - Connect phone to WiFi: "PlantGuard-SLAVE_001"
   - Browser opens automatically (or go to: 192.168.4.1)
   - Select your WiFi network and enter password
   - Click Save

5. **After WiFi Connects:**
   - SLAVE connects to MQTT for configuration
   - Display shows node status and sensor readings

6. **Display shows:**
   - Node ID with WiFi/MQTT status (✓ or ✗)
   - LoRa TX stats (packets sent / failed)
   - Battery percentage and deep sleep status
   - Update interval and RSSI

7. **Button Controls:**
   - Quick press: Wake display, force sensor read
   - Hold 3 sec: **Reset WiFi** settings and restart
   - Hold 10 sec: Shutdown

8. **Remote Configuration:**
   - Use dashboard to configure update intervals
   - Enable/disable deep sleep remotely
   - Adjust LoRa settings in real-time
   - All settings pushed via MQTT instantly!

---

### Step 2C: Using Unified Firmware (Alternative)

**If you prefer one firmware for both:**

1. **Open:** `ESP32_PlantGuard_UNIFIED.ino`

2. **For MASTER node, set:**
   ```cpp
   #define NODE_ROLE "MASTER"
   const char* NODE_ID = "MASTER_001";
   ```

3. **For SLAVE node, set:**
   ```cpp
   #define NODE_ROLE "SLAVE"
   const char* NODE_ID = "SLAVE_001";
   ```

4. **Upload to respective ESP32 boards**

5. **To switch roles later:**
   - Just change `NODE_ROLE` and re-upload
   - No need to load different firmware!

---

## 📊 How Communication Works

### Data Flow:

```
SLAVE Node                    MASTER Node                Dashboard
┌─────────────┐              ┌─────────────┐           ┌─────────────┐
│ Read RS485  │              │             │           │             │
│ Sensors     │              │   Receive   │           │   Display   │
│             │   LoRa       │   LoRa      │   MQTT    │   Sensor    │
│ Temperature ├──────────────>   Packet    ├──────────>   Data       │
│ Moisture    │  915 MHz     │             │           │             │
│ pH, NPK     │              │   Forward   │           │   Generate  │
│ EC          │              │   to MQTT   │           │   Alerts    │
│             │              │             │           │             │
│ Deep Sleep  │              │ WiFi/Cloud  │           │  Web UI     │
└─────────────┘              └─────────────┘           └─────────────┘
```

### SLAVE Transmits:
```json
{
  "nodeId": "SLAVE_001",
  "temperature": 25.5,
  "moisture": 65.2,
  "pH": 6.8,
  "nitrogen": 120,
  "phosphorus": 45,
  "potassium": 180,
  "ec": 850,
  "batteryVoltage": 4.05,
  "batteryPercent": 89
}
```

### MASTER Receives:
- Parses JSON from LoRa packet
- Extracts `nodeId` to identify which SLAVE sent it
- Publishes to MQTT: `plantguard/sensor/SLAVE_001`
- Dashboard receives and displays data

---

## ⚙️ Configuration from Dashboard (Real-time!)

Both MASTER and SLAVE nodes support **remote configuration via MQTT** from the dashboard. Changes are applied instantly without reuploading firmware!

### Power Settings (Both MASTER & SLAVE):
Configure from **ESP32 Raw Data** tab → Click **Power Settings** button:

- **Update Interval:** How often to read/transmit sensors (1-3600 seconds)
  - MASTER default: 5 seconds
  - SLAVE default: 60 seconds (battery saving)
  
- **Deep Sleep:** Enable/disable deep sleep mode (SLAVE only)
  - When enabled, SLAVE wakes → reads → transmits → sleeps
  - Saves massive battery life!
  
- **Deep Sleep Duration:** How long to sleep between wake cycles (60-86400 seconds)
  - Default: 300 seconds (5 minutes)
  - Longer sleep = longer battery life
  
- **OLED Timeout:** Display auto-off timer (10-3600 seconds)
  - Default: 300 seconds (5 minutes) on both nodes
  - Display wakes when PRG button pressed

### LoRa Settings (Both MASTER & SLAVE):
Configure from **ESP32 Raw Data** tab → Click **LoRa Settings** button:

- **Frequency:** 433/868/915 MHz
  - US: 915.0 MHz (default)
  - EU: 868.0 MHz
  - Asia: 433.0 MHz
  - **Both nodes MUST use same frequency!**
  
- **Bandwidth:** 125/250/500 kHz
  - Lower = longer range, slower
  - Higher = shorter range, faster
  
- **Spreading Factor:** 6-12
  - Higher = longer range, more reliable, slower
  - Lower = shorter range, faster
  
- **TX Power:** 2-20 dBm
  - Higher = longer range, more battery drain
  - Lower = save power

**All changes pushed via MQTT in real-time!** Just click Save in dashboard and watch ESP32 Serial Monitor to see configuration applied instantly.

---

## 🔍 Troubleshooting

### SLAVE not transmitting?

1. **Check Serial Monitor:**
   ```
   📡 Reading sensors...
   📤 Sending LoRa packet...
   ✓ Packet sent!
   ```

2. **Verify LoRa frequency matches:**
   - MASTER: `#define LORA_FREQUENCY 915.0`
   - SLAVE: `#define LORA_FREQUENCY 915.0`
   - **MUST BE IDENTICAL!**

3. **Check distance:**
   - LoRa range: ~2km line-of-sight
   - Indoors: ~500m depending on walls
   - Start with nodes close together for testing

4. **Battery level:**
   - Low battery = weak transmission
   - Check voltage on display

### MASTER not receiving?

1. **Check Serial Monitor:**
   ```
   📥 LoRa RX [RSSI:-45 SNR:9.5]: {"nodeId":"SLAVE_001"...}
   ✓ Published to MQTT: plantguard/sensor/SLAVE_001
   ```

2. **Verify LoRa is in RX mode:**
   - MASTER should show "LoRa init success!"
   - Mode: RX (Gateway)

3. **Check MQTT connection:**
   - Display should show "MQTT: ✓"
   - If "MQTT: ✗", check WiFi or restart

### Weak signal (RSSI/SNR)?

**RSSI (Signal Strength):**
- **-30 to -50 dBm:** Excellent
- **-50 to -80 dBm:** Good
- **-80 to -100 dBm:** Weak (may have packet loss)
- **Below -100 dBm:** Very weak (unreliable)

**SNR (Signal-to-Noise Ratio):**
- **> 10 dB:** Excellent
- **5-10 dB:** Good
- **0-5 dB:** Fair
- **< 0 dB:** Poor (high packet loss)

**Improve signal:**
1. Reduce distance between nodes
2. Increase TX power (dashboard or firmware)
3. Increase spreading factor (slower but more reliable)
4. Add antenna or use external antenna
5. Move nodes away from metal objects

---

## 🔋 Battery Life Optimization

### SLAVE Node (Sensor):

**Without Deep Sleep:**
- Update every 60 seconds
- OLED sleeps after 30 seconds
- **Battery life:** 2-3 days

**With Deep Sleep:**
- Wake every 5 minutes (configurable)
- Read sensors + transmit + sleep
- OLED off during sleep
- **Battery life:** 2-4 weeks

**Best Practices:**
1. Enable deep sleep (hold button 3 seconds)
2. Set update interval to 5-10 minutes via dashboard
3. Use external antenna for better signal
4. Keep OLED timeout short (30 seconds)

### MASTER Node (Gateway):

**Powered by USB:**
- No battery concerns
- Can stay on 24/7

**Battery Powered (optional):**
- Disable OLED sleep (needs to show status)
- Use light sleep between operations
- **Battery life:** 1-2 days continuous

---

## 📈 Monitoring on Dashboard

### Check ESP32 Raw Data Tab:
- Shows latest sensor readings from all nodes
- MASTER_001 shows gateway stats
- SLAVE_001 shows sensor data

### Check Node-RED Tab:
- Real-time sensor graphs
- Alert notifications
- Moving averages

### Check Mobile PWA:
- Optimized for phone viewing
- Large cards with key metrics
- Auto-refresh every 5 seconds

---

## ✅ Success Checklist

- [ ] MASTER node connected to WiFi
- [ ] MASTER node connected to MQTT (shows "✓" on display)
- [ ] MASTER display rotates through 4 pages
- [ ] SLAVE node reads RS485 sensors successfully
- [ ] SLAVE node transmits LoRa packets (shows "✓ Packet sent!")
- [ ] MASTER receives LoRa packets (counter increments)
- [ ] MASTER forwards to MQTT (packets forwarded counter increments)
- [ ] Dashboard shows sensor data from SLAVE node
- [ ] RSSI/SNR values are good (>-80 dBm, >5 dB)
- [ ] Deep sleep works on SLAVE (if enabled)
- [ ] Configuration from dashboard updates ESP32 (test interval change)

---

## 🚀 Next Steps

### Add More SLAVE Nodes:
1. Clone SLAVE firmware
2. Change `NODE_ID` to "SLAVE_002", "SLAVE_003", etc.
3. Upload to new ESP32 boards
4. Each SLAVE automatically appears on dashboard

### Customize Sensor Readings:
- Edit RS485 Modbus requests in firmware
- Add custom sensors (I2C, analog, etc.)
- Modify JSON payload in `sendLoRaData()` function

### Extend Range:
- Use external antennas (u.FL connector on board)
- Increase spreading factor (slower but longer range)
- Position MASTER node higher up

### Monitor in Real-Time:
- Open Serial Monitor (115200 baud) on both nodes
- Watch packet transmission/reception
- Debug signal quality issues

---

**Enjoy your PlantGuard LoRa sensor network!** 🌱📡

**Last Updated:** November 3, 2025
