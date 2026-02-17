# 🔴🔒 Laser Security System with Buzzer & SMS Alert

> **A laser beam guards your premises — the moment someone crosses it, a buzzer sounds instantly and an SMS is sent directly to your phone.**

An Arduino-based perimeter security system that uses a **laser transmitter** and **LDR receiver** to create an invisible tripwire across any area. If the beam is broken by an intruder, a local **buzzer alarm** triggers immediately and a **GSM SMS alert** is sent to the owner's mobile phone — providing real-time dual protection.

---

## 📌 Project Overview

| Feature | Detail |
|--------|--------|
| **Platform** | Arduino UNO |
| **Transmitter** | Laser Module (5mW, 650nm Red Laser) |
| **Receiver** | LDR (Light Dependent Resistor) |
| **Local Alert** | Active Buzzer — sounds immediately on intrusion |
| **Remote Alert** | SIM800L GSM Module — SMS sent to owner's phone |
| **Response Time** | < 100ms from beam break to buzzer activation |
| **SMS Cooldown** | 30 seconds (prevents message flooding) |
| **Auto Re-arm** | System resets automatically when beam is restored |
| **Coverage** | Expandable with multiple beams or mirror reflection |

---

## 🧠 How It Works:


### 🔐 The Idea — A Laser Tripwire

You've probably seen laser security systems in movies — a thief tries to sneak through a room full of criss-crossing laser beams and sets off an alarm. This project does exactly that, using real, affordable components.

The core idea is simple: **a laser beam travels from one side of the area to the other**. As long as the beam reaches the other side safely, everything is normal. The moment something — a person, an animal, an object — **steps into the beam and blocks it**, the system detects that the beam is gone and triggers an alert.

---

### 💡 Two Sides of the System

The system has exactly **two sides** that work as a team:

```
  TRANSMITTER SIDE                       RECEIVER SIDE
  ─────────────────                      ─────────────────

  [ Laser Module ]  ════════════════►  [ LDR Sensor ]
   Sends laser beam                     Receives laser beam
   continuously                         and reports to Arduino
```

**Transmitter (Laser Module):**
- Continuously fires a focused red laser beam across the protected area
- Powered by Arduino's D7 pin so it can be controlled by software
- The beam is straight, narrow, and travels in a perfect line

**Receiver (LDR — Light Dependent Resistor):**
- Placed directly opposite the laser, facing it
- The LDR is a special component whose **electrical resistance changes based on how much light it receives**
- When laser light hits it → resistance is LOW → voltage reading is HIGH
- When laser light is blocked → resistance is HIGH → voltage reading is LOW

This change in voltage is read by Arduino through the Analog pin A0.

---

### 🔴 What is an LDR and How Does It Work?

An **LDR (Light Dependent Resistor)** is a sensor that reacts to light intensity:

```
  Bright light on LDR:                  No light on LDR:
  ┌──────────────────┐                  ┌──────────────────┐
  │ Resistance LOW   │                  │ Resistance HIGH  │
  │ (few hundred Ω)  │                  │ (mega ohms MΩ)   │
  │ Voltage at A0:   │                  │ Voltage at A0:   │
  │  HIGH (~1023)    │                  │  LOW (~0–200)    │
  └──────────────────┘                  └──────────────────┘
```

We use this property to detect the laser beam:
- **Laser hits LDR → A0 reads HIGH value (≥ 400)** → Beam is intact → SAFE
- **Laser blocked → A0 reads LOW value (< 400)** → Beam is broken → ALERT!

The number 400 is the **threshold** — you can calibrate this for your environment.

---

### 🚨 What Happens When the Beam is Broken?

The moment the Arduino detects a LOW reading from the LDR (beam broken), it kicks off a two-part alert system:

#### Part 1 — Local Alert (Buzzer)
The buzzer at the protected location starts sounding immediately. It first beeps 3 rapid times as a warning burst, then stays ON continuously until the beam is restored. Anyone nearby (security guard, family member) can hear it right away.

#### Part 2 — Remote Alert (SMS via GSM)
The Arduino commands the SIM800L GSM module to send an SMS to the owner's registered phone number. The message reads:

```
SECURITY ALERT! Laser beam broken at your premises.
Possible intrusion detected. Please check immediately.
```

