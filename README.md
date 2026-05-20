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

## 📡 Spectral Engineering: Synchronized Channel Hopping

Jx targets all three standard non-overlapping 2.4 GHz channels in rotation. A static GDx channel assignment — including the previously evaluated Ch 9 "sweet spot" — was found to be insufficient once the nRF24L01+ PA+LNA module was paired with a high-gain external antenna. The elevated Total Radiated Power (TRP) widened the spectral skirts of each transmission to the point where no single fixed channel maintained a reliable margin across all three Jx cycles.

The solution is a **Synchronized Deterministic Channel Hop** — GDx mirrors Jx's EEPROM-driven rotation index and stays permanently one step ahead, guaranteeing a minimum 25 MHz center-to-center separation on every power cycle.

### **Jx Effector — Target Frequencies**
The Jx effector uses an nRF24L01+ at **RF24_2MBPS** with continuous `0xAA` pattern transmission.

**Targeted Frequencies ($F = 2400 + \text{Register Value}$):**
1. **Wi-Fi Ch 1:** 2412 MHz (Register 12)
2. **Wi-Fi Ch 6:** 2437 MHz (Register 37)
3. **Wi-Fi Ch 11:** 2462 MHz (Register 62)

Jx stores its current channel index in **EEPROM address 0**, incrementing on every power cycle. The rotation is deterministic and persistent across power loss.

### **GDx C2 Gateway — Hop Logic**
GDx (ESP32-CAM) maintains a parallel index in **ESP32 NVS (Non-Volatile Storage)**, incrementing in lockstep with Jx on every shared power cycle. The AP channel is selected from an offset table that ensures GDx is never on the same channel Jx is currently targeting.

**Hop Table:**

| Power Cycle | Jx EEPROM Index | Jx Targets | GDx NVS Index | GDx AP Channel | Separation |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 0 (first boot) | 0 | Ch 1 — 2412 MHz | 0 | **Ch 6 — 2437 MHz** | 25 MHz |
| 1 | 1 | Ch 6 — 2437 MHz | 1 | **Ch 11 — 2462 MHz** | 25 MHz |
| 2 | 2 | Ch 11 — 2462 MHz | 2 | **Ch 1 — 2412 MHz** | 50 MHz |
| 3 | 0 | Ch 1 — 2412 MHz | 0 | **Ch 6 — 2437 MHz** | loops |

**Key implementation details:**
* No wiring between Jx and GDx is required. Synchronization is achieved purely through shared power — both devices boot simultaneously from the same LiPo source, read their respective persistent counters, and select their channel independently.
* GDx uses `AP_CHANNELS[] = {6, 11, 1}` — index N of this array always resolves to the channel Jx is **not** targeting at index N.
* The same out-of-range guard (`>= NUM_CHANNELS → clamp to 0`) used in Jx's EEPROM logic is mirrored in GDx's NVS read to prevent desync on corrupted storage.
* HT20 (20 MHz) bandwidth is enforced on the GDx AP via `esp_wifi_set_bandwidth()` to minimize the GDx occupied footprint.

### **Visual Channel Identification**
GDx blinks its camera flash LED on boot to confirm the active AP channel, mirroring Jx's onboard LED pattern:

| Flash Blinks | GDx AP Channel | Jx Targeting |
| :---: | :---: | :---: |
| 1 blink | Ch 1 — 2412 MHz | Ch 11 |
| 2 blinks | Ch 6 — 2437 MHz | Ch 1 |
| 3 blinks | Ch 11 — 2462 MHz | Ch 6 |

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
