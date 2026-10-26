# Drishti AI — Empirical Verification Results & Test Records

## 1. Executive Test Record Summary

This document aggregates verified test metrics produced across automated test runners and physical device verification sessions:

```text
+---------------------------------------------------------------------------------+
|                         VERIFIED TEST RESULTS AT A GLANCE                       |
+---------------------------------------------------------------------------------+
| Automated Unit & Serialization Suites  | 476 / 476 PASS (100%)                  |
| Settings & Downstream Integration      | 472 / 472 PASS (100%)                  |
| Visual Mobility & Intent Routing       | 72 / 72 PASS (100%)                    |
| Adversarial Database & RLS Suite       | 62 / 62 PASS (100%)                    |
| Real-World Cellular Telephony (Device) | 20 / 20 PASS (100%)                    |
| Two-Sided Live GPS Telemetry (Device)  | 14 / 14 PASS (100%)                    |
| Wake Word Model Evaluation             | 118 Synthetic Clips (Status: BLOCKED)  |
+---------------------------------------------------------------------------------+
```

---

## 2. Test Suite 1: Database, Serialization & RLS Security

* **Audit Date:** October 21, 2026
* **Runner:** Gradle `testDebugUnitTest` & Node.js ESM adversarial harness
* **Target:** Live Supabase instance (`https://jbefmywcskovcsdcmdjb.supabase.co`)
* **Results:** **476 / 476 Unit Tests PASS** & **62 / 62 Adversarial Checks PASS**

| Verification Area | Total Assertions | Passed | Failed | Status |
| :--- | :---: | :---: | :---: | :---: |
| PostgreSQL Schema Completeness (21 Tables, 8 RPCs) | 29 | 29 | 0 | **PASS** |
| Android Moshi / Kotlin Model Field Alignment | 21 | 21 | 0 | **PASS** |
| Cross-Tenant Data Isolation (5 Personas) | 17 | 17 | 0 | **PASS** |
| Brute-Force Pairing Rate Limiting & Lockout | 5 | 5 | 0 | **PASS** |
| Ward Revocation Sovereign Lockdown | 5 | 5 | 0 | **PASS** |
| Serialization & Domain Unit Tests | 476 | 476 | 0 | **PASS** |

---

## 3. Test Suite 2: Settings & Downstream System Verification

* **Audit Date:** October 20, 2026
* **Environment:** Android 16 (API 36) / Vivo V2338 (Device ID: `10BEAS227B0055N`)
* **Results:** **472 / 472 Automated Tests PASS** & **20 / 20 Settings Inventoried PASS**

| Subsystem Setting Verified | UI Control | Storage Mechanism | Verified Downstream Consumer | Status |
| :--- | :--- | :--- | :--- | :---: |
| **Speech Rate** | Slider ($0.5\times\text{--}2.0\times$) | SharedPreferences | `SpeechEngineManager.setSpeechRate()` | **PASS** |
| **TTS Voice Volume** | Slider ($0\text{--}100\%$) | SharedPreferences | `AudioManager.STREAM_VOICE_CALL` | **PASS** |
| **Haptic Intensity** | Segmented Switch | SharedPreferences | `HapticAlertManager.vibrationEffect` | **PASS** |
| **Collision Distance Alert** | Stepper ($0.5\text{--}2.5\,\text{m}$) | SharedPreferences | `HazardRiskEngine.dangerThreshold` | **PASS** |
| **Language Preference** | Radio Dialog | SharedPreferences | `DefaultVoiceCommands` & TTS Locale | **PASS** |
| **Cane Telemetry Auto-Pair** | Switch | SharedPreferences | `Esp32BleManager.autoConnect` | **PASS** |

---

## 4. Test Suite 3: Real-World Cellular Telephony Suite

* **Audit Date:** October 2026
* **Hardware:** Vivo V2338 receiving cellular calls from physical caller phone
* **Results:** **20 / 20 Test Scenarios PASS (100%)**

| Test Group | Scenarios Tested | Verified Hardware Actions | Status |
| :--- | :--- | :--- | :---: |
| **Caller Identification** | Saved contact, unsaved number, private caller | Local `ContactsContract` lookup, TTS announced caller | **PASS** |
| **Multilingual Voice Answering** | English, Hindi ("कॉल उठाओ"), Marathi ("कॉल उचल") | `TelecomManager.acceptRingingCall()`, went offhook | **PASS** |
| **Multilingual Voice Rejection** | English, Hindi ("कॉल काट दो"), Marathi ("कॉल कट कर") | `TelecomManager.endCall()`, busy tone returned | **PASS** |
| **In-Call Audio Routing** | Mute, Unmute, Speaker On, Speaker Off | `AudioManager.setCommunicationDevice(Speaker)` | **PASS** |
| **State Guards** | Answering when idle, ending when no call active | TTS spoken notices: `"No call to answer"` | **PASS** |

---

## 5. Test Suite 4: Two-Sided Live Telemetry Suite

* **Audit Date:** October 22, 2026
* **Hardware:** Vivo V2338 & Remote Supabase & Parent Dashboard
* **Results:** **14 / 14 Tests PASS (100%)**

* **Child GPS Fix:** Confirmed on physical Vivo hardware: `(20.009° N, 73.767° E)` with accuracy $\pm 30.76\,\text{m}$ at `07:27:34 UTC`.
* **Telemetry Upload:** Confirmed in ADB logcat: `HEARTBEAT_TELEMETRY_UPLOAD result=Success ... batt=41%`.
* **Database Row Persistence:** Single row verified in `public.device_live_status` for Child `fd43ecae...`.
* **Approved Parent Visibility:** Authenticated Parent 1 retrieved exact child coordinates, battery ($41\%$), and live GPS marker under RLS.
* **UI Render Verification:** Frame buffer capture (`96,693 bytes`) verified Google Maps `TextureView` and green `LIVE GPS` badge on device.

---

## 6. Test Suite 5: Wake Word Model Evaluation

* **Model:** `wakeword/models/hello_drishti_float.keras` / `.tflite`
* **Test Dataset:** 118 synthetic clips (1.0s each) from Microsoft Edge TTS neural voices
* **Status:** **BLOCKED FOR PRODUCTION DEPLOYMENT**

| Metric | Measured Value | Basis / Notes | Status |
| :--- | :---: | :--- | :---: |
| **True Positives (TP)** | 46 | Correctly detected synthetic wake words | Verified |
| **False Negatives (FN)** | 2 | Missed wake attempts | Verified |
| **False Positives (FP)** | 5 | Non-target audio triggering false wake | Verified |
| **Precision** | 90.20% | $46 / (46 + 5)$ | Benchmark |
| **Recall** | 95.83% | $46 / (46 + 2)$ | Benchmark |
| **Host CPU Latency (Avg)**| 83.86 ms | Benchmark on mobile ARM CPU | Benchmark |
| **Production Recommendation**| **BLOCKED** | Zero human training samples; high false accepts in real ambient noise | Blocked |
