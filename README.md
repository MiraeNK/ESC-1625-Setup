# 🧠 ESC WP‑1625 (Brushed) Control via ESP32 (GPIO 22)

This project demonstrates how to control the Hobbywing **QuicRun WP‑1625 brushed ESC** using an **ESP32** and PWM signals (GPIO 22). It includes the full arming sequence, forward/reverse logic, and insights into ESC signal processing.

---

## 🔧 Hardware & Components

- **ESP32 Dev Board**
- **Hobbywing QuicRun WP‑1625 Brushed ESC**
  - 25 A continuous, 100 A peak output
  - Supports 2–3S LiPo or 5–9 NiMH batteries :contentReference[oaicite:1]{index=1}
  - Waterproof/dustproof for rough RC use :contentReference[oaicite:2]{index=2}
  - Default Fwd / Brake / Rev mode (double‑click reverse) :contentReference[oaicite:3]{index=3}
- **Brushed DC Motor**
- **Power source** (battery matching ESC spec)
- **Signal wire** from ESC to **GPIO 22** on ESP32

---

## ⚙️ How PWM Signal Controls the ESC

The WP‑1625 interprets **RC‑style PWM signals (≈50 Hz)** on its signal pin. Pulse widths map to throttle commands:

| Pulse Width (µs) | Throttle Action     |
|------------------|---------------------|
| ~1000            | Full Reverse (50%)  |
| ~1250            | Brake / transition  |
| ~1500            | Neutral / stop      |
| ~2000            | Full Forward        |

### 🚦 Arming Sequence  
On power-up, the ESC expects a neutral (~1500 µs) signal to arm safely—otherwise it will beep repeatedly :contentReference[oaicite:4]{index=4}.

---

## 🔁 Drive Sequence (Code Logic)

1. **Neutral (Arming)** – 1500 µs, ESP32 waits >3 s
2. **Forward** – ~1750 µs (≈50% forward throttle)
3. **Brake** – 1250 µs; first reverse signal acts as brake
4. **Neutral Pause** – 1500 µs
5. **Reverse** – 1250 µs again to actually move backward
6. **Full Stop** – back to 1500 µs  
*Required “Brake → Neutral → Reverse” sequence ensures safety and proper behavior in Fwd/Br/Rev mode :contentReference[oaicite:5]{index=5}.*

---

## 🛡️ ESC Built-in Protections

- **Low Voltage Cut-off**: reduces output at threshold, disables when too low :contentReference[oaicite:6]{index=6}
- **Over-temperature shut-off** at ~100 °C, resumes below ~80 °C :contentReference[oaicite:7]{index=7}
- **Signal Loss Protection**: stops output after ~0.1 s without valid signal :contentReference[oaicite:8]{index=8}

---

## 🛠️ Code Overview (`main.cpp`)

```cpp
esc.attach(22, 1000, 2000);            // PWM on GPIO 22
esc.writeMicroseconds(1500);           // Arming (neutral)
delay(4000);                           // Wait for ESC arm beep

esc.writeMicroseconds(1750); delay(3000); // Forward ~50%
esc.writeMicroseconds(1250); delay(1000); // Brake
esc.writeMicroseconds(1500); delay(1500); // Neutral
esc.writeMicroseconds(1250); delay(3000); // Reverse ~50%
esc.writeMicroseconds(1500); delay(5000); // Full stop, repeat
