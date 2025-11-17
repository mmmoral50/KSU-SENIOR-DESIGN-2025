# PlantGuard Remote Configuration Guide

## 🎯 Overview

Your PlantGuard system now supports **complete remote configuration** from the web dashboard! Both MASTER and SLAVE nodes can be configured in real-time via MQTT without re-uploading firmware.

---

## 🔧 What Can Be Configured Remotely?

### ⚡ Power Settings

Configure from dashboard **ESP32 Raw Data** tab → **Power Settings** button:

| Setting | MASTER | SLAVE | Range | Default |
|---------|--------|-------|-------|---------|
| **Update Interval** | ✅ | ✅ | 1-3600 sec | MASTER: 5s<br>SLAVE: 60s |
| **Deep Sleep Enable** | ❌ | ✅ | ON/OFF | OFF |
| **Deep Sleep Duration** | ❌ | ✅ | 60-86400 sec | 300s (5 min) |
| **OLED Sleep Timeout** | ✅ | ✅ | 10-3600 sec | 300s (5 min) |

**Update Interval** = How often nodes read sensors and transmit/update
**Deep Sleep** = SLAVE enters deep sleep between readings (huge battery savings!)
**OLED Timeout** = Display turns off after this period, wakes on PRG button press

---

### 📡 LoRa Settings

Configure from dashboard **ESP32 Raw Data** tab → **LoRa Settings** button:

| Setting | MASTER | SLAVE | Options | Default |
|---------|--------|-------|---------|---------|
| **Frequency** | ✅ | ✅ | 410-930 MHz | 915.0 MHz (US) |
| **Bandwidth** | ✅ | ✅ | 125/250/500 kHz | 125 kHz |
| **Spreading Factor** | ✅ | ✅ | 6-12 | 7 |
| **TX Power** | ✅ | ✅ | 2-20 dBm | 14 dBm |

**⚠️ Important:** MASTER and SLAVE must use the **same LoRa frequency** to communicate!

**Regional Presets:**
- **US:** 915.0 MHz
- **EU:** 868.0 MHz
- **Asia:** 433.0 MHz

**Bandwidth:**
- **125 kHz** = Longer range, slower speed (recommended)
- **250 kHz** = Medium range, medium speed
- **500 kHz** = Shorter range, faster speed

**Spreading Factor:**
- **Higher (9-12)** = Longer range, more reliable, slower
- **Lower (6-8)** = Shorter range, faster, less reliable

**TX Power:**
- **Higher (17-20 dBm)** = Longer range, more battery drain
- **Lower (10-14 dBm)** = Shorter range, saves battery

---

## 📱 How to Configure from Dashboard

### Step-by-Step:

1. **Open PlantGuard Dashboard** in your browser

2. **Go to "ESP32 Raw Data" tab**

3. **Select your node** from the dropdown (e.g., MASTER_001 or SLAVE_001)

4. **Click "Power Settings" button:**
   - Adjust update interval (how often data is sent)
   - Enable/disable deep sleep (SLAVE only)
   - Set deep sleep duration
   - Set OLED sleep timeout
   - Click **Save**

5. **Click "LoRa Settings" button:**
   - Select frequency preset (US/EU/Asia) or custom
   - Adjust bandwidth (125/250/500 kHz)
   - Set spreading factor (6-12)
   - Set TX power (2-20 dBm)
   - Click **Save**

6. **Watch ESP32 Serial Monitor:**
   ```
   📩 MQTT message on topic: plantguard/SLAVE_001/config
   📝 Received power configuration:
     Update interval: 120 seconds
     Deep sleep: ENABLED
     Deep sleep interval: 600 seconds
     OLED sleep timeout: 300 seconds
   ```

7. **Configuration applied instantly!** No firmware re-upload needed.

---

## 🎮 ESP32 Button Controls

### MASTER Node:
- **Quick press (<1s):** Wake display & scroll to next page
- **Hold 3 seconds:** Reset WiFi settings and restart
- **Hold 10 seconds:** Shutdown (deep sleep)

### SLAVE Node:
- **Quick press (<1s):** Wake display & force sensor reading
- **Hold 3 seconds:** Reset WiFi settings and restart
- **Hold 10 seconds:** Shutdown (deep sleep)

---

## 💡 Usage Examples

### Example 1: Battery-Powered SLAVE in Field

**Goal:** Maximize battery life for outdoor deployment

**Configuration:**
- **Update Interval:** 300 seconds (5 minutes)
- **Deep Sleep:** ENABLED
- **Deep Sleep Duration:** 300 seconds (5 minutes)
- **OLED Timeout:** 60 seconds (1 minute)
- **TX Power:** 14 dBm (medium power)

**Result:** SLAVE wakes every 5 minutes, reads sensors, transmits via LoRa, sleeps. Battery lasts **3-4 weeks**.

---

### Example 2: Continuous Monitoring

**Goal:** Real-time monitoring with fast updates

