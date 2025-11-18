# MASTER Node Display Troubleshooting Guide

## 🔍 Problem: Display showing nothing (blank screen)

---

## ✅ STEP 1: Upload Simple Test Sketch

**Upload this first:** `ESP32_MASTER_DisplayTest.ino`

This minimal sketch tests ONLY the display (no WiFi, MQTT, LoRa complexity).

### What to expect:
- **Serial Monitor (115200 baud):**
  ```
  === PlantGuard Display Test ===
  Heltec board initialized
  ✓ Display initialized
  ✓ Test text drawn
  ```

- **OLED Display:**
  ```
  PlantGuard
  Count: 1
  Display works!
  ```
  (Counter increments every second)

### ✅ If test works:
→ Display hardware is fine, issue is in main firmware

### ❌ If test fails:
→ Hardware or library issue - continue to Step 2

---

## 🔧 STEP 2: Verify Hardware & Library Setup

### A. Check Arduino IDE Board Settings
1. **Tools → Board:** 
   - Select: **"WiFi LoRa 32(V3) / Wireless shell(V3) / Wireless stick lite (V3)"**
   - (NOT V2, NOT "Heltec WiFi Kit 32")

2. **Tools → Upload Speed:** `921600`
3. **Tools → CPU Frequency:** `240MHz (WiFi/BT)`
4. **Tools → Port:** Select your ESP32 COM port

### B. Install Correct Library
**CRITICAL:** You need the **UNOFFICIAL** Heltec library for V3.2

#### Install Steps:
1. Arduino IDE → **Sketch → Include Library → Manage Libraries**
2. Search: **"Heltec ESP32 LoRa v3"**
3. Install: **"Heltec ESP32 LoRa v3" by ropg** (NOT official Heltec)
4. Version: Latest (check for 1.1.0 or newer)

#### Wrong Library Signs:
- ❌ "Heltec ESP32 Dev-Boards" by Heltec Automation
- ❌ Any library that mentions "V2" only
- ❌ Library that uses `#include <heltec.h>` (old version)

✅ **Correct library uses:** `#include <heltec_unofficial.h>`

---

## 🔍 STEP 3: Check Serial Monitor Output

### Upload test sketch and watch Serial Monitor (115200 baud)

#### ✅ Good Output:
```
=== PlantGuard Display Test ===
Heltec board initialized
✓ Display initialized
✓ Test text drawn
Counter: 1
Counter: 2
```

#### ❌ Bad Output (Crashes):
```
Guru Meditation Error: Core 1 panic'ed
```
**→ Wrong board selected or library issue**

#### ❌ Bad Output (Nothing at all):
```
(completely blank)
```
**→ USB cable issue or wrong COM port**

---

## 🔋 STEP 4: Check Hardware Power

### Battery vs USB:
1. **Try USB power only** (disconnect battery)
2. **Try battery only** (fully charged, disconnect USB)
3. Check battery voltage: Should be >3.7V

### Physical Inspection:
- ✅ OLED ribbon cable connected firmly to board
- ✅ No physical damage to screen
- ✅ Board LED lights up when powered

---

## 🐛 STEP 5: Common Issues & Fixes

### Issue: "Display not declared in this scope"
**Fix:** Wrong library installed
- Uninstall all Heltec libraries
- Install ONLY: "Heltec ESP32 LoRa v3 by ropg"

### Issue: Compile error "Vext not defined"
**Fix:** Updated firmware (already fixed in latest version)
- The Heltec V3.2 library handles Vext automatically
- Code no longer uses manual `digitalWrite(Vext, LOW/HIGH)`

### Issue: Code uploads but screen stays blank
**Possible causes:**
1. Wrong board selected (using V2 instead of V3)
2. Display sleeping (press PRG button to wake)
3. Hardware defect (rare)

### Issue: Screen works in test but not in MASTER firmware
**Fix:** Upload fresh MASTER firmware
- Make sure you have latest version (Vext code removed)
- WiFi config might timeout - display sleeps after

---

## 🎯 STEP 6: Upload MASTER Firmware

**After test sketch works:**

1. Open: `ESP32_PlantGuard_MASTER.ino`
2. Verify settings at top of file:
   ```cpp
   const char* NODE_ID = "MASTER_001";  // Your node ID
   #define LORA_FREQUENCY 915.0          // 915 for US
   ```
3. Upload to board
4. Open Serial Monitor (115200 baud)

### Expected Boot Sequence:

**Serial Monitor:**
```
🌱 PlantGuard MASTER Node
============================
PlantGuard MASTER
LoRa init success!
Starting WiFi Manager...
```

**Display shows:**
```
Page 1: Logo
"PlantGuard"
"MASTER"

Page 2: WiFi Config
"WiFi Config Mode"
"Connect to:"
"PlantGuard-MASTER_001"
```

### WiFi Configuration:
1. Connect phone/computer to WiFi: **"PlantGuard-MASTER_001"**
2. Browser opens automatically (or go to: 192.168.4.1)
3. Select your WiFi network and enter password
4. Save

**After WiFi connects:**
Display rotates through 4 pages every 15 seconds:
- Page 1: MASTER status (WiFi, MQTT, battery)
- Page 2: LoRa gateway stats (packets received/forwarded)
- Page 3: Signal quality (RSSI/SNR)
- Page 4: Local sensor data

---

## 🔘 Button Controls

**PRG/BOOT Button (GPIO 0):**
- **Quick press (<1s):** Wake display + scroll to next page
- **Hold 3 seconds:** Reset WiFi settings and restart
- **Hold 10 seconds:** Shut down (deep sleep)

---

## 📞 Still Not Working?

### Share this info:
1. **Serial Monitor output** (copy/paste full boot log)
2. **Arduino IDE settings** (Board, Upload Speed)
3. **Library version** (Sketch → Include Library → Manage Libraries → search "Heltec")
4. **Test sketch result** (did display test work?)
5. **Hardware details** (battery connected? USB power?)

---

## ✅ Success Checklist

- [ ] Test sketch shows counter on display
- [ ] Serial monitor shows boot messages
- [ ] WiFi config page appears on display
- [ ] After WiFi connects, display rotates through 4 pages
- [ ] PRG button wakes display and scrolls pages
- [ ] MQTT connected (check Serial Monitor)
- [ ] Dashboard shows MASTER node data

---

**Last Updated:** November 3, 2025
**Firmware Version:** MASTER V3.2 with LoRa Config + Power Management