This reaches the owner's phone within 5–8 seconds, no matter where they are — even if they're kilometres away.

---

### ⏱️ The SMS Cooldown — Why It Matters

Without a cooldown, if an intruder stays in the beam for 30 seconds, the system could try to send 300 SMS messages — flooding the owner and draining the SIM balance rapidly.

So the system has a **30-second SMS cooldown**: after sending one SMS, it waits at least 30 seconds before sending another one. The buzzer stays ON throughout, but the SMS is sent once per event.

```
  Beam broken at 10:00:00 PM  →  Buzzer ON + SMS sent
  Intruder still present      →  Buzzer stays ON, no repeat SMS (cooldown)
  10:00:30 PM (30s passed)    →  If still broken, another SMS is sent
  Beam restored               →  Buzzer OFF, system re-arms
```

---

### 🔄 Complete System Flow (Step by Step)

```
                   ┌───────────────────────┐
                   │     SYSTEM STARTS     │
                   │  Laser ON, GSM Ready  │
                   └──────────┬────────────┘
                              │
                   ┌──────────▼────────────┐
                   │  READ LDR at A0       │
                   │  (every 100ms)        │
                   └──────────┬────────────┘
                              │
           ┌──────────────────┴──────────────────┐
           │                                     │
  ┌────────▼────────┐                   ┌────────▼────────┐
  │  A0 < 400       │                   │  A0 ≥ 400       │
  │  BEAM BROKEN    │                   │  BEAM INTACT    │
  └────────┬────────┘                   └────────┬────────┘
           │                                     │
  ┌────────▼────────┐                   ┌────────▼────────┐
  │  Trigger Alert  │                   │  System SAFE    │
  │  Buzzer ON      │                   │  LED solid ON   │
  │  SMS (if ready) │                   │  No action      │
  └────────┬────────┘                   └─────────────────┘
           │
  ┌────────▼────────┐
  │  Wait & Monitor │
  │  Beam restored? │
  └────┬───────┬────┘
       │ YES   │ NO
  ┌────▼───┐  ┌▼──────────────┐
  │ Buzzer │  │ Keep Buzzer ON │
  │  OFF   │  │ SMS cooldown?  │
  │Re-arm  │  │ If yes → SMS   │
  └────────┘  └───────────────┘
```

---

### 📋 Detection Logic Table

| LDR Reading (A0) | Beam Status | System State | Buzzer | SMS Sent |
|:----------------:|:-----------:|:------------:|:------:|:--------:|
| ≥ 400 (HIGH) | ✅ Intact | ARMED / SAFE | OFF | No |
| < 400 (LOW) — first detect | ❌ Broken | ALERT | ON | Yes |
| < 400 (LOW) — ongoing | ❌ Broken | ALERT | ON | Only if cooldown passed |
| ≥ 400 (restored) | ✅ Intact | RE-ARMED | OFF | No |

---

### 📡 How the GSM Module Sends SMS

The **SIM800L** is a small GSM (mobile network) module that can make calls and send SMS messages using a regular SIM card. It communicates with Arduino using **AT commands** — a set of simple text instructions.

```
  Arduino sends:    AT+CMGF=1             ← Set SMS to text mode
  Arduino sends:    AT+CMGS="+91XXXXX"    ← Recipient number
  Arduino sends:    Message text...       ← The SMS content
  Arduino sends:    [Ctrl+Z]              ← Signal to send the SMS
  SIM800L responds: +CMGS: 1             ← SMS sent successfully!
```

This entire process takes about 3–5 seconds, after which the owner receives the SMS on their phone.

---

### 🏠 Covering a Full Area — Multi-Beam Setup

A single beam creates one tripwire. To secure an entire room, doorway, or perimeter, you can use:

**Option 1 — Multiple Parallel Beams:**
```
  [Laser 1] ════════════════════════════ [LDR 1]   ← Top
  [Laser 2] ════════════════════════════ [LDR 2]   ← Middle
  [Laser 3] ════════════════════════════ [LDR 3]   ← Bottom

  (Connect LDR 1 → A0, LDR 2 → A1, LDR 3 → A2)
  Any beam broken → ALERT
```

