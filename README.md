# ☀️ Dual Axis Solar Tracker — Logic Gate Implementation

> A hardware-only, microcontroller-free solar tracking system using combinational logic gates, LDR sensors, and an L293D motor driver to orient a solar panel toward maximum light intensity on both the horizontal (azimuth) and vertical (elevation) axes.

---

## 📋 Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Circuit Components](#circuit-components)
- [Logic Design](#logic-design)
  - [Sensor Mapping](#sensor-mapping)
  - [Truth Tables](#truth-tables)
  - [Gate-Level Logic](#gate-level-logic)
- [IC Reference](#ic-reference)
- [L293D Motor Driver Wiring](#l293d-motor-driver-wiring)
- [Power Supply](#power-supply)
- [Simulation](#simulation)
- [Circuit Schematic Notes](#circuit-schematic-notes)
- [Repository Structure](#repository-structure)
- [Future Improvements](#future-improvements)
- [License](#license)

---

## Overview

This project implements a **dual-axis solar tracker** entirely using discrete logic gates — no microcontroller, no software. Four Light Dependent Resistors (LDRs) are placed at the corners of the solar panel frame. Their outputs are processed through a network of AND, OR, and NOT gates to generate direction-control signals for two DC motors via an **L293D H-bridge motor driver**.

The system continuously and autonomously corrects the panel orientation to minimize the difference in light intensity between opposing sensor pairs.

---

## How It Works

```
         SUN
          ↓
   ┌──────────────┐
   │  S1(TL) S2(TR)│   ← 4 LDR Sensors on panel corners
   │               │
   │  S3(BL) S4(BR)│
   └──────────────┘
          ↓
   [Logic Gate Network]
   AND + OR gates derive
   directional error signals
          ↓
      [L293D]
   H-Bridge Motor Driver
       ↙       ↘
  Motor A     Motor B
(Horizontal) (Vertical)
```

### Decision Logic

| Condition | Horizontal Motor | Vertical Motor |
|-----------|-----------------|----------------|
| Left side brighter (S1+S3 > S2+S4) | Rotate Left (CCW) | — |
| Right side brighter (S2+S4 > S1+S3) | Rotate Right (CW) | — |
| Top side brighter (S1+S2 > S3+S4) | — | Tilt Up |
| Bottom side brighter (S3+S4 > S1+S2) | — | Tilt Down |
| Balanced (all equal) | Stop | Stop |

---

## Circuit Components

| Ref | Component | Qty | Description |
|-----|-----------|-----|-------------|
| S1–S4 | LDR + Voltage Divider | 4 | Light sensors; output HIGH when bright |
| NOT0 (×4) | NOT Gate (e.g. 74HC04) | 4 | Signal inversion for each sensor input |
| U16:A, U16:B | 74HC21 | 1 | Dual 4-input AND gate |
| U8:A, U8:B | 74HC21 | 1 | Dual 4-input AND gate |
| U5:A, U5:C | 4073 | 1 | Triple 3-input AND gate |
| U6:A, U6:B, U6:C | 4073 | 1 | Triple 3-input AND gate |
| U10, U12, U13 | AND | — | Generic AND gates |
| U7:A | 4072 | 1 | Dual 4-input OR gate |
| U9 | OR | 1 | OR gate |
| U11:A, U14:A | 4075 | 1 | Triple 3-input OR gate |
| U15 | **L293D** | 1 | Quadruple H-Bridge Motor Driver |
| V1 | 9V Battery | 1 | Power supply |
| M1, M2 | DC Motor | 2 | Horizontal & Vertical axis motors |

---

## Logic Design

### Sensor Mapping

```
S1 = Top-Left     (A)
S2 = Top-Right    (B)
S3 = Bottom-Left  (C)
S4 = Bottom-Right (D)

Ā = NOT(S1),  B̄ = NOT(S2),  C̄ = NOT(S3),  D̄ = NOT(S4)
```

**Convention:** Signal is `HIGH (1)` when the corresponding LDR receives more light (lower resistance → higher voltage at divider output).

---

### Truth Tables

#### Horizontal Axis (Motor A — Azimuth)

Motor A drives the panel LEFT when the left sensors are brighter than the right sensors, and RIGHT when the right sensors are brighter.

| S1 (A) | S2 (B) | S3 (C) | S4 (D) | Motor_A_Left | Motor_A_Right |
|--------|--------|--------|--------|--------------|---------------|
| 1 | 0 | 1 | 0 | **1** | 0 |
| 1 | 0 | 0 | 0 | **1** | 0 |
| 0 | 0 | 1 | 0 | **1** | 0 |
| 0 | 1 | 0 | 1 | 0 | **1** |
| 0 | 1 | 0 | 0 | 0 | **1** |
| 0 | 0 | 0 | 1 | 0 | **1** |
| 1 | 1 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 | 0 |
| 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 1 | 1 | 1 | 0 | 0 |

#### Vertical Axis (Motor B — Elevation)

Motor B tilts the panel UP when the top sensors are brighter, and DOWN when the bottom sensors are brighter.

| S1 (A) | S2 (B) | S3 (C) | S4 (D) | Motor_B_Up | Motor_B_Down |
|--------|--------|--------|--------|------------|--------------|
| 1 | 1 | 0 | 0 | **1** | 0 |
| 1 | 0 | 0 | 0 | **1** | 0 |
| 0 | 1 | 0 | 0 | **1** | 0 |
| 0 | 0 | 1 | 1 | 0 | **1** |
| 0 | 0 | 1 | 0 | 0 | **1** |
| 0 | 0 | 0 | 1 | 0 | **1** |
| 1 | 0 | 1 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 0 | 0 |
| 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 1 | 1 | 1 | 0 | 0 |

---

### Gate-Level Logic

The combinational logic derives the four motor control outputs:

```
-- Horizontal Axis --
Motor_A_Left  = OR(A·B̄·C·D̄,  A·B̄·C̄·D̄,  Ā·B̄·C·D̄)
Motor_A_Right = OR(Ā·B·C̄·D,  Ā·B·C̄·D̄,  Ā·B̄·C̄·D)

-- Vertical Axis --
Motor_B_Up   = OR(A·B·C̄·D̄,  A·B̄·C̄·D̄,  Ā·B·C̄·D̄)
Motor_B_Down = OR(Ā·B̄·C·D,  Ā·B̄·C·D̄,  Ā·B̄·C̄·D)
```

Each AND-term (minterm) is implemented with a multi-input AND gate. The OR layer merges the qualifying minterms into a single control signal.

---

## IC Reference

### 74HC21 — Dual 4-Input AND Gate

```
         ┌────┐
  1A ─1 ─┤    ├─ 6 ─ 1Y
  1B ─2 ─┤    │
  1C ─4 ─┤    │
  1D ─5 ─┤74HC21
  2A ─9 ─┤    ├─ 8 ─ 2Y
  2B ─10─┤    │
  2C ─12─┤    │
  2D ─13─┤    │
  GND─7 ─┤    │
  VCC─14─┘    │
         └────┘
```

### 4073 — Triple 3-Input AND Gate
- Three independent 3-input AND gates in one package
- VCC: Pin 14, GND: Pin 7

### 4075 — Triple 3-Input OR Gate
- Three independent 3-input OR gates in one package

### 4072 — Dual 4-Input OR Gate
- Two independent 4-input OR gates

---

## L293D Motor Driver Wiring

```
         ┌─────────────────┐
  IN1 ───┤ 2     VS  ── 8 ├─── 9V Motor Supply
  IN2 ───┤ 7         VCC─16 ├─── 5V Logic Supply
  EN1 ───┤ 1     GND  ── 4 ├─── GND
  OUT1 ──┤ 3     GND  ── 5 ├─── GND
  OUT2 ──┤ 6               │
         │   (Motor A)     │
  IN3 ───┤ 10    OUT3 ──11 ├── Motor B Terminal 1
  IN4 ───┤ 15    OUT4 ──14 ├── Motor B Terminal 2
  EN2 ───┤ 9               │
         └─────────────────┘
```

| EN | IN1 | IN2 | Motor |
|----|-----|-----|-------|
| 1  |  1  |  0  | Forward (CW) |
| 1  |  0  |  1  | Reverse (CCW) |
| 1  |  1  |  1  | Brake |
| 1  |  0  |  0  | Coast |
| 0  |  X  |  X  | Disabled |

> **Note:** EN1 and EN2 are tied HIGH (VCC) in this design since the direction logic handles enabling implicitly.

---

## Power Supply

- **Logic supply (VCC):** 5V (from regulator or USB)
- **Motor supply (VS):** 9V battery (V1 in schematic)
- **GND:** Common ground between logic and motor supply

> ⚠️ Always connect logic GND and motor GND to the same reference point.

---

## Simulation

This circuit was designed and simulated in **Proteus Design Suite**. The `.pdsprj` simulation file is included in the `simulation/` folder.

### Running the Simulation

1. Open Proteus ISIS
2. Load `simulation/dual_axis_tracker.pdsprj`
3. Toggle switches S1–S4 to simulate LDR brightness levels
4. Observe motor direction changes on the virtual DC motors

### Test Cases

```
Test 1 — Left side bright:     S1=1, S2=0, S3=1, S4=0 → Motor A rotates LEFT
Test 2 — Right side bright:    S1=0, S2=1, S3=0, S4=1 → Motor A rotates RIGHT
Test 3 — Top side bright:      S1=1, S2=1, S3=0, S4=0 → Motor B tilts UP
Test 4 — Bottom side bright:   S1=0, S2=0, S3=1, S4=1 → Motor B tilts DOWN
Test 5 — Balanced (all HIGH):  S1=1, S2=1, S3=1, S4=1 → Both motors STOP
Test 6 — No light (all LOW):   S1=0, S2=0, S3=0, S4=0 → Both motors STOP
```

---

## Circuit Schematic Notes

| Item | Detail |
|------|--------|
| Simulation tool | Proteus ISIS |
| Gate family | Mix of CMOS (4000 series) and HC TTL (74HC series) |
| Logic levels | 5V CMOS compatible |
| Motor driver | L293D (up to 600 mA per channel) |
| No. of ICs | ~7 ICs + motor driver |
| Power | Single 9V battery + optional 5V regulator |

---

## Repository Structure

```
dual-axis-solar-tracker/
│
├── README.md                        ← This file
├── docs/
│   ├── circuit-analysis.md          ← Detailed gate-level analysis
│   ├── truth-tables.md              ← Full expanded truth tables
│   ├── component-datasheet-links.md ← Datasheets for all ICs
│   └── images/
│       └── schematic.png            ← Circuit schematic screenshot
│
├── simulation/
│   └── dual_axis_tracker.pdsprj     ← Proteus simulation file
│
├── hardware/
│   ├── BOM.csv                      ← Bill of Materials
│   └── pcb-layout-notes.md          ← Notes for PCB implementation
│
└── LICENSE
```

---

## Future Improvements

- [ ] **Hysteresis band:** Add a deadband comparator so motors don't hunt at equilibrium
- [ ] **PWM speed control:** Replace binary ON/OFF with PWM for smoother tracking
- [ ] **Limit switches:** Add mechanical end-stops to prevent over-rotation
- [ ] **Night mode:** Add a timer or overall light threshold to park the panel at sunset
- [ ] **PCB design:** Convert breadboard prototype to a custom PCB using KiCad
- [ ] **Arduino version:** Add a microcontroller branch for comparison / logging
- [ ] **Efficiency logging:** Add a solar panel voltage/current monitor to log gain from tracking

---

## License

MIT License — see `LICENSE` for details.

---

*Built with ❤️ using only logic gates. No microcontrollers were harmed.*
