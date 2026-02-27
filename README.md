# WiFi Quadcopter Flight Controller

A distributed quadcopter control system using Arduino for flight control, Android for USB-to-WiFi bridging, and a web-based dashboard for remote piloting. Designed for manual (ACRO) flight mode without IMU stabilization.

![System Architecture](https://img.shields.io/badge/Arduino-Uno-00979D?logo=arduino&logoColor=white)
![Android](https://img.shields.io/badge/Android-Bridge-3DDC84?logo=android&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-Real--time-FF6B6B?logo=socket.io&logoColor=white)

## 🚁 System Overview

```
┌─────────────┐   USB OTG   ┌──────────────┐   WiFi   ┌──────────────┐
│   Arduino   │◄────────────│Android Phone │◄─────────│   Dashboard  │
│ (Controller)│             │   (Bridge)   │          │  (Web UI)    │
└──────┬──────┘             └──────────────┘          └──────────────┘
       │
    4× ESCs
       │
   4× Motors
```

### Key Features

- ✅ **Manual Flight Mode (ACRO)** - No IMU required, pure rate control
- ✅ **WiFi Control** - Pilot from laptop via web dashboard
- ✅ **Real-time Telemetry** - Motor speeds, armed status, flight mode
- ✅ **Low Latency** - ~25ms response time (100Hz command rate)
- ✅ **Auto-Arming** - Motors arm automatically on connection
- ✅ **Adjustable Gains** - Tune roll/pitch/yaw sensitivity on-the-fly
- ✅ **Failsafe Protection** - Auto-disarm on connection loss (2s timeout)
- ✅ **Kill Switch** - Emergency motor cutoff

---

## 📦 Hardware Requirements

### Flight Controller
- **Arduino Uno R3** (or compatible)
- **4× Brushless ESCs** (30A recommended)
- **4× Brushless Motors** (2204-2300KV recommended)
- **LiPo Battery** - 3S 2200mAh 80C minimum
- **Power Distribution Board (PDB)**
- **Quadcopter frame** (X configuration)

### Bridge System
- **Android phone** with USB OTG support
- **USB OTG cable**
- **Phone mount** for quadcopter frame

### Control Station
- **Laptop/Desktop** with WiFi and web browser
- Same WiFi network as Android phone

---

## 🔧 Software Components

### 1. Arduino Flight Controller (`manual_flight_controller.ino`)

**Features:**
- X-frame motor mixing (no stabilization)
- Adjustable control gains (roll, pitch, yaw)
- Serial command interface (115200 baud)
- 10Hz telemetry output
- 2-second failsafe timeout

**Pin Configuration:**
```
ESC 1 (Front Right) → Pin 1
ESC 2 (Rear Right)  → Pin 3
ESC 3 (Rear Left)   → Pin 5
ESC 4 (Front Left)  → Pin 7
ESC GND (any one)   → Arduino GND
```

### 2. Android Bridge App (`MainActivity.java`)

**Features:**
- USB OTG serial communication (115200 baud)
- WebSocket server on port 8080
- Auto-detects Arduino Uno R3 (VID:0x2341, PID:0x0043)
- Auto-arms motors on connection
- Real-time telemetry forwarding

**Dependencies:**
```gradle
implementation 'com.github.felHR85:UsbSerial:6.1.0'
implementation 'org.java-websocket:Java-WebSocket:1.5.3'
```

### 3. Web Dashboard (`quadcopter_dashboard.html`)

**Features:**
- Joystick for pitch/roll control
- Throttle slider (0-1000, limited to 700 for testing)
- ARM/DISARM/KILL buttons
- Real-time telemetry display
- 100Hz command rate for low latency

---

## 🚀 Quick Start

### Step 1: Arduino Setup

```bash
# 1. Calibrate ESCs
#    Upload esc_calibration.ino
#    Follow Serial Monitor instructions

# 2. Upload flight controller
#    Upload manual_flight_controller.ino
#    Verify "READY FOR MANUAL FLIGHT" appears
```

### Step 2: Android App

```bash
# 1. Open Android Studio
# 2. Build → Build APK
# 3. Install on phone
# 4. Grant USB permissions
```

### Step 3: Connect Everything

```bash
# 1. Connect Arduino to phone via USB OTG
# 2. Open Quadcopter Bridge app
# 3. Wait for "✓ Motors AUTO-ARMED"
# 4. Note phone's IP address
# 5. Open quadcopter_dashboard.html in browser
# 6. Connect to phone_ip:8080
# 7. Start flying!
```

---

## 🎮 Flight Controls

**Dashboard Interface:**
- **Joystick:** Pitch (forward/back) and Roll (left/right)
- **Throttle Slider:** Vertical thrust (0-700 for testing)
- **Yaw Slider:** Rotation (left/right)
- **ARM/DISARM:** Enable/disable motors
- **KILL:** Emergency stop

**⚠️ Manual Mode Warning:**
Unlike stabilized flight, manual mode has NO auto-leveling. The drone will not self-correct - you must constantly make small adjustments to maintain stable flight. Requires 20-50 hours of practice!

---

## 🛡️ Safety Features

- **Failsafe (2s timeout):** Auto-disarms if connection lost
- **Throttle Limiting:** Max 70% power during testing
- **Kill Switch:** Instant motor cutoff
- **Low Voltage Protection:** ESC-based battery monitoring

### Pre-Flight Checklist

```
□ Remove all propellers for bench testing
□ Battery fully charged (12.6V for 3S)
□ All connections secure
□ Dashboard connected (green indicator)
□ Telemetry updating
□ Start with LOW gains (0.2,0.2,0.1)
□ Clear flying area
□ Safety glasses on
```

---

## 📊 Performance

- **Latency:** ~20-25ms end-to-end
- **Command Rate:** 100Hz (10ms intervals)
- **Telemetry Rate:** 10Hz
- **Arduino Memory:** ~8KB flash, ~800 bytes SRAM
- **Android APK:** ~2-3 MB

---

## 🐛 Troubleshooting

### "No USB devices found"
- Check USB OTG cable (data, not charging-only)
- Enable USB OTG in phone settings
- Grant USB permissions
- Verify Arduino VID/PID (0x2341:0x0043)

### Motors cut out at higher throttle
- Check battery voltage (> 11.4V for 3S)
- Add 1000µF capacitor to Arduino power
- Use battery with 80C+ rating

### Continuous ESC beeping
- Low battery voltage (< 10.5V)
- Check ESC signal wire connections
- Verify throttle starts at 1000µs

### High latency
- Use 5GHz WiFi instead of 2.4GHz
- Reduce WiFi distance (< 3m)
- Close background apps on phone

---

## 📝 Serial Protocol

```
Commands (to Arduino):
  ARM:1              - Arm motors
  ARM:0              - Disarm motors
  CTL:t,r,p,y        - Control: throttle, roll, pitch, yaw
  GAIN:r,p,y         - Set gains (0.0-1.0)
  KILL:              - Emergency stop

Responses (from Arduino):
  TEL:MANUAL,t,a,m1,m2,m3,m4,rg,pg,yg
  PONG               - Ping response
  FAILSAFE:TRIGGERED - Connection lost
```

---

## 🎓 Learning Progression

1. **Bench Testing (1-2 hrs):** Test without propellers
2. **Tethered Hovering (10-20 hrs):** Practice with 1m tether
3. **Ground Effect (20+ hrs):** Low altitude hovering
4. **Forward Flight (50+ hrs):** Full control practice

**Manual mode requires 10x more practice than stabilized flight!**

---

## 🤝 Contributing

Contributions welcome! Areas for improvement:

- [ ] Add IMU support for stabilized flight
- [ ] Implement PID control loops
- [ ] Add GPS waypoint navigation
- [ ] Battery voltage monitoring
- [ ] Flight logging
- [ ] Mobile dashboard app

---

## 📧 Support

- **Issues:** GitHub Issues
- **Discussions:** GitHub Discussions

---

## ⚠️ Disclaimer

This project is for educational purposes. Fly at your own risk. Always:
- Start with propellers removed
- Use tether for first flights
- Fly in open areas only
- Follow local regulations
- Never fly near people or property

---

## 📚 Documentation

- [Complete Installation Guide](docs/INSTALLATION.md)
- [Manual Flight Guide](MANUAL_FLIGHT_GUIDE.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Serial Protocol](docs/SERIAL_PROTOCOL.md)

---

## 📄 License

MIT License - See LICENSE file for details

---

**🚀 Happy Flying!**