**Option 2 — Mirror Reflection (Single Laser, Full Perimeter):**
```
  [Laser] ══════► [Mirror 1]
                      ║
                  [Mirror 2] ◄══════ [Mirror 3]
                                          ║
                  [LDR] ◄═══════════ [Mirror 4]

  One laser zig-zags around the entire room using cheap mirrors!
```

---

## 🔧 Hardware Components

| Component | Quantity | Purpose |
|----------|:--------:|---------|
| Arduino UNO | 1 | Main microcontroller — brain of the system |
| Laser Module (5mW) | 1 | Transmit focused laser beam across protected area |
| LDR Sensor | 1 | Receive the laser beam (1 per beam line) |
| Resistor (10kΩ) | 1 | Voltage divider for LDR reading |
| Active Buzzer | 1 | Local audible alarm on intrusion |
| SIM800L GSM Module | 1 | Send SMS to owner's mobile number |
| SIM Card (any network) | 1 | Required for GSM module to send SMS |
| Li-Ion Battery (3.7V 2A) | 1 | Dedicated power for SIM800L (not Arduino 5V!) |
| Resistor (220Ω) | 1 | For status LED |
| Jumper Wires | Several | Electrical connections |
| Breadboard | 1 | Prototyping |
| Mounting Brackets / Pipe | 2 | Align laser and LDR precisely opposite each other |

---

## 🔌 Circuit Connections

### Pin Connection Table

| Component | Component Pin | Arduino Pin | Role |
|-----------|:------------:|:-----------:|------|
| Laser Module | IN / Signal | **D7** | OUTPUT — Arduino controls laser ON/OFF |
| Laser Module | VCC | 5V | Power |
| Laser Module | GND | GND | Ground |
| LDR Sensor | Leg 1 | **A0** | Analog input — light level reading |
| LDR Sensor | Leg 2 | GND | Ground |
| 10kΩ Resistor | Between A0 & GND | A0 ↔ GND | Voltage divider |
| Active Buzzer | + (positive) | **D8** | OUTPUT — alarm signal |
| Active Buzzer | – (negative) | GND | Ground |
| SIM800L GSM | TX | **D10** | Software Serial — receive from GSM |
| SIM800L GSM | RX | **D11** | Software Serial — send to GSM |
| SIM800L GSM | VCC | External 3.7–4.2V | **Separate supply — NOT Arduino 5V!** |
| SIM800L GSM | GND | GND (shared) | Common ground |
| Status LED | + (via 220Ω) | **D13** | Blinks during alert, solid when armed |

### ASCII Circuit Diagram

```
TRANSMITTER SIDE                         RECEIVER SIDE
─────────────────                        ─────────────────

 [Laser Module]      Laser Beam           [LDR Sensor]
   VCC → 5V    ════════════════════════►   Leg 1 → A0
   GND → GND                              Leg 2 → GND
   IN  → D7                              [10kΩ]  A0 → GND


                      [Arduino UNO]
                   ┌──────────────────────┐
          D7  ────►│ Laser Control        │
          A0  ◄────│ LDR Reading          │
          D8  ────►│ Buzzer Alarm         │
          D10 ◄────│ GSM TX               │
          D11 ────►│ GSM RX               │
          D13 ────►│ Status LED           │
          GND ─────│ Common Ground        │
          5V  ─────│ Laser + LDR Power    │
                   └──────────────────────┘

                      [SIM800L GSM]            [Buzzer]
                   ┌─────────────────┐      ┌──────────┐
                   │ VCC → 3.7–4.2V  │      │ +  → D8  │
                   │ GND → GND       │      │ –  → GND │
                   │ TX  → D10       │      └──────────┘
                   │ RX  → D11       │
                   └─────────────────┘
                   ⚠️ Use separate Li-Ion
                     NOT Arduino 5V!
```

> ⚠️ **IMPORTANT:** The SIM800L GSM module can draw up to **2 Amperes** during SMS transmission. Powering it from Arduino's 5V pin will damage the Arduino. Always use a **dedicated 3.7V Li-Ion battery** with its GND connected to Arduino's GND (shared ground).

---

## ⚙️ Calibrating the Laser Threshold

Before deploying, you must calibrate the **LASER_THRESHOLD** value for your specific environment:

**Step 1** — Open `laser_security.ino` and set:
```cpp
const int LASER_THRESHOLD = 400;  // Starting value
```

**Step 2** — Upload the code and open **Serial Monitor** at 9600 baud.

**Step 3** — With the laser beam intact and hitting the LDR, note the LDR reading in the serial monitor. It might show something like `LDR Reading: 780`.

**Step 4** — Set the threshold to about **50–100 less** than that value:
```cpp
const int LASER_THRESHOLD = 680;  // (780 - 100 = 680)
```

**Step 5** — Re-upload. Now block the beam with your hand — the reading should drop below 680, triggering the alert. ✅

---

## 🚀 Getting Started

### Step 1 — Clone the Repository
```bash
git clone https://github.com/thirumala-rao/laser-security-system.git
cd laser-security-system
```

### Step 2 — Set Your Phone Number
Open `src/laser_security.ino` and update this line with your mobile number:
```cpp
const char OWNER_PHONE[] = "+91XXXXXXXXXX";  // ← Your number here
```

### Step 3 — Assemble the Circuit
Wire components as shown in the Circuit Connections section. Use a breadboard first. Insert an active SIM card into the SIM800L module.

### Step 4 — Open in Arduino IDE
- Open `src/laser_security.ino` in the **Arduino IDE**
- Go to **Tools → Board** → select **Arduino UNO**
- Go to **Tools → Port** → select your COM port

### Step 5 — Upload & Calibrate
- Click **Upload** (→)
- Open **Serial Monitor** at **9600 baud**
- Follow the calibration steps above
- Test by blocking the laser beam with your hand

---

## 📟 Serial Monitor Output (Sample)

```
============================================
   Laser Security System — Initializing
============================================
[GSM] Initializing SIM800L...
[GSM] GSM Module Ready.
[SYSTEM] Armed and monitoring...
[SYSTEM] Laser threshold set to: 400
--------------------------------------------
[SENSOR] LDR Reading: 812 ✅ Beam OK
[SENSOR] LDR Reading: 809 ✅ Beam OK
[SENSOR] LDR Reading: 47  ⚠️  BEAM BROKEN!
[ALERT] INTRUSION DETECTED! Triggering buzzer...
[GSM] Sending SMS alert to owner...
[GSM] SMS sent successfully!
[SENSOR] LDR Reading: 31  ⚠️  BEAM BROKEN!
[SENSOR] LDR Reading: 798 ✅ Beam OK
[SYSTEM] Beam restored. System re-armed.
```

---

## 📁 Project Structure

```
laser-security-system/
│
├── src/
│   └── laser_security.ino         ← Main Arduino sketch
│
├── circuit/
│   └── circuit_diagram.md         ← Detailed pin & wiring reference
│
├── docs/
│   └── project_report.md          ← Academic project report
│
├── README.md                      ← This file (complete documentation)
└── LICENSE                        ← MIT License
```

---

## 🔭 Future Improvements

- [ ] **ESP32-CAM Integration** — Capture photo of intruder and send via WhatsApp/Telegram
- [ ] **Multi-Zone Detection** — Identify which beam was broken (Zone 1, Zone 2, etc.)
- [ ] **Mobile App Dashboard** — Real-time alerts and arm/disarm via Firebase + Flutter app
- [ ] **Password-Protected Arm/Disarm** — 4x4 keypad to arm or disarm without triggering alarm
- [ ] **Voice Announcement** — Add DFPlayer Mini voice module to announce "Intruder Detected"
- [ ] **Solar Powered** — Fully autonomous outdoor system with solar panel + battery
- [ ] **Night Vision Mode** — IR LED + photodiode instead of visible laser for covert setup
- [ ] **WhatsApp Alert** — Replace SMS with WhatsApp using Twilio API via ESP8266

---

## 👤 Author

**Thirumala Rao Bommineni**  
B.Tech — Electronics & Communication Engineering  
Alturi Mastan Reddy Memorial College of Engineering & Technology  
Specialization: Embedded Systems · IoT · VLSI  
· 📧 thirumal.ofcl@gmail.com

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

> ⭐ If this project helped you or you found it useful, consider giving it a star on GitHub!