**Configuration:**
- **Update Interval:** 10 seconds
- **Deep Sleep:** DISABLED
- **OLED Timeout:** 300 seconds (5 minutes)
- **Spreading Factor:** 7 (faster transmission)

**Result:** Data updates every 10 seconds. **Requires USB power** or large battery (lasts ~2 days on battery).

---

### Example 3: Long-Range Communication

**Goal:** Extend range between MASTER and SLAVE

**Configuration (both nodes):**
- **Frequency:** 915.0 MHz (US)
- **Bandwidth:** 125 kHz (narrower = longer range)
- **Spreading Factor:** 10 (higher = more reliable)
- **TX Power:** 20 dBm (maximum power)

**Result:** Increased range up to **2-3 km** line-of-sight. Slower transmission but more reliable.

---

## 🔋 Battery Life Optimization

### Power Consumption Breakdown:

| Mode | WiFi | LoRa TX | OLED | Sensor | Total |
|------|------|---------|------|--------|-------|
| **Active (all on)** | 160 mA | 120 mA | 20 mA | 50 mA | ~350 mA |
| **WiFi + LoRa** | 160 mA | 120 mA | 0 mA | 50 mA | ~330 mA |
| **LoRa only** | 0 mA | 120 mA | 0 mA | 50 mA | ~170 mA |
| **Deep Sleep** | 0 mA | 0 mA | 0 mA | 0 mA | ~0.01 mA |

### Battery Life Estimates (3.7V 2000mAh battery):

**No Deep Sleep (continuous):**
- Update every 10s: ~6 hours
- Update every 60s: ~12 hours
- Update every 300s: ~24 hours

**With Deep Sleep:**
- Wake every 5 min: ~2 weeks
- Wake every 10 min: ~3 weeks
- Wake every 30 min: ~4 weeks

**💡 Tip:** For field deployment, use deep sleep with 5-10 minute intervals!

---

## 📊 Monitoring Configuration Changes

### Check Serial Monitor (115200 baud):

**When configuration is received:**
```
📩 MQTT message on topic: plantguard/SLAVE_001/config
📝 Received power configuration:
  Update interval: 120 seconds
  Deep sleep: ENABLED
  Deep sleep interval: 600 seconds
  OLED sleep timeout: 300 seconds
```

**When LoRa config is received:**
```
📩 MQTT message on topic: plantguard/SLAVE_001/config
📝 Received LoRa configuration:
  Frequency: 915.0 MHz
  Bandwidth: 125.0 kHz
  Spreading Factor: 9
  TX Power: 17 dBm

🔄 Applying LoRa configuration...
✓ LoRa reconfigured successfully!
```

### Check Dashboard:

- **ESP32 Raw Data tab** shows latest sensor readings
- **SLAVE display** shows current update interval and sleep status
- **MASTER display** shows gateway statistics and RSSI/SNR

---

## ⚙️ Configuration Persistence

**Important Notes:**

1. **Settings are stored in RAM only** (not in EEPROM/flash)
2. **Rebooting ESP32 resets to firmware defaults**
3. **Deep sleep preserves settings** (doesn't reboot)
4. **WiFi reset reboots** → settings lost, reconfigure via dashboard
5. **Dashboard automatically pushes config** when node reconnects to MQTT

**Best Practice:**
- After first setup, configure via dashboard
- Settings apply immediately
- If node reboots, just push settings again from dashboard

---

## 🚨 Troubleshooting

### Configuration not applying?

**Check Serial Monitor:**
1. Is node connected to WiFi? (Should show IP address)
2. Is node connected to MQTT? (Should show "connected!")
3. Is node subscribed to config topic? (Should show "Subscribed to: plantguard/SLAVE_001/config")

**If WiFi/MQTT issues:**
- Press and hold PRG button for 3 seconds → WiFi reset
- Reconnect to "PlantGuard-SLAVE_001" WiFi
- Configure WiFi again

### Deep sleep not working?

1. Check dashboard: Is "Deep Sleep" enabled?
2. Check Serial Monitor: Does it show "💤 Entering deep sleep..."?
3. Make sure update interval has passed at least once
4. SLAVE only supports deep sleep (MASTER stays awake as gateway)

### LoRa communication lost after config change?

**Most common issue: Frequency mismatch!**
- MASTER and SLAVE must use **same frequency**
- Check dashboard → both nodes should show same frequency
- Reconfigure both nodes to 915.0 MHz (US) or your regional frequency

---

## ✅ Best Practices

1. **Always configure BOTH nodes** if changing LoRa settings
2. **Test with short intervals first** (e.g., 10s) before enabling deep sleep
3. **Watch Serial Monitor** when applying configuration to confirm success
4. **Start with default settings** and adjust incrementally
5. **Use regional frequency** (915 MHz for US, 868 MHz for EU)
6. **Enable deep sleep** for battery-powered deployments
7. **Keep OLED timeout at 5 minutes** for good balance

---

**Last Updated:** November 3, 2025  
**Firmware:** MASTER & SLAVE V3.2 with Remote Configuration
