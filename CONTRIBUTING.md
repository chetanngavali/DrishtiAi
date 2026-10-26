# Contributing to Drishti AI

Thank you for contributing to **Drishti AI**! By participating in this project, you help build a safer, more accessible world for visually impaired individuals.

---

## 1. Our Core Engineering Principles

1. **Accessibility First:** Every screen, button, and interaction must be 100% accessible via Android TalkBack, provide large touch boundaries ($\ge 48\times 48\,\text{dp}$), and support multi-modal feedback (Visual, Auditory TTS, and Haptics).
2. **Offline-First Resilience:** Assistive features (obstacle sensing, OCR, call controls, alarms) must operate without an internet connection. Write to local storage (Room) first, then synchronize asynchronously.
3. **Privacy & Security by Design:** Zero video streaming to external clouds. Contact resolution must execute locally. All database tables must enforce Row-Level Security (RLS).
4. **Engineering Honesty:** Accurately document feature readiness. Clearly distinguish between code that is **implemented & empirically verified**, **partially implemented**, **planned**, or **blocked**. Never make unsupported claims of compliance.

---

## 2. Documentation Architecture

Before contributing code, review the architecture specifications in the [`/docs`](docs/) directory:
* [Project Overview](docs/project-overview.md) & [Requirements Inventory](docs/requirements-and-features.md)
* [System Architecture](docs/system-architecture.md) & [Android App Architecture](docs/android-app/app-architecture.md)
* [Voice Assistant](docs/android-app/voice-assistant.md) & [Hands-Free Calling](docs/android-app/call-management.md)
* [Smart Cane Hardware](docs/hardware/smart-cane-overview.md) & [VL53L0X ToF Lidar](docs/hardware/esp32-vl53l0x.md)
* [Supabase Database Schema](docs/backend/database-schema.md) & [RLS Security Matrix](docs/backend/authentication-and-roles.md)
* [Offline Synchronization](docs/offline-first/synchronization.md) & [Testing Strategy](docs/testing/test-strategy.md)

---

## 3. How to Submit Contributions

### 3.1 Reporting Bugs
* Use clear, descriptive issue titles.
* Specify the target device model and Android version (e.g., Vivo V2338 / Android 16).
* Include reproduction steps and ADB logcat snippets.
* **Security Notice:** Do NOT report vulnerabilities via public issues; see [SECURITY.md](SECURITY.md).

### 3.2 Pull Request Workflow
1. Fork the repository and create your feature branch:
   ```bash
   git checkout -b feature/accessible-voice-intent
   ```
2. Adhere to the **Kotlin Style Guide** and Jetpack Compose best practices.
3. Add unit tests for all new business logic or parsers.
4. Verify that existing tests pass:
   ```bash
   ./gradlew testDebugUnitTest
   ```
5. **Secret Audit:** Ensure no API tokens, signing keys, private phone numbers, or credentials exist in your diff.
6. Open a Pull Request with a clear summary of changes, test evidence, and any updated documentation.

---

## 4. Code of Conduct

Please review our [Code of Conduct](CODE_OF_CONDUCT.md) to ensure an inclusive, respectful environment for all contributors.

---

*By contributing to Drishti AI, you agree that your contributions will be licensed under the [MIT License](LICENSE).*
