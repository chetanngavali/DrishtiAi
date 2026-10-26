# Drishti AI — Requirements and Feature Inventory

This document defines the functional and non-functional requirements for Drishti AI and provides a verified feature inventory classified by engineering status.

## 1. Feature Status Legend

* **Implemented & Empirically Verified:** Production code deployed, passing automated test suites, and verified on physical hardware (Vivo V2338 / ESP32-CAM).
* **Partially Implemented:** Functional implementation present in codebase; specific automated workflows or cross-module integrations require further completion or device validation.
* **Planned / In Development:** Technical architecture specified and designs finalized; active development underway.
* **Blocked:** Identified blockers prevent production deployment (e.g., awaiting real-world human data collection).

---

## 2. Comprehensive Module Inventory Matrix

| Module | Sub-Feature / Capability | Engineering Status | Primary Source Artifacts & Verified Path | Verification Evidence & Notes |
| :--- | :--- | :---: | :--- | :--- |
| **Vision & Perception** | CameraX Lifecycle & Live Preview | **Verified** | `vision/VisionManager.kt`, `ScanScreen.kt` | 30+ FPS preview, stable Android lifecycle transitions |
| | ML Kit On-Device Text Recognition | **Verified** | `vision/OcrRecognitionEngine.kt` | Local OCR on physical device; printed text announced via TTS |
| | OCR Text Deduplication & Reading | **Verified** | `vision/OcrDeduplicationManager.kt` | Jaccard similarity windowing prevents repetitive spoken loops |
| | Real-Time Object Detection | **Verified** | `vision/ObjectDetectionEngine.kt`, `CameraSearchEngine.kt` | ML Kit Object Detection with bounding box tracking |
| | Perspective Pinhole Depth Estimation | **Verified** | `vision/DepthEstimator.kt` | Monocular pinhole geometry; labeled as ESTIMATED meters |
| | Signboard & Portal Detection | **Verified** | `vision/signboard/SignBoardAnalyzer.kt` | Visual exit detection & luminance contrast analysis |
| | Hazard Risk Engine & Priority Haptics | **Verified** | `vision/HazardRiskEngine.kt`, `HapticAlertManager.kt` | Dynamic multi-tier vibration alerts based on approach speed |
| **Voice Interaction** | Multilingual Voice Command Suite | **Verified** | `voice/DefaultVoiceCommands.kt`, `VoiceRouter.kt` | Offline deterministic parsing across English, Hindi, and Marathi |
| | On-Demand Microphone Coordination | **Verified** | `voice/MicrophoneCoordinator.kt`, `VoiceRecognizerManager.kt` | Audio focus management & priority audio capture |
| | Intent Routing & Domain Classification | **Verified** | `voice/VoiceRouter.kt`, `ActionRegistry.kt` | Routes between Camera, Places, Device Settings, and Gemini AI |
| | Supabase Edge Function AI Fallback | **Verified** | `supabase/functions/voice-ai/`, `VoiceQuestionHandler.kt` | General conversational Q&A routed to `gemini-1.5-flash` |
| | Custom Wake Word ("Hello Drishti") | **Blocked** | `wakeword/models/hello_drishti_float.tflite` | Synthetic Edge-TTS dataset only (0 human samples); blocked from prod |
| **Smart Cane (Hardware)** | ESP32-CAM Dual-Core Firmware | **Verified** | `hardware/esp32_cam_drishti/esp32_cam_drishti.ino` | FreeRTOS tasking, camera frame grab, VL53L0X I2C loop |
| | VL53L0X Time-of-Flight Lidar | **Verified** | `hardware/esp32_smart_cane/esp32_smart_cane.ino` | 10 Hz obstacle ranging with millimeter precision |
| | Autonomous Handle Haptics & Buzzer | **Verified** | Hardware bench circuit | Local vibration/buzzer triggers independently of mobile connection |
| | BLE GATT Telemetry (NUS Service) | **Verified** | `hardware/Esp32BleManager.kt`, firmware GATT | Nordic UART Service (UUID `6E400001-...`) distance & battery sync |
| | Authenticated Wi-Fi MJPEG Stream | **Verified** | `hardware/Esp32CamManager.kt`, firmware `/stream` | HTTP MJPEG stream protected by dynamic Bearer token authentication |
| **Telephony & Calls** | Broadcast Incoming Call Detection | **Verified** | `call/IncomingCallDetector.kt`, `DrishtiCallManager.kt` | `PHONE_STATE` receiver with `goAsync()` self-bootstrapping |
| | On-Device Caller Resolution | **Verified** | `call/CallerIdentityResolver.kt` | Local `ContactsContract.PhoneLookup` query; 0 cloud data leakage |
| | Spoken Caller Announcement | **Verified** | `call/CallerAnnouncementManager.kt` | Priority TTS caller announcement (Name or Unsaved Number) |
| | Ringing Command Listening Mode | **Verified** | `voice/DrishtiWakeWordService.kt` | Automatic wake-word bypass during ringing state |
| | Voice Call Control (Pick/Reject/End) | **Verified** | `call/CallVoiceController.kt` | 20/20 test cases verified on Vivo V2338 (EN, HI, MR) |
| | In-Call Audio Controls (Mute/Speaker) | **Verified** | `call/CallVoiceController.kt` | `AudioManager` mute & `setCommunicationDevice` speaker toggles |
| | Local Spam & Fraud Rule Classifier | **Partially Implemented** | `call/spam/LocalSpamRepository.kt`, `SpamRuleEngine.kt` | Rule heuristics implemented; full national database sync pending |
| **Navigation & Mobility** | Indoor-to-Outdoor Visual Egress | **Verified** | `navigation/indoor/`, `INDOOR_TO_OUTDOOR_...report.md` | Optical doorway tracking handing off to Google Maps walking route |
| | Google Places (New) Nearby Search | **Verified** | `search/PlaceSearchEngine.kt` | Resolves Hospitals, Pharmacies, ATMs via live device GPS |
| | Turn-by-Turn Outdoor Guidance | **Verified** | `navigation/NavigationEngine.kt`, `NavigationScreen.kt` | Google Maps Compose SDK walking route guidance & spoken maneuvers |
| | Saved Home Location & Compass Return | **Verified** | `homemapping/HomeMapRepository.kt`, `saved_places` table | Saved home GPS coordinate persistence and heading guidance |
| | Multi-Floor Indoor Point-Cloud Mapping | **Partially Implemented** | `homemapping/scanner/HomeSpatialScanner.kt`, `HomeMap3DRenderer.kt` | Spatial scanner & 3D renderer implemented; multi-floor BLE beacons planned |
| **Calendar & Events** | Conversational Event Creation Flow | **Verified** | `events/EventConversationalFlow.kt`, `EventTimeParser.kt` | Multi-turn speech slot filling (Title, Date, Time, Reminders) |
| | Room Database Event Persistence | **Verified** | `db/dao/EventDao.kt`, `db/entity/EventEntities.kt` | Full local CRUD operations for calendar events and reminders |
| | Exact Alarm Scheduling & Wakeup | **Verified** | `events/EventAlarmScheduler.kt`, `EventAlarmReceiver.kt` | `AlarmManager.setExactAndAllowWhileIdle()` execution |
| | Full-Screen Accessible Alarm Overlay | **Verified** | `events/EventAlarmActivity.kt`, `EventAlarmService.kt` | High-contrast visual overlay with continuous TTS announcement |
| | Device Reboot Alarm Rescheduling | **Verified** | `events/EventRebootReceiver.kt` | `BOOT_COMPLETED` receiver restores pending alarms from Room |
| **Emergency SOS** | Grace Period Countdown & Cancellation | **Verified** | `sos/EmergencyManager.kt`, `EmergencyActivity.kt` | 5-second countdown with audible cadence and haptic ticks |
| | GPS & Cane Telemetry Session Payload | **Verified** | `db/entity/SosSessionEntity.kt`, `SosTelemetryEntity.kt` | Captures GPS coords, accuracy, cane distance, battery, timestamp |
| | National Emergency Dialing (112) | **Verified** | `sos/EmergencyManager.kt` | Telephony Intent integration to dial 112 directly |
| | SMS Fallback Dispatch | **Verified** | `sos/sms/EmergencySmsSender.kt` | Direct multi-part SMS dispatch to configured emergency contacts |
| | Two-Sided Guardian Live Telemetry | **Verified** | `final_runtime_audit_report.md` | 14/14 tests passed: Child device -> Supabase -> Parent Dashboard |
| | "I Am Safe" Resolution Flow | **Verified** | `sos/EmergencyManager.kt`, `end_emergency_session` RPC | Formally resolves active emergency status across all linked clients |
| **Guardian Management** | Role-Based Profiles (`ward` vs `parent`) | **Verified** | `public.profiles`, `010_secure_profile_...sql` | Database check constraints prevent unauthorized role escalation |
| | Cryptographic Pairing Code Generator | **Verified** | `generate_pairing_code` RPC | 6-character alphanumeric code hashed with SHA-256 and unique salt |
| | Rate-Limited Code Claiming | **Verified** | `submit_pairing_code` RPC, `pairing_attempt_tracker` | 10-minute expiry; 5 failed attempts trigger lock |
| | Ward-Sovereign Approval & Revocation | **Verified** | `respond_to_pairing_request` RPC, `012_fix_...sql` | Ward can approve, reject, or unilaterally revoke guardian access |
| | Cross-Tenant Row-Level Security (RLS) | **Verified** | `docs/database_security_rls_matrix.md` | 17/17 RLS checks passed; pending/unrelated users blocked (0 rows) |
| **Offline-First Data** | Local Room Relational Storage | **Verified** | `db/DrishtiDatabase.kt` | Persistent storage for SOS sessions, telemetry, and events |
| | Connectivity Monitoring & Outbox Sync | **Verified** | `sos/sync/SosSyncManager.kt` | `ConnectivityManager.NetworkCallback` triggers sync upon internet return |
| | Settings Sync & SharedPreferences Cache | **Verified** | `settings_functional_test_report.md` | 20/20 settings verified; sandboxed local storage with cloud sync |
| | Generalized WorkManager Background Queue | **Planned** | Roadmap milestone | Migrating standalone network listener to WorkManager periodic worker |
| **Accessibility & UI** | High-Contrast Accessible Design System | **Verified** | `ui/theme/Color.kt`, `Theme.kt` | Warm Sand (#F1DAC4), Technical Purple (#7C3AED), Graphite (#1F1F1F) |
| | Touch Target Dimensions ($\ge 48\,\text{dp}$) | **Verified** | Compose UI components | Large interactive surfaces compliant with Material 3 guidelines |
| | TalkBack Semantics & Focus Ordering | **Partially Implemented** | `ui/screens/` | Accessibility labels added; comprehensive external user audit pending |
| | Multi-Tier Haptic Feedback Engine | **Verified** | `vision/HapticAlertManager.kt` | Distinct vibration waveforms for warnings, confirmations, and alerts |

---

## 3. Non-Functional Requirements & Engineering Standards

### 3.1 Security & Privacy Bounds
1. **Zero External Frame Leakage:** Camera frames processed for obstacle and text detection must remain strictly on-device. No frame data shall be transmitted to external servers without explicit user confirmation.
2. **Contact Sandboxing:** Caller identification must resolve via local `ContactsContract` queries. No contact lists or phone numbers shall be uploaded to Supabase or third-party databases.
3. **RLS Tenancy Enforcement:** PostgreSQL Row-Level Security policies must strictly isolate telemetry and profile records. A parent with a `pending` or `revoked` status must receive 0 rows upon querying ward telemetry.
4. **Secret Protection:** Zero private API credentials, Supabase `service_role` keys, or production certificates shall be committed to git repositories or compiled into client APK assets.

### 3.2 Performance & Responsiveness Targets
1. **Camera Frame Processing Rate:** Maintain 6 to 10 FPS on-device inference using CameraX `STRATEGY_KEEP_ONLY_LATEST` backpressure handling without memory leakage or thermal throttling.
2. **Voice Intent Resolution Latency:** Local deterministic commands (Answer call, Stop navigation, SOS) must parse within $\le 150\,\text{ms}$. Cloud AI fallback queries via Supabase Edge Function must respond within $\le 2.0\,\text{s}$ under typical 4G/5G connections.
3. **Cane Obstacle Reaction Latency:** Embedded VL53L0X obstacle detection must trigger handle vibration within $\le 50\,\text{ms}$ of threshold breach, operating autonomously of smartphone connectivity.
4. **Offline Resilience:** The application must boot, accept voice commands, identify callers, read printed text, and schedule alarms when in full Airplane Mode.
