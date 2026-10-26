# Drishti AI — Computer Vision, OCR & Environmental Perception

## 1. Vision Architecture Overview

Drishti AI’s vision pipeline transforms raw camera frames into structured auditory descriptions, spatial hazard warnings, and spoken text reading without streaming video to external cloud infrastructure.

```mermaid
flowchart LR
    Camera[CameraX ImageAnalysis] -->|YUV_420_888 @ 6-10 FPS| Pipeline[PerceptionPipeline.kt]
    Pipeline --> OCR[OcrRecognitionEngine (ML Kit)]
    Pipeline --> ObjectDet[ObjectDetectionEngine (ML Kit)]
    Pipeline --> Depth[DepthEstimator (Optical Pinhole)]
    
    OCR --> Dedup[OcrDeduplicationManager]
    Dedup --> TTS[TTS Spoken Reading]
    
    ObjectDet --> Hazard[HazardRiskEngine]
    Depth --> Hazard
    
    Hazard --> Haptics[HapticAlertManager]
    Hazard --> AudioAlert[Directional Audio Cues]
```

---

## 2. On-Device Text Recognition (OCR) & Deduplication

### 2.1 OCR Recognition Engine (`OcrRecognitionEngine.kt`)
* Powered by Google **ML Kit Text Recognition v2** running on-device via Android Neural Networks API (NNAPI) / GPU delegates.
* Supports printed Latin scripts, Hindi (Devanagari), and Marathi.
* Capable of recognizing book passages, medicine labels, nutrition cards, storefront signage, and office room placards.

### 2.2 Temporal Deduplication (`OcrDeduplicationManager.kt`)
A primary failure mode of naive camera OCR for visually impaired users is **speech stuttering**: when pointing a camera at a sign, the same words are detected 10 times a second, flooding the user with repeated speech.

Drishti AI resolves this with a temporal sliding window and Jaccard token similarity:
$$\text{Similarity}(A, B) = \frac{|A \cap B|}{|A \cup B|}$$
* New text segments are normalized, lowercase-tokenized, and compared against recently spoken segments within a 6-second window.
* If similarity exceeds $0.70$, the reading is suppressed as a duplicate.
* If the user shifts the camera to a new paragraph or sign, similarity falls below $0.40$ and the new content is read aloud immediately.

---

## 3. Object Detection & Perspective Depth Estimation

### 3.1 On-Device Object Detection
* Uses Google ML Kit Object Detection configured for real-time tracking with classification.
* Identifies prominent environmental entities: `PERSON`, `VEHICLE`, `CHAIR`, `TABLE`, `DOOR`, `STAIRS`.
* Maintains temporal track IDs (`ObjectTrackManager.kt`) to monitor approaching obstacles vs receding entities.

### 3.2 Perspective Pinhole Distance Estimation (`DepthEstimator.kt`)
Because standard smartphones (such as the target Vivo V2338) lack active hardware Time-of-Flight lidar sensors and ARCore depth engines, Drishti AI implements monocular perspective geometry based on the camera pinhole model:

$$d_{\text{est}} = \frac{H_{\text{real}} \cdot f_y}{h_{\text{box}}}$$

* Where $H_{\text{real}}$ is the canonical real-world height of the classified object (e.g., adult human $\approx 1.7\,\text{m}$, standard chair $\approx 0.85\,\text{m}$, interior door $\approx 2.1\,\text{m}$).
* $f_y$ is the vertical focal length extracted from Camera2 `CameraCharacteristics.LENS_INFO_AVAILABLE_FOCAL_LENGTHS`.
* $h_{\text{box}}$ is the bounding box height in pixels.

> [!NOTE]
> **Engineering Honesty Disclaimer:** Distance derived from monocular camera frames is strictly labeled as **ESTIMATED** (e.g., *"Person detected, approximately 3 meters ahead"*). Millimeter precision is reserved exclusively for the hardware ESP32 cane equipped with physical VL53L0X time-of-flight sensors.

---

## 4. Hazard Risk Engine & Dynamic Haptics

### 4.1 Hazard Risk Scoring (`HazardRiskEngine.kt`)
Obstacle risk is computed as a function of object velocity, center-offset angle $\theta$, and distance:

$$\text{RiskScore} = \left(\frac{1}{d_{\text{est}}}\right) \cdot \cos(\theta) \cdot (1 + v_{\text{approach}})$$

* **Tier 1 (Notice):** Obstacle detected at $3.0\,\text{m}$ to $5.0\,\text{m}$. Spoken audio notice queued if requested.
* **Tier 2 (Caution):** Obstacle in direct path at $1.5\,\text{m}$ to $3.0\,\text{m}$. Gentle single-pulse haptic vibration.
* **Tier 3 (Immediate Danger):** Obstacle in direct path at $< 1.2\,\text{m}$. High-frequency continuous haptic vibration and urgent auditory tone.

### 4.2 Haptic Alert Profiles (`HapticAlertManager.kt`)
Uses Android `VibrationEffect.createWaveform()` for distinct tactile signatures:
* **Confirmation Pulse:** Single short tick ($50\,\text{ms}$).
* **Warning Pulse:** Double sharp pulse ($100\,\text{ms}$ on, $100\,\text{ms}$ off, $100\,\text{ms}$ on).
* **Collision Alert:** Fast repeating rumble ($200\,\text{ms}$ interval) until obstacle clears the hazard zone.
