# Drishti AI Android Application — Architecture & Implementation

## 1. Technical Stack & Environment

* **Primary Language:** Kotlin 1.9.0
* **Target SDK:** 37 (Android 16 / Funtouch OS validated)
* **Minimum SDK:** 28 (Android 9.0 Pie)
* **UI Framework:** Jetpack Compose with Material 3 Design Tokens
* **Concurrency:** Kotlin Coroutines (`Dispatchers.Main`, `Dispatchers.IO`, `SupervisorJob`) and Reactive State (`StateFlow`, `SharedFlow`)
* **Local Relational Database:** Android Jetpack Room (v2.6+)
* **Networking & Cloud Integration:** Supabase Kotlin Multiplatform Client (GoTrue Auth, PostgREST, Realtime, Storage) & OkHttp 4
* **Computer Vision & OCR:** Google ML Kit (Text Recognition v2 & Object Detection) and CameraX 1.3+
* **Telephony & Audio:** Android `TelecomManager`, `TelephonyCallback` (API 31+), `PhoneStateListener`, `AudioManager`, and Android `TextToSpeech`

---

## 2. Layered Architectural Design

Drishti AI follows Clean Architecture and MVVM (Model-View-ViewModel) with strict Unidirectional Data Flow (UDF):

```text
+---------------------------------------------------------------------------------+
|                               Jetpack Compose UI                                |
| (HomeScreen, NavigationScreen, ScanScreen, ParentHomeScreen, EmergencyActivity) |
+---------------------------------------------------------------------------------+
                                       |
                         User Events   |   UI State (StateFlow)
                                       v
+---------------------------------------------------------------------------------+
|                                   ViewModels                                    |
|   (HomeViewModel, ParentDashboardViewModel, EmergencyViewModel, SettingsVM)     |
+---------------------------------------------------------------------------------+
                                       |
                         Suspend Calls |   Flow Subscriptions
                                       v
+---------------------------------------------------------------------------------+
|                                Repository Layer                                 |
|  (SupabaseRepository, HomeMapRepository, VoiceCommandRepository, EventDao)      |
+---------------------------------------------------------------------------------+
                      /                                    \
       Local Data    /                                      \  Remote Data
                    v                                        v
+-----------------------------+               +-----------------------------------+
|     Room Database & Cache   |               |         Supabase Backend          |
|  - DrishtiDatabase.kt       |               |  - PostgREST REST API             |
|  - EventDao.kt              |               |  - GoTrue Auth                    |
|  - SosSessionDao.kt         |               |  - Realtime WebSockets            |
|  - SharedPreferences Cache  |               |  - Edge Functions (voice-ai)      |
+-----------------------------+               +-----------------------------------+
```

---

## 3. Background Services & Broadcast Receivers

To guarantee assistive and emergency reliability even when the UI is closed or backgrounded, Drishti AI implements dedicated native services and receivers:

### 3.1 Background & Foreground Services
* **`DrishtiWakeWordService` (Foreground Service):**
  * Maintains an active foreground notification adhering to Android 14+ foreground service types (`microphone`, `connectedDevice`).
  * Coordinates real-time microphone acquisition via `MicrophoneCoordinator`.
  * Bootstraps the `DrishtiCallManager` and `CallerAnnouncementManager` dynamically to handle incoming calls without latency.
* **`EventAlarmService` (Foreground Service):**
  * Invoked by `EventAlarmReceiver` upon firing of an exact scheduled calendar alarm.
  * Aquires wake-locks to guarantee CPU execution while the screen is off.
  * Triggers full-screen visual intent (`EventAlarmActivity`) and drives high-priority TTS voice alerts.

### 3.2 Broadcast Receivers
* **`IncomingCallDetector`:**
  * Registered in `AndroidManifest.xml` with the `android.intent.action.PHONE_STATE` intent filter.
  * Uses `goAsync()` to transition broadcast processing onto an asynchronous coroutine, preventing ANRs while resolving contact records from `ContactsContract`.
  * Dispatches incoming ring events to `DrishtiCallManager` and transitions the voice engine into `CALL_COMMAND_LISTENING` mode.
* **`EventAlarmReceiver`:**
  * Catches exact wake-up intents from Android's `AlarmManager.setExactAndAllowWhileIdle()`.
  * Dispatches alarm payloads directly to `EventAlarmService`.
* **`EventRebootReceiver`:**
  * Catches `android.intent.action.BOOT_COMPLETED` broadcasts.
  * Queries `EventDao.getAllUpcomingEvents()` from Room and immediately reschedules exact alarms in the system `AlarmManager` after device restarts.

---

## 4. Concurrency & Asynchronous Design

All operations are strictly scheduled across appropriate thread pools:
* **`Dispatchers.Main.immediate`:** Fast UI updates, Compose state transitions, and immediate view rendering.
* **`Dispatchers.Default`:** Heavy CPU calculations, ML Kit frame normalization, and audio feature extraction.
* **`Dispatchers.IO`:** Room database read/write queries, Supabase HTTPS requests, BLE GATT socket writes, and `ContactsContract` queries.
* **`SupervisorJob` Scopes:** Background network listeners and sync managers use isolated supervisor jobs to ensure a failed network packet does not cancel the parent service scope.

---

## 5. Runtime Permission Architecture

Drishti AI features a centralized `PermissionCenterScreen` and runtime permission handler that degrades functionality gracefully when non-critical permissions are withheld:

| Permission | Criticality | Fallback Behavior When Denied |
| :--- | :--- | :--- |
| `RECORD_AUDIO` | **Critical** | Voice assistant disabled; UI displays high-contrast buttons for manual touch navigation. |
| `CAMERA` | **High** | OCR and obstacle vision disabled; smart cane BLE distance sensing remains operational. |
| `ACCESS_FINE_LOCATION` | **High** | GPS turn-by-turn guidance unavailable; indoor camera egress and compass heading remain active. |
| `READ_PHONE_STATE` | **Medium** | Automated caller announcement disabled; incoming calls ring normally through Android system UI. |
| `READ_CONTACTS` | **Medium** | Caller name lookup disabled; TTS announces raw phone number (`"Incoming call from +91..."`). |
| `ANSWER_PHONE_CALLS` | **Medium** | Spoken call answering disabled; TTS announces caller name, user must pick up manually on screen. |
| `SEND_SMS` | **Emergency** | SOS falls back to direct telephony 112 dialing and internet Supabase telemetry upload. |
