# 📄 OMR Machine Using Digital Logic Devices

<div align="center">

![University](https://img.shields.io/badge/University-BUET-red)
![Simulation](https://img.shields.io/badge/Simulation-Proteus-orange)
![Hardware](https://img.shields.io/badge/Implementation-Breadboard-yellow)

**A fully hardware-based Optical Mark Recognition (OMR) system built using digital logic ICs, no microcontroller, no software.**

</div>

---


## 📖 About

This project implements a functional **Optical Mark Recognition (OMR)** machine using only fundamental digital logic ICs — no microcontroller, no FPGA, no software. A custom LDR-based scanner feeds signals into a purely hardware logic circuit that stores correct answers, evaluates a student's responses, and displays the score on a 7-segment display.

### Why this matters

Commercial OMR machines rely on expensive proprietary hardware or complex software. This project shows that the same core functionality can be achieved entirely with basic digital ICs, making it an ideal hands-on platform for learning digital electronics.


## 🏗️ System Architecture

The system is divided into three functional units:

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  STORAGE UNIT   │     │  SCANNING UNIT   │     │  EVALUATION UNIT    │
│                 │     │                  │     │                     │
│  8× 74HC175     │────▶│  5× LDR Sensors  │────▶│  74HC85 Comparator  │
│  Quad D FF      │     │  74N14 Schmitt   │     │  74HC151 MUX × 4    │
│  DIP Switch     │     │  RC Debouncing   │     │  74HC4040 Counter   │
│                 │     │                  │     │  4511 BCD Decoder   │
│  Stores correct │     │  Detects marks   │     │  7-Segment Display  │
│  answers (32 b) │     │  on OMR sheet    │     │  Shows score 0–8    │
└─────────────────┘     └──────────────────┘     └─────────────────────┘
```

### Unit 1 — Answer Storage

Eight **74HC175** quad D flip-flops (U1–U8) store the correct answer key for 8 MCQ questions. Each flip-flop holds one question's 4-bit pattern (options A, B, C, D). The correct option is stored as logic `0` and incorrect options as logic `1`. A DIP switch loads the answer key before scanning.

**Example:** If option B is correct for question 1 → stored as `A=1, B=0, C=1, D=1`

### Unit 2 — LDR Scanner

A hand-built cardboard scanner houses **5 LDRs**:

- 4 LDRs scan options A, B, C, D simultaneously
- 1 LDR detects the always-filled question marker circle to increment the question counter
- Each LDR feeds a **74N14 Schmitt trigger inverter** for clean digital output
- RC circuit (100Ω + 10µF) provides debouncing per channel

**Detection logic:** A filled (darkened) circle blocks light → LDR resistance increases → output = `1`. An unfilled circle lets light through → output = `0`.

### Unit 3 — Evaluation

Four **74HC151** 8-to-1 MUXes serialize stored answers one question at a time. A 3-bit question counter (driven by the 5th LDR) controls the MUX select lines. The serialized answer is compared against the scanned answer by a **74HC85** 4-bit comparator. On a match, a correct-answer counter (**74HC4040**) increments and the score is decoded to the 7-segment display via a **4511** BCD decoder.

---

## 🔌 Component List

| Component | Part Number | Qty | Function |
|---|---|---|---|
| Quad D Flip-Flop | 74HC175 | 8 | Answer storage (1 per question) |
| 8-to-1 Multiplexer | 74HC151 / 74150 | 4 | Answer serialization (1 per option) |
| 4-bit Comparator | 74HC85 / 7485 | 1 | Answer matching |
| Ripple Counter | 74HC4040 / CD4040 | 4 | Question counter + score counter |
| Schmitt Trigger Inverter | 74N14 | 1 | LDR signal conditioning |
| BCD to 7-Seg Decoder | 4511 | 1 | Display driver |
| 7-Segment Display | Common cathode | 1 | Score output (0–8) |
| Light Dependent Resistor | LDR | 5 | Optical sensing |
| DIP Switch | — | 1 | Answer key input |
| Breadboard | — | 7 | Circuit assembly |
| Resistors | 100Ω, 10kΩ | Various | Pull-up, current limiting |
| Capacitors | 10µF | 5 | RC debouncing per LDR channel |

## 💻 Simulation

The full circuit is simulated in **Proteus Design Suite**.

### Opening the Simulation

1. Install **Proteus** (version 8.x or later recommended)
2. Open `simulation/OMR.pdsprj`
3. Click **Run** (play button) to start simulation
4. Use the DIP switch to set the answer key, then click the flip-flop clock buttons to latch each answer
5. Toggle the push buttons to simulate LDR scanning signals

---

## 🚀 How to Use the Hardware

**Step 1 — Store the answer key:**
Power on and reset all flip-flops, MUXes, and counters. Use the DIP switch to set each question's correct answer bit, then press the clock button to latch it. Repeat for all 8 questions.

**Step 2 — Prepare the OMR sheet:**
Fill answer circles completely with black ink or dark pencil. The 5th circle (question marker) on every row must always be filled. Partial fills, tick marks, or gaps register as wrong.

**Step 3 — Scan:**
Slide the OMR sheet through the scanner box at a steady moderate speed under an external light source.

**Step 4 — Read the result:**
The 7-segment display shows total correct answers (0–8). The R/W LED gives per-question feedback. Reset the counter before scanning the next sheet.

---

## ⚡ Evaluation Logic

The system uses **strict 4-bit comparison** — a question is correct only when all four option bits match exactly:

| A | B | C | D | Result |
|---|---|---|---|---|
| 0 | 1 | 1 | 1 | ✅ Correct (A marked, A is correct) |
| 1 | 0 | 1 | 1 | ❌ Wrong (B marked, A is correct) |
| 0 | 0 | 1 | 1 | ❌ Wrong (A+B both marked) |
| 1 | 1 | 1 | 1 | ❌ Wrong (nothing marked) |

This mirrors real-world OMR: only a single, fully-filled correct bubble counts. Multiple marks, partial marks, and blank responses all count as wrong.

---

## ⚠️ Known Limitations

- **LDR sensitivity** varies with ambient light — a consistent, controlled external light source is required for reliable readings.
- **Manual scanning speed** must be steady; too fast causes missed reads, too slow can cause double counts.
- **Partial fills** (incomplete circles, tick marks, gaps in the mark) register as wrong answers.
- **7-segment display** occasionally misses a count pulse in hardware due to timing; a manual reset fixes this.
- **Scale:** Currently supports 8 questions × 4 options. Expanding requires additional flip-flop and MUX ICs.
- **No auto-reset:** The counter must be manually reset between each sheet scan.

---

## 👥 Team

| Name | Contribution |
|---|---|---|
| **Joyonta Debnath** | Hardware implementation, testing & debugging |
| **Md. Maksudur Rahman Turzo** | Theory, Proteus simulation, hardware, testing |
| **Ahamad Abtahi** | Hardware, components, testing, presentation |
| **Anas Rohan** | Simulation, hardware, theory, component sourcing |

---

## 🎓 Academic Context

| Field | Detail |
|---|---|
| Course | EEE 304 — Digital Electronics Laboratory |
| Department | Electrical & Electronic Engineering |
| University | Bangladesh University of Engineering and Technology (BUET) |
| Section / Group | A2 / Group 07 |

---

## 📄 License

This project was submitted as an academic assignment at BUET. Free to use for educational purposes with attribution.

---

<div align="center">

Built with solder, breadboards, and digital logic at BUET 🎓

</div>
