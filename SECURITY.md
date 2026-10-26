# Security Policy & Responsible Disclosure — Drishti AI

## 1. Security Overview & Philosophy

Drishti AI protects users who rely on the platform for physical mobility and personal safety. The platform enforces strict technical boundaries across cloud databases, Android client memory, and embedded firmware to protect user privacy, physical location data, and identity records.

---

## 2. Supported Versions

Security updates and patches are actively applied to the following release branches:

| Version Branch | Release Status | Security Support Level |
| :--- | :--- | :--- |
| **`v1.0.x` (RC-1)** | Release Candidate | **Active Security Support** (High Priority) |
| **`< 1.0.0`** | Alpha / Prototype | Deprecated; upgrade to Release Candidate |

---

## 3. Reporting a Vulnerability

We welcome responsible security research from developers, accessibility advocates, and cybersecurity professionals.

If you identify a security vulnerability, privacy leak, or authentication bypass:

1. **Do NOT open a public GitHub Issue.**
2. Send an email with complete reproduction details to:  
   **`security@drishtiai.org`** (or directly contact project maintainer Chetan Gavali at `chetan.gavali.dev@gmail.com`).
3. Include the following in your report:
   * Description of the vulnerability and its potential impact.
   * Step-by-step reproduction instructions or proof-of-concept payload.
   * Affected component (e.g., Android Client, Supabase Database RLS, ESP32 Firmware).
4. **Response Timelines:**
   * **Acknowledgment:** Within **48 hours** of report receipt.
   * **Initial Assessment & Triage:** Within **5 business days**.
   * **Remediation & Patch Deployment:** Target within **14 business days**.
   * We request that reporters maintain confidentiality until a patch is released to safeguard visually impaired users.

---

## 4. Core Security Controls & Architecture

Drishti AI is designed around four strict security guarantees:

### 4.1 Zero Public Secrets
* **Rule:** Private API keys, signing keystores (`*.jks`), Supabase `service_role` keys, and production service credentials must never be committed to source repositories or compiled into client binaries.
* **Implementation:** Client builds reference solely public anon tokens (`SUPABASE_ANON_KEY`) constrained by Row-Level Security. Private keys (e.g., Google Gemini AI keys) are isolated inside backend Supabase Edge Functions.

### 4.2 Contact & Video Privacy
* **Rule:** Camera frames and personal phone contacts must remain strictly on-device.
* **Implementation:** On-device OCR (ML Kit) and caller identification (`ContactsContract.PhoneLookup`) execute locally on CPU/NPU hardware without transmitting images, video streams, or address books to cloud servers.

### 4.3 Database Row-Level Security (RLS)
* **Rule:** Multi-tenant access to location and emergency data is constrained at the PostgreSQL kernel layer.
* **Implementation:** All 21 application tables enforce RLS policies. Unapproved or pending guardians receive 0 rows upon querying ward telemetry. Wards maintain the absolute right to unilaterally revoke access at any time.

### 4.4 Cryptographic Pairing Protection
* **Rule:** Pairing codes must resist brute-force enumeration and database leaks.
* **Implementation:** Codes are 6-character uppercase alphanumeric tokens hashed using SHA-256 with a unique cryptographic salt. Codes expire after 10 minutes, and 5 consecutive failed attempts trigger an automated account lockout.

---

## 5. Contributor Security Guidelines

When submitting pull requests or updating documentation:
* Exclude personal phone numbers, test GPS coordinates from private residences, and real user credentials from test fixtures and markdown documents.
* Ensure all database migrations include explicit `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;` statements.
* Verify that new API calls do not expose plaintext tokens in Android logcat outputs.
