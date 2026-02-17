# Project Report: Laser Security System with Buzzer & SMS Alert

**Author:** Thirumala Rao Bommineni  
**Department:** Electronics & Communication Engineering  
**Institution:** Alturi Mastan Reddy Memorial College of Engineering & Technology

---

## 1. Abstract

This project presents a low-cost, reliable perimeter security system that uses laser light technology to detect intruders. A laser beam is continuously transmitted across a protected area. If any person or object breaks the beam, a buzzer immediately sounds and an SMS alert is sent to the owner's mobile phone via a GSM module. The system can be scaled to cover large areas using multiple laser-LDR pairs or mirror-reflection techniques.

---

## 2. Problem Statement

Traditional security systems (CCTV, motion sensors) are expensive, require power-hungry components, and may miss slow-moving intruders. There is a need for a simple, affordable, and highly responsive perimeter security solution for homes, warehouses, and restricted areas.

---

## 3. Proposed Solution

A laser transmitter continuously fires an invisible or visible beam at an LDR receiver. The Arduino monitors the LDR continuously. The moment the beam is broken, the system immediately activates a buzzer and sends an SMS to the owner — providing instant dual-alert (local + remote).

---

## 4. System Architecture

```
┌──────────────┐   Laser Beam   ┌──────────────┐
│   LASER      │════════════════│  LDR / Photo │
│  TRANSMITTER │                │   RECEIVER   │
└──────────────┘                └──────┬───────┘
                                       │ Analog signal
                                ┌──────▼───────┐
                                │  Arduino UNO │
                                │  (Controller)│
                                └──────┬───────┘
                                  ┌────┴────┐
                              ┌───▼───┐ ┌───▼────┐
                              │Buzzer │ │SIM800L │
                              │(Local)│ │(Remote │
                              │ Alert │ │  SMS)  │
                              └───────┘ └────────┘
```

---

## 5. Working Principle

| LDR Reading | Beam Status | System State | Action |
|:-----------:|:-----------:|:------------:|--------|
| HIGH (≥400) | Beam intact | ARMED / SAFE | No action |
| LOW (<400) | Beam broken | ALERT | Buzzer ON + SMS sent |
| HIGH again (restore) | Beam restored | RE-ARMED | Buzzer OFF, reset |

---

## 6. Key Features

- **Instant detection** — < 100ms response from beam break to buzzer
- **Dual alert** — Local buzzer (immediate) + SMS to owner (remote)
- **SMS cooldown** — Prevents SMS spam by enforcing a 30-second cooldown
- **Auto re-arm** — System resets automatically when beam is restored
- **Scalable** — Multiple beams can cover an entire room or warehouse
- **Low cost** — Total component cost < ₹500

---

## 7. Components Used

| Component | Specification | Purpose |
|----------|--------------|---------|
| Arduino UNO | ATmega328P, 16MHz | Main controller |
| Laser Module | 5mW, 650nm Red | Transmit laser beam |
| LDR Sensor | GL5528 or equivalent | Detect laser beam |
| Resistor | 10kΩ | Voltage divider with LDR |
| Active Buzzer | 5V | Local audible alarm |
| SIM800L GSM | Quad-band 850/900/1800/1900 MHz | SMS alert to owner |
| Status LED | 5mm (onboard D13) | Visual system status |

---

## 8. Results

- System detects beam interruption in under 100ms
- SMS delivered to owner within 5–8 seconds of intrusion
- SMS cooldown prevents flooding the owner's phone
- Successfully tested with single and multi-beam configurations
- False alarm rate near zero in indoor environments

---

## 9. Applications

- Home and office perimeter security
- Museum and artifact protection
- Warehouse and factory boundary monitoring
- Server room and restricted area access control
- ATM and vault protection systems

---

## 10. Future Scope

1. **Camera Integration** — Capture an image using ESP32-CAM on intrusion and send via WhatsApp
2. **Multi-zone Detection** — Identify which beam was broken (zone mapping)
3. **Mobile App Dashboard** — Real-time alert dashboard using Firebase + Flutter
4. **Solar Powered** — Make the system fully autonomous with solar panel
5. **Voice Alert** — Add a voice module to announce "Intruder Detected"
6. **Password-Protected Arm/Disarm** — Keypad to arm or disarm the system

---

## 11. References

1. Arduino Documentation — https://www.arduino.cc/reference/en/
2. SIM800L AT Command Manual — https://simcom.ee/documents/
3. NPTEL IoT Course Materials
4. Embedded Systems Design — Rajkamal

---

*This project was developed as part of the B.Tech ECE curriculum with a focus on Embedded Systems and IoT.*
