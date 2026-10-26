# Drishti AI — Conflict Resolution & Data Reconciliation Policies

## 1. Domain-Specific Reconciliation Policies

Because offline-first applications permit local modifications during network disconnections, data divergence can occur when reconnecting. Drishti AI defines explicit, deterministic conflict-resolution policies tailored to each data category:

```text
+---------------------------------------------------------------------------------+
|                       CONFLICT RESOLUTION STRATEGY BY DOMAIN                    |
+---------------------------------------------------------------------------------+
| 1. Emergency & Telemetry Logs | Immutable Append-Only Stream (No Overwrites)    |
| 2. User & System Settings     | Last-Write-Wins (LWW) with Monotonic Timestamps  |
| 3. Relationship Permissions   | Ward Sovereignty Precedence (Ward Overrules All)|
| 4. Pairing & Delivery Queues  | Idempotent Deduplication (Unique Constraints)   |
+---------------------------------------------------------------------------------+
```

---

## 2. Policy Specifications

### 2.1 Emergency & Telemetry Data: Immutable Append-Only Stream
* **Policy:** Telemetry breadcrumbs (`public.emergency_telemetry`) and session events (`public.emergency_sessions`) are **immutable**.
* **Reconciliation:** Telemetry points are tagged with local hardware capture timestamps (`captured_at`). When syncing after an outage, points are appended into the database time-series without overwriting existing data. If duplicate points are received due to network retries, database unique indices on `(session_id, captured_at)` discard the duplicate safely.

### 2.2 User Settings & Preferences: Last-Write-Wins (LWW)
* **Policy:** Configuration settings (`user_settings`, `navigation_preferences`) resolve via **Last-Write-Wins** using UTC millisecond timestamps:
  ```sql
  INSERT INTO public.user_settings (user_id, speech_rate, volume, updated_at)
  VALUES ($1, $2, $3, $4)
  ON CONFLICT (user_id) DO UPDATE
  SET speech_rate = EXCLUDED.speech_rate,
      volume = EXCLUDED.volume,
      updated_at = EXCLUDED.updated_at
  WHERE EXCLUDED.updated_at > public.user_settings.updated_at;
  ```
* **Reconciliation:** If settings were modified offline on the device while modified remotely on the web dashboard, the record with the more recent `updated_at` timestamp takes precedence.

### 2.3 Guardian Authorization: Ward Sovereignty Precedence
* **Policy:** In any conflict between a parent action and a ward action, **the ward's action unconditionally wins**.
* **Reconciliation:** If a parent submits a pairing code while offline or attempts to claim a revoked session, the server-side trigger evaluates the ward's latest relationship status. If the ward has marked `status = 'revoked'` or `'rejected'`, incoming parent updates are rejected with an explicit database exception.

### 2.4 Idempotency in Pairing & Outbox Queues
* **Policy:** Network retransmissions must never duplicate state changes or generate multiple relationships.
* **Reconciliation:** Stored procedures use atomic transactions and conditional check blocks (`WHERE NOT EXISTS`). Duplicate pairing code submissions for an already approved relationship return the existing relationship record without resetting or altering approval timestamps.
