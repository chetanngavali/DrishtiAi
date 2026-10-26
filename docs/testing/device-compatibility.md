# Drishti AI — Hardware & Device Compatibility Matrix

## 1. Supported Platform Matrix

Drishti AI is developed for and validated across standard Android smartphone hardware and low-power ESP32 microcontrollers:

```text
+---------------------------------------------------------------------------------+
|                          COMPATIBILITY AT A GLANCE                              |
+---------------------------------------------------------------------------------+
| Supported Android Versions   | Android 9.0 (API 28) through Android 16 (API 36) |
| Primary Reference Device     | Vivo V2338 (Android 16 / Funtouch OS)            |
| Bluetooth Low Energy Support | BLE 4.2 & BLE 5.0 Core Specifications            |
| Embedded Microcontrollers    | Espressif ESP32-CAM, ESP32-WROOM-32              |
| Distance Ranging Sensor      | STMicroelectronics VL53L0X Time-of-Flight        |
+---------------------------------------------------------------------------------+
```

---

## 2. Tested Android Handsets & Status

| Manufacturer / Model | OS Version / API | Chipset / Architecture | Verification Status | Notes & Performance |
| :--- | :--- | :--- | :---: | :--- |
| **Vivo V2338 (Reference)** | Android 16 (API 36) | MediaTek Dimensity / ARM64 | **Verified & Validated** | Primary test device; 72 QA tests, 20 call tests, 14 telemetry tests passed |
| **Google Pixel 7 / 8** | Android 14 / 15 | Google Tensor G2/G3 (ARM64) | **Supported** | Clean CameraX, high-fidelity NNAPI ML Kit acceleration |
| **Samsung Galaxy A53 / S23**| Android 13 / 14 | Exynos / Snapdragon | **Supported** | Samsung TTS engine supported; requires background battery whitelist |
| **OnePlus / Realme / Oppo** | Android 12 / 13 | MediaTek / Snapdragon | **Supported** | OxygenOS / ColorOS power management requires auto-start enablement |
| **Xiaomi / Redmi / Poco** | Android 11 / 12 / 13 | Snapdragon / MediaTek | **Supported** | MIUI / HyperOS requires "Display pop-up windows while running in background" |

---

## 3. OEM-Specific Behavior & Workarounds

Custom OEM Android distributions incorporate aggressive battery-saving daemons that can suppress background assistive listeners. Drishti AI handles these quirks programmatically:

### 3.1 Vivo / Funtouch OS
* **Behavior:** Funtouch OS terminates long-running foreground services after prolonged screen-off intervals.
* **Workaround:** Drishti AI checks `PowerManager.isIgnoringBatteryOptimizations()`. If false, the assistant prompts the user via TTS: *"Please allow Drishti to run in the background without battery restrictions"*, and presents `ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`.

### 3.2 Android 12+ (API 31+) Bluetooth Permissions
* **Behavior:** Legacy Android required `ACCESS_FINE_LOCATION` to scan for Bluetooth devices. Android 12 introduced dedicated `BLUETOOTH_SCAN` and `BLUETOOTH_CONNECT` permissions.
* **Workaround:** Drishti AI sets `neverForLocation` on `android.permission.BLUETOOTH_SCAN` in `AndroidManifest.xml`. This enables smart cane pairing without forcing the user to enable system GPS solely for Bluetooth.

### 3.3 Android 14+ Foreground Service Types
* **Behavior:** Android 14 enforces strict declaration of foreground service types. Failure to specify types causes immediate runtime crashes.
* **Workaround:** `DrishtiWakeWordService` declares:
  ```xml
  android:foregroundServiceType="microphone|connectedDevice"
  ```
  allowing continuous microphone coordination and smart cane BLE streaming simultaneously.

---

## 4. Embedded Hardware Specifications

| Component | Minimum Supported Hardware | Recommended Hardware |
| :--- | :--- | :--- |
| **Microcontroller** | ESP32-WROOM-32 (Standard) | ESP32-CAM with 4MB external PSRAM |
| **Camera Module** | None (Sensors only) | OmniVision OV2640 (2 Megapixel) |
| **Ranging Sensor** | STMicroelectronics VL53L0X | VL53L0X with protective optical window |
| **Haptic Motor** | Standard $3.0\,\text{V}$ coin ERM motor | $1027$ ERM motor with MOSFET low-side drive |
| **Battery Cell** | 3.7V 18650 Li-ion cell ($1800\,\text{mAh}$) | Protected 3.7V 18650 cell ($2600\text{--}3000\,\text{mAh}$) |
