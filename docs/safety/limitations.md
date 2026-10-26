# Drishti AI — Safety Disclaimers, Hardware Bounds & System Limitations

## 1. Ethical Stance & Engineering Honesty

Assistive technology platforms must be uncompromisingly transparent about their real-world capabilities and failure modes. Misrepresenting experimental prototypes as certified life-safety instruments can lead to severe personal injury.

Drishti AI is developed under the foundational principle of **Engineering Honesty**: we document physical sensor limits, environmental boundaries, and software assumptions explicitly.

---

## 2. Hardware Sensor Boundaries & Physical Limits

### 2.1 VL53L0X Time-of-Flight Lidar Sensor
* **Rated Range:** Nominal ranging from $30\,\text{mm}$ to $2000\,\text{mm}$ ($2.0\,\text{m}$) in controlled indoor environments.
* **Ambient Solar Degradation:** The sensor operates via a $940\,\text{nm}$ infrared VCSEL laser. In bright outdoor sunlight, ambient infrared radiation elevates the noise floor of the SPAD array, reducing effective range to approximately $1.0\,\text{m}$ to $1.2\,\text{m}$.
* **Specular & Transparent Surfaces:** Clear glass doors, clean mirrors, and extremely dark matte surfaces (absorptive black velvet) can cause beam refraction or total absorption, resulting in late or missed obstacle detections.
* **Angular Cutoff:** The sensor field of view is approximately $25^\circ$. Obstacles positioned outside this cone (such as objects approaching rapidly from extreme lateral angles) will not trigger the handle vibration motor.

### 2.2 Smartphone Camera & Monocular Depth Estimation
* **Perspective Estimation vs. True Depth:** Standard commercial smartphones (including the test Vivo V2338) lack active hardware ToF depth arrays. Camera distance measurements are mathematical optical estimations derived from bounding box heights. **They must never be relied upon for millimeter-precise clearance.**
* **Illumination Requirements:** Optical OCR and object detection require adequate ambient lighting ($\ge 50\,\text{lux}$). In pitch darkness or heavy fog, camera-based perception will degrade or fail entirely.

---

## 3. Location & Network Limitations

### 3.1 GPS Precision & Urban Canyon Drift
* Standard mobile satellite GNSS typically delivers horizontal accuracy between $\pm 5\,\text{m}$ and $\pm 15\,\text{m}$ under clear skies.
* In dense high-rise urban areas ("urban canyons"), signal reflection against glass towers can cause multipath drift up to $\pm 50\,\text{m}$.
* GPS satellite signals do not penetrate indoor concrete structures. Indoor navigation must rely on visual landmarks and egress detection rather than satellite coordinates.

### 3.2 Emergency Delivery Dependencies
An in-app SOS broadcast or emergency SMS is subject to external telecom variables:
1. **Cellular Coverage:** If the device is located in an underground basement or rural dead zone with zero signal, SMS and cloud uploads are staged in the local Room outbox and delayed until network restoration.
2. **Device Power:** A depleted phone battery stops all software services. (Note: The ESP32 smart cane continues autonomous vibration sensing on its independent battery).
3. **OS Battery Optimization:** Aggressive third-party OEM background managers (such as extreme power saving on certain custom Android distributions) can terminate background receivers if the user fails to grant "Unrestricted Battery" permissions.

---

## 4. Formal Assistive Disclaimer

> [!CAUTION]
> **ASSISTIVE AID DISCLAIMER**  
> Drishti AI is an assistive intelligence platform designed to augment environmental awareness and communication.  
> 
> * **It is NOT a medical device** certified under FDA, CE-MDR, or ISO 13485 standards.
> * **It does NOT replace** formal Orientation and Mobility (O&M) training with a certified instructor.
> * **It does NOT replace** a standard white mobility cane for physical ground tactile feedback, drop-off detection, or curb identification.
> * Users must continue to exercise independent situational awareness, tactile cane sweeping, and personal caution at all times.
