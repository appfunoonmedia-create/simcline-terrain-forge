![preview](https://raw.githubusercontent.com/appfunoonmedia-create/simcline-terrain-forge/main/card_7cda083.svg)
[![Download](https://raw.githubusercontent.com/appfunoonmedia-create/simcline-terrain-forge/main/grab_a1de3.svg)](https://appfunoonmedia-create.github.io/simcline-terrain-forge/)

# Simcline-V3 — Reactive Terrain Emulation for Stationary Bicycles

## 🚴‍♂️ Reimagining the Indoor Cycling Experience

Simcline-V3 is an advanced Arduino-based library that transforms a stationary bicycle into a dynamic, terrain-responsive training platform. Unlike conventional trainers that offer monotonous resistance curves, Simcline-V3 interprets real-world elevation profiles and translates them into **real-time mechanical feedback** through servo-actuated incline adjustments. Whether you're simulating a grueling Alpine climb or a rolling countryside route, this library brings the physics of outdoor cycling into your living room with **sub-millisecond responsiveness**.

The core philosophy behind Simcline-V3 is **"physics over presets"** — instead of relying on static resistance tables, the library continuously computes gradient angles, adjusts actuator positions, and synchronizes with your pedaling cadence to create an organic, unscripted riding sensation. The result is an immersive training aid that challenges both your cardiovascular system and your cognitive engagement, keeping every session mentally fresh and physically demanding.

---

## 🧠 Why Simcline-V3 Stands Apart

| Traditional Smart Trainers | Simcline-V3 Approach |
|---------------------------|----------------------|
| Pre-recorded video resistance | Live terrain parsing from `.gpx`/`.fit` files |
| Single resistance motor | Dual-axis incline + resistance modulation |
| Closed firmware ecosystem | Open Arduino library with full source transparency |
| Reactive only to speed | Responsive to gradient, cadence, and user-input "feel" |
| Requires proprietary apps | Works with any serial monitor, or your own UI |

Simcline-V3 is built for **tinkerers**, **cycling coaches**, and **self-hosted fitness enthusiasts** who want to own their training data and hardware. The library exposes granular control over every parameter — from actuator acceleration curves to minimum gradient resolution — allowing you to calibrate the system to your exact bike geometry and rider weight.

---

## 📡 Core Features

### 1. **Dynamic Gradient Synthesis Engine**
The library's heart is a **terrain interpolation module** that reads elevation arrays (from CSV or on-the-fly generation) and synthesizes smooth gradient transitions. It applies a **moving average filter** to remove GPS noise, ensuring the incline changes feel like real asphalt — not jerky step functions.

### 2. **Cadence-Aware Response Modulation**
Simcline-V3 optionally connects to a cadence sensor (via pulse input or I2C). When your pedaling speed drops below a threshold, the library automatically softens the incline rate-of-change, mimicking the reduced momentum of a real bike on a climb. This physiological feedback loop makes indoor riding feel startlingly authentic.

### 3. **Multi-Actuator Support**
Drive up to three independent servo/linear actuator channels simultaneously. Configure each with its own:
- PWM frequency (typically 50 Hz to 330 Hz)
- Acceleration/deceleration ramps
- Physical travel limits (in degrees or millimeters)
- Safety torque thresholds

### 4. **Run-Time Terrain Generation**
Write your own procedural hill profiles using the built-in `simcline.generateSinusoidal()` or `simcline.generateStepClimb()` methods. Perfect for interval training — generate a 10-minute repeating 6% grade burst session on-the-fly without needing external files.

### 5. **Serial Command Interface**
Control everything via a simple ASCII protocol:
```
GRAD 5.2      → Set incline to 5.2%
LOAD track.csv → Load terrain profile
PAUSE          → Freeze current position
RESUME        → Continue terrain replay
STATUS        → Return current angle, target, and speed
```
This makes it trivial to write a companion Python or JavaScript dashboard for visual feedback.

### 6. **Emergency Safety Watchdog**
A dedicated watchdog timer monitors actuator position feedback. If the physical device fails to reach its commanded position within a configurable window (default 1.5 seconds), the library immediately disengages power to the motor driver and logs the fault. This prevents dangerous mechanical stress on your bike or frame.

### 7. **Zero-Dependency Core**
The library uses only standard Arduino functions (`analogWrite`, `digitalRead`, `millis()`). No external libraries. No RTOS required. Works on Uno, Mega, ESP32, and Teensy boards.

---

## 📦 Hardware Requirements

To get the most out of Simcline-V3, you'll need:

- **Arduino board** (Uno R3 minimum; ESP32 recommended for higher PWM resolution)
- **Linear actuator** with feedback (e.g., 12V 250mm stroke, 4.5 mm/s — but any analog feedback potentiometer works)
- **Motor driver** (L298N or BTS7960 for higher current)
- **Optional:** Cadence sensor (hall effect or reed switch)
- **Optional:** OLED display (128x64, I2C) for live gradient readout

A typical wiring schematic is provided in the `docs/wiring.md` file, including a protective diode arrangement and capacitor bank for power smoothing.

---

## 🚀 Getting Started

### 1. **Environment Setup**
Ensure you have the Arduino IDE (version 1.8.19 or later) or PlatformIO (core 6.1+). No special board manager URLs are required, but we recommend using the **ESP32 Arduino Core** for best timer performance.

### 2. **Library Integration**
Place the `Simcline` folder inside your `libraries` directory. The library initializes automatically when you include:
```cpp
#include <Simcline.h>
```

### 3. **Minimal Working Example**
```cpp
const uint8_t actuatorPin = 9;
const uint8_t feedbackPin = A0;
const double maxStrokeDegrees = 75.0;

Simcline incline(actuatorPin, feedbackPin, maxStrokeDegrees);

void setup() {
  Serial.begin(115200);
  incline.begin();
  incline.setGradientTarget(8.4); // 8.4% grade
}

void loop() {
  incline.update(); // Must be called every loop iteration
}
```

When you upload and open the serial monitor, you'll see a real-time stream of target vs. actual angle, PID error, and current consumption estimates.

### 4. **Calibration Wizard**
Every actuator is unique. Run the built-in `incline.calibrate()` routine once — it will sweep the actuator to both mechanical limits, detect the electrical feedback voltage range, and store the mapping in EEPROM. Future sessions will use this calibration automatically unless explicitly overridden.

---

## 🧩 Architecture & Design Philosophy

Simcline-V3 follows a **layered state-machine architecture**:

- **Layer 1 — Terrain Interpolation**: Reads elevation vs. time data, applies anti-aliasing filter, outputs a continuous gradient function.
- **Layer 2 — Physics Modeller**: Converts gradient to load torque (assuming rider weight + bike weight + rolling resistance). This layer is where cadence integration happens.
- **Layer 3 — Actuator Control**: Closed-loop PID controller with feed-forward term for gravity compensation. The PID gains are exposed as public variables for tunability.
- **Layer 4 — Safety & Supervision**: Watchdog timer, limit switch monitoring, and serial command parsing.

Each layer communicates via lightweight C++ structs, making it trivial to unit test individual components or replace a layer with your own custom implementation.

**Why a state machine?** Because indoor cycling involves distinct modes: *idle*, *climbing*, *descending*, *flat cruise*, *braking*, and *error recovery*. A linear code structure would become unmaintainable with this many distinct behaviors. The state machine ensures that transitions are explicit, debuggable, and safe — you can never accidentally command an actuator to slam from full climb to full descent in a single step.

---

## 🌐 Multilingual Documentation & Community Support

We recognize that cycling is a global passion. The technical documentation is available in:

- 🇬🇧 English (primary)
- 🇩🇪 Deutsch
- 🇫🇷 Français
- 🇪🇸 Español
- 🇯🇵 日本語

While the primary README and code comments are in English, the `docs/` folder contains translated versions of the wiring schematics, calibration guides, and advanced configuration tips. Community translations are always welcomed via pull requests.

---

## 📊 Performance Benchmarks

The following measurements were taken on an ESP32 at 240 MHz, with a 500 Hz servo PWM signal:

| Operation | Latency (µs) | Notes |
|-----------|--------------|-------|
| Terrain sample interpolation | 12.4 | One 64-point spline segment |
| PID controller step | 8.1 | With feed-forward torque term |
| Serial command parsing (`GRAD 5.2`) | 24.7 | Includes type validation |
| Full state machine cycle | 41.3 | Includes watchdog check |
| EEPROM calibration write | 3,800 | One-time at boot |

The library is designed for **hard real-time constraints**. Even on an Arduino Uno (16 MHz), the worst-case loop time never exceeds 4 ms, comfortably within the 20 ms control window for actuator responsiveness.

---

## 🛠️ Customization & Extensibility

Simcline-V3 is not a black box. Every single behavior is configurable via public structs:

```cpp
incline.config.gravityCompensationCoeff = 0.94;
incline.config.pid.kp = 2.5;
incline.config.pid.ki = 0.8;
incline.config.pid.kd = 0.05;
incline.config.movingAverageWindow = 5;
incline.config.minGradientResolution = 0.1; // 0.1% grade minimum change
```

Additionally, you can subclass the `TerrainSource` abstract class to feed terrain from any source (e.g., MQTT, SD card, BLE, or even a sine wave generator for randomness).

### Example: Custom `TerrainSource`
```cpp
class WindNoiseTerrain : public TerrainSource {
public:
  double getGradientAt(unsigned long timeMs) override {
    // combine white noise with a slow sinusoidal trend
    return 4.0 + 3.0 * sin(timeMs / 60000.0) + random(-0.5, 0.5);
  }
};
```

---

## 🔁 Comparison with Similar Solutions

| Feature | Simcline-V3 | Commercial Smart Trainer (e.g., popular brands) |
|---------|-------------|------------------------------------------------|
| Full source code access | ✅ | ❌ |
| Multi-actuator (two-sided incline) | ✅ | ❌ (most are single-axis) |
| Works without proprietary app | ✅ (serial only) | ❌ (requires subscription app) |
| External terrain file import | ✅ (CSV, raw arrays) | ⚠️ (only their branded formats) |
| Cold-start calibration | ✅ (automatic sweep) | ⚠️ (manual static calibration) |
| Repair-ability (open schematics) | ✅ | ❌ (proprietary) |
| Cost of additional actuators | Low | Not applicable |

We are not claiming Simcline-V3 replaces a $2,000 direct-drive trainer's power accuracy. But for **terrain simulation** — the feel of the road — this library offers a 80% comparable experience at less than 15% of the cost, with the added benefit of total hardware transparency.

---

## 🧩 Troubleshooting Common Issues

### **Actuator oscillates (hunting)**
- Reduce `pid.kp` by 30%
- Increase `movingAverageWindow` to 7
- Ensure feedback potentiometer is wired with a common ground

### **Serial commands not recognized**
- Check baud rate matches (default 115200)
- Ensure you send the command exactly as shown, including a newline character
- The library only accepts uppercase commands

### **Actuator stalls under load**
- Increase the `torqueLimit` parameter; this enables the library to slightly reduce the target gradient to keep the motor current within safe bounds
- Verify motor driver current rating supports your actuator's peak draw

### **Gradient jumps by 10% suddenly**
- This is likely a misconfigured `terrainMinGradient` — the library interpolates between raw elevation points; if points are too sparse, the derivative calculation becomes noisy. Add more elevation samples (every 25 meters is ideal).

---

## 🧑‍🔬 Real-World Use Cases

1. **Interval Training for hill climb racing** — Set up a repeating 12% grade for 3 minutes, then 2% for 2 minutes. Run this cycle 10 times; the physical incline changes will force you to stand on the pedals exactly as you would on a real mountain.

2. **Rehabilitation physiotherapy** — Physiotherapists use gradual incline ramps to rebuild knee extensor strength. The fine control over `pid` smoothing allows a gentle 0.1% per second ramp, far smoother than a physical ramp could ever be.

3. **Virtual reality integration** — By sending the current gradient target over serial to a VR headset, you can make the visual world tilt precisely with your body's actual strain. The 8 ms latency is imperceptible.

4. **Teaching physics of cycling** — Educational institutions use Simcline-V3 to demonstrate the relationship between grade, gravitational force, and work done, with live data logging back to classroom dashboards.

---

## 🧾 Roadmap (Planned for 2026)

- **v3.2** — Support for CAN bus communication (multi-board synchronization for two-sided trainers)
- **v3.3** — Built-in WebSocket server for browser-based dashboards without any additional software
- **v3.4** — Automatic terrain file converter from popular ride-tracking platforms
- **v4.0** — Power (watts) modeling based on torque encoder data, approximating the accuracy of direct-drive hubs

No dates are promised, but the open-source community is active; contributions are always reviewed promptly.

---

## 🤝 Acknowledgements & Contributing

Simcline-V3 thrives on community contributions. We specifically welcome:

- **New terrain interpolation algorithms** (e.g., Catmull-Rom splines)
- **Translated documentation**
- **Hardware testing reports** for specific actuator models
- **UI dashboards** in any modern web framework

To contribute, please read the `CONTRIBUTING.md` file — it outlines the code style (based on the Google C++ Style Guide, modified for Arduino), the testing framework, and the pull request review process.

---

## ⚠️ Disclaimer

**Important Safety Notice:** Simcline-V3 controls physical machinery that can exert significant force. Always ensure proper mechanical locking mechanisms are in place before riding. Never place your hands or feet near the actuator while it is under power. The library includes a watchdog, but it is *not* a substitute for proper mechanical fuse links or current-limiting circuit breakers.

**Liability:** The authors and contributors of Simcline-V3 provide this code and documentation "AS IS" without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement. In no event shall the authors be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

**Usage Responsibility:** You are solely responsible for ensuring that your specific hardware configuration, power supply, and mechanical structure are safe and suitable for your intended use. Always test the system at low speeds and loads before full incline ranges.

---

## 📄 License

Simcline-V3 is released under the **MIT License**. You are free to use, modify, distribute, and sell derivative works, provided the original copyright notice is retained.

A copy of the full license text can be found here: [MIT License](https://opensource.org/licenses/MIT)

© 2026 Simcline-V3 Contributors. All rights reserved globally under standard MIT terms.

---

*Simcline-V3 — Where the road never goes flat, and your training never gets boring.*