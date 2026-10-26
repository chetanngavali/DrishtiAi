# Changelog — Drishti AI

All notable changes to the Drishti AI assistive mobility platform are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0-rc1] — 2026-10-26

### Added
* **Two-Sided Live Telemetry Pipeline:** End-to-end streaming from physical Vivo V2338 child handset to remote Supabase PostgreSQL and live Google Maps parent dashboard (14/14 empirical verification tests passed).
* **Voice-Driven Calendar & Event Scheduling:** Multi-turn conversational event creation (`EventConversationalFlow.kt`), natural language time parsing (`EventTimeParser.kt`), Room persistence (`EventDao.kt`), exact `AlarmManager` wakeups, accessible full-screen overlay (`EventAlarmActivity.kt`), and boot restoration receiver (`EventRebootReceiver.kt`).
* **Hands-Free Telephony & Call Control:** Autonomous incoming call detection via `PHONE_STATE` receiver with `goAsync()` self-bootstrapping, zero-cloud on-device caller identification via `ContactsContract.PhoneLookup`, automatic ringing command-listening mode, and voice controls (Answer, Reject, End, Mute, Speaker) in English, Hindi, and Marathi (20/20 test scenarios passed on physical hardware).
* **Indoor-to-Outdoor Visual Egress:** Camera-assisted optical doorway and portal tracking with monocular perspective pinhole depth estimation and smooth handoff to Google Maps walking navigation.
* **Offline Outbox Synchronization:** Local Room relational outbox (`DrishtiDatabase.kt`) coupled with network state monitoring (`ConnectivityManager.NetworkCallback` in `SosSyncManager.kt`) for automatic synchronization of emergency sessions and telemetry upon internet reconnection.
* **Complete Settings & Policy Audit:** Inventory and downstream wiring for 20 user settings (speech rate, TTS volume, haptic feedback, collision distance, language selection), verified across 472 unit tests and 4 instrumented device tests.
* **Cryptographic Guardian Pairing:** Server-side 6-character pairing code generation with SHA-256 salted digest storage, 10-minute expiry, 5-attempt rate-limiting, and ward revocation sovereignty.
* **Comprehensive Documentation Suite:** 26 modular architecture, hardware, security, testing, and operational guides housed in `/docs`.

### Changed
* **Conversational AI Gateway:** Configured Supabase `voice-ai` Edge Function to use Google's `gemini-1.5-flash` model, ensuring low-latency conversational responses suitable for Text-to-Speech output.
* **Cane Obstacle Sensing:** Upgraded hardware ranging from legacy ultrasonic sensors to the STMicroelectronics VL53L0X Time-of-Flight (ToF) laser lidar (10 Hz sampling with 3-sample median filter).
* **Database Architecture:** Deployed and verified 21 application tables and 8 security-definer RPC stored procedures on Supabase PostgreSQL 15 with 100% Row-Level Security (RLS) enforcement.

### Security
* **Adversarial Security Audit:** Executed automated penetration suite with 62 security assertions across 5 actor personas (`ward`, `approved parent`, `pending parent`, `unrelated user`, `anon`), confirming 100% cross-tenant isolation.
* **Secret Protection:** Ensured zero private API keys, signing keystores, or Supabase service-role credentials exist within the client APK or public git repository.

### Blocked
* **Hands-Free Wake Word ("Hello Drishti"):** The `hello_drishti_float.tflite` model was evaluated and determined to be trained on synthetic Edge-TTS audio only (0 human samples). Formal production status marked as **BLOCKED** pending human speech corpus collection across diverse Indian accents. In the current release, voice commands operate reliably via on-demand microphone activation and ringing auto-listen mode.

---

## [0.9.0] — 2026-09-15

### Added
* CameraX preview and Google ML Kit on-device text recognition with basic spoken feedback.
* Initial ESP32 firmware prototype with single-zone handle vibration alert.
* Initial Supabase authentication setup and basic profile schemas.
* Emergency SOS button with 5-second countdown timer.

---

## [0.1.0] — 2026-08-01

### Added
* Initial project inception, accessibility vision, and native Kotlin Jetpack Compose repository setup.
