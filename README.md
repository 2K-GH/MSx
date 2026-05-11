# MSx (Mobile Suppression Experimental)

**MSx** is a federated, high-mobility Electronic Countermeasures (ECM) platform designed for localized 2.4GHz physical-layer interference research. By utilizing a 4WD carrier, MSx is optimized to analyze the impact of high-output interference on target hardware to test the robustness of embedded IoT nodes and frequency-hopping spread spectrum (FHSS) mitigations.

---

## ⚖️ Legal & Ethical Disclaimer

> **NOTICE**: This repository is for **Educational and Security Research purposes only**.
> * All development and validation of this platform were conducted within a controlled, shielded laboratory environment.
> * The author does not condone the unauthorized disruption of licensed communication networks.
> * This tool is intended for civil defense strategies and infrastructure hardening research. Full compliance with local telecommunications regulations is the sole responsibility of the user.

---

## 🛠 Distributed System Architecture

MSx utilizes a **Distributed Load** strategy to ensure real-time stability across three independent microcontrollers. Source code for individual modules can be found in their respective repositories:

| Node | Module | Role | Hardware | Repository |
| :--- | :--- | :--- | :--- | :--- |
| **C2 Gateway** | GDx | Vision / WebSocket Interface | ESP32-CAM | [2K-GH/GDx](https://github.com/2K-GH/GDx) |
| **Execution** | GDx | Motor & Gimbal control | Arduino Uno | [2K-GH/GDx](https://github.com/2K-GH/GDx) |
| **Effector** | Jx | Physical-Layer Disruption | Gizduino (Arduino-compatible) | [2K-GH/Jx](https://github.com/2K-GH/Jx) |

---

## 📡 Spectral Engineering: The Channel 9 "Sweet Spot"

To prevent the Jx module from "self-jamming" the MSx control link, the GDx (ESP32-CAM) is reconfigured to operate on **Wi-Fi Channel 9**.

### **The Math: Coexistence via Spectral Gapping**
The Jx effector targets standard non-overlapping Wi-Fi channels using an nRF24L01+ set to a **2MBPS data rate**, which creates an approximate **2MHz spread** per transmission.

**Targeted Frequencies ($F = 2400 + \text{Register Value}$):**
1. **Wi-Fi Ch 1:** 2412 MHz (Register 12)
2. **Wi-Fi Ch 6:** 2437 MHz (Register 37)
3. **Wi-Fi Ch 11:** 2462 MHz (Register 62)

### **Comparative Gap Analysis: Why Ch 9 (Gap 6–11) vs. Gap 1–6?**
While a gap exists between Channels 1 and 6, the selection of **Channel 9 (2452 MHz)** is mathematically superior for a high-bandwidth MJPEG stream.

#### **1. Spectral Clearance Comparison**
* **The Ch 1–6 Gap (Center 2424 MHz):** Provides a **+2 MHz** margin from Ch 1 and a **+3 MHz** margin from Ch 6.
* **The Ch 9 Sweet Spot (2452 MHz):** Provides a massive **+5 MHz** safety margin from the high-power Ch 6 interference.
* **Verdict:** The 5MHz margin provides 2.5x more spectral isolation, significantly reducing Adjacent Channel Interference (ACI).

#### **2. Power Density & SINR Management**
We optimize the **Signal-to-Interference-plus-Noise Ratio (SINR)** by exploiting Wi-Fi's native resilience:
* **Asymmetric Protection:** While Ch 9's edge sits at $2462\text{ MHz}$ (Ch 11), it isolates the **critical center frequency** and the **entire lower sideband** in a "Quiet Zone".
* **OFDM Resilience:** Wi-Fi can survive edge interference via Forward Error Correction (FEC). In contrast, the narrow Ch 1–6 gap "squeezes" the signal from both sides, which would lead to catastrophic video jitter.

#### **3. Antenna Resonance & Physics**
Standard 2.4GHz "rubber ducky" antennas are typically tuned for a center resonant frequency of **2450 MHz**.
* **Wavelength Calculation:** $\lambda = \frac{c}{f} \approx \frac{3 \times 10^8}{2.45 \times 10^9} \approx 12.24 \text{ cm}$.
* **Resonance:** $2452\text{ MHz}$ (Channel 9) aligns almost perfectly with the antenna's design center, ensuring a lower **Standing Wave Ratio (SWR)** and higher effective radiated power compared to Channel 1 ($2412\text{ MHz}$).

---

## ⚡ Power Subsystem: Single-Source 30C Integration

MSx runs entirely from a **single high-discharge power source** (7.4V 1400mAh **30C** LiPo). The 30C rating is critical for preventing voltage sag during the **115mA peak bursts** of the Jx module and simultaneous motor operation.

---

## 📸 Hardware Gallery

| MSx Front Profile | GDx Vision Interface |
| :---: | :---: |
| <img src="pictures/MSx_GDx.jpg" width="450"> | <img src="pictures/MSx_GDx2.jpg" width="450"> |
| 4WD mobile carrier housing the integrated layout of the federated control nodes. | Front-facing view centering the ESP32-CAM gateway for real-time C2. |

<div align="center">
  <br>
  <img src="pictures/MSx_Jx.jpg" width="600" style="border-radius: 8px; border: 1px solid #333;">
  <p><b>Jx Payload:</b> The high-gain nRF24L01+ (PA+LNA) effector responsible for localized interference.</p>
</div>

---

## **License**
This project is released under the **MIT License**.
