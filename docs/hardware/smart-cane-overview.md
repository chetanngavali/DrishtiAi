# Drishti AI Smart Cane — Hardware System Overview

## 1. Hardware Objectives & Context

Traditional white canes are the gold standard for tactile ground-level feedback, detecting drop-offs, curbs, and path textures. However, they possess a critical geometric blind spot: obstacles located between waist and head height (such as open truck tailgates, low-hanging tree branches, construction barriers, and scaffolding).

The **Drishti AI Smart Cane** augments conventional mobility canes with an intelligent, lightweight embedded sensor suite:
* **Millimeter-Accurate Ranging:** VL53L0X Time-of-Flight (ToF) laser lidar detects approaching obstacles up to 2.0 meters away.
* **Autonomous Real-Time Safety:** Handle vibration and buzzer alerts trigger directly from the onboard ESP32 within $\le 50\,\text{ms}$, completely independent of smartphone connectivity.
* **Wireless Telemetry Link:** Streams obstacle distance, battery percentage, and sensor health to the Android host application via Bluetooth Low Energy (BLE).
* **Optional Optical Streaming:** An ESP32-CAM variant provides localized MJPEG video frames over a secured Wi-Fi link for advanced environmental parsing.

```
+-----------------------------------------------------------------------------------+
|                        DRISHTI AI SMART CANE SYSTEM ARCHITECTURE                  |
+-----------------------------------------------------------------------------------+
|  [VL53L0X ToF Lidar] ---> I2C Bus ---> [ESP32 Dual-Core MCU (FreeRTOS)]           |
|                                                |                                  |
|         +--------------------------------------+---------------------+            |
|         |                                                            |            |
|         v                                                            v            |
|  [Autonomous Alerts]                                         [Wireless Telemetry] |
|   - Handle ERM Vibration Motor (PWM)                          - BLE GATT (NUS)    |
|   - Piezoelectric Buzzer                                      - HTTP /stream WiFi |
|                                                                      |            |
|                                                                      v            |
|                                                           [Drishti Android App]   |
+-----------------------------------------------------------------------------------+
```

---

## 2. Core Bill of Materials (BOM)

| Component | Technical Specification | Functional Role |
| :--- | :--- | :--- |
| **Microcontroller** | AI-Thinker ESP32-CAM / ESP32-WROOM-32 (240 MHz, 520 KB SRAM, 4 MB PSRAM) | Central compute, sensor sampling, FreeRTOS tasks, BLE & Wi-Fi radios |
| **Distance Sensor** | STMicroelectronics VL53L0X Time-of-Flight (940 nm VCSEL laser) | 10 Hz obstacle ranging ($30\,\text{mm}$ to $2000\,\text{mm}$ range) |
| **Haptic Actuator** | 1027 Coin-Type Eccentric Rotating Mass (ERM) Vibration Motor ($3.0\,\text{V}$) | Direct tactile tactile pulse delivered inside the user's grip handle |
| **Acoustic Buzzer** | $3.3\,\text{V}$ Active Magnetic Piezo Buzzer ($85\,\text{dB}$ at $10\,\text{cm}$) | Audible proximity beeps for high-risk collision warnings |
| **Battery Cell** | 3.7V 18650 Protected Li-ion ($2600\,\text{mAh}$) or 3.7V LiPo ($1200\,\text{mAh}$) | Portable power source powering MCU and sensors |
| **Power Management** | TP4056 USB-C Li-ion Charger + ME6211 $3.3\,\text{V}$ Low-Dropout (LDO) Regulator | Safe 5V USB-C charging, over-discharge protection, clean 3.3V rail |
| **Camera Sensor** | OmniVision OV2640 (2 Megapixel DVP interface) *(ESP32-CAM model)* | Onboard optical frame capture for streaming to phone |
| **Enclosure** | Custom 3D Printed PETG / ABS Ergonomic Handle Housing | Weather-resistant, impact-resistant mount for cane shaft attachment |

---

## 3. Operational Modes

### 3.1 Standalone Autonomous Mode (Zero Phone Dependency)
When the user is walking without their smartphone or if the phone's battery dies:
* The ESP32 boots in $< 500\,\text{ms}$ upon power switch engagement.
* The sensor loop continuously reads the VL53L0X at 10 Hz.
* When an obstacle enters the warning zone ($< 1.2\,\text{m}$), the handle vibration motor pulses with frequency inversely proportional to distance.
* If an obstacle breaches the critical danger threshold ($< 0.6\,\text{m}$), the buzzer sounds an urgent auditory alert.
* **Result:** User safety is never compromised by phone operating system updates, crashes, or dead smartphone batteries.

### 3.2 Paired Telemetry Mode (Connected to Drishti App)
When paired with the Drishti Android client:
* The ESP32 advertises the Nordic UART Service (NUS) over BLE GATT.
* The Android `Esp32BleManager` connects automatically.
* Distance readings and battery voltage telemetry stream to the phone at 5 Hz.
* Telemetry data fuses with the phone's GPS and CameraX pipeline:
  * Voice commands can answer: *"What is my cane battery?"* or *"What is the distance ahead?"*
  * Active emergency SOS broadcasts include live cane obstacle distance to show caregivers if the user has fallen against a wall or obstacle.
