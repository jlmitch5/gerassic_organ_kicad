# Gerassic Organ (KiCad Remake)

A modern, documented, KiCad-based recreation and extension of the **Gerassic Organ** CMOS synth voice.  
This repository contains the full schematic and PCB files, along with corrections, improvements, and an optional 2164-based pitch control input.

---

## Table of Contents
1. [Overview](#overview)  
2. [Block Diagram](#block-diagram)  
3. [Power & Biasing](#power--biasing)  
4. [Integrator Core](#integrator-core)  
5. [Schmitt Trigger & Reset Path](#schmitt-trigger--reset-path)  
6. [4067 Selector](#4067-selector)  
7. [2164-Based Pitch CV Input](#2164-based-pitch-cv-input)  
8. [Bias Buffering](#bias-buffering)  
9. [Known Behaviors & Tuning Considerations](#known-behaviors--tuning-considerations)  
10. [Parts Notes / SMD Equivalents](#parts-notes--smd-equivalents)  
11. [Schematic](#schematic)

---

## Overview

This project recreates the **Gerassic Organ** with the following goals:

- Preserve the original sonic character  
- Correct wiring inconsistencies from earlier versions  
- Add optional **V/Oct-style pitch control** using a **SSM2164/V2164**  
- Allow banana or 3.5 mm jacks  
- Provide modern KiCad files for community development

The README focuses only on the **final, corrected engineering implementation**.

---

## Block Diagram

```

Keyboard → 4067 Selector → Integrator → Schmitt Trigger → Transistor Reset → Integrator
↑
│ (optional)
2164 Pitch CV

````

---

## Power & Biasing

The system runs on a **single +9V supply**.

### Bias Voltages
- **VBias (~4.5V)** is used extensively throughout the circuit.  
- The original design relied on VBias sagging and interacting across subcircuits.  
- This remake **buffers VBias** to stabilize operation, particularly with the 2164.

### 2164 Supply (Single-Supply Mode)

Per Pichenettes’ notes:

- 2164 **Vcc = +9V**
- 2164 **Vee = GND**
- 2164 **GND pin receives VBias** (used as the internal reference)

This allows the 2164 to operate correctly from a single supply.

---

## Integrator Core

The integrator is the heart of the Gerassic Organ tone generator.

### ✔ Corrected Pin Connections (Important)

- **C5/C6/C7** connect **between pin 6 and pin 7** of the op-amp.  
  *These are part of the feedback loop, not tied to ground or Z.*

- **Pin 5** connects **only to VBias**.

- **R25** connects to **pin 7**.

- The 2164-modified version injects current from the 2164 output to **pin 6**.

---

## Schmitt Trigger & Reset Path

The Schmitt trigger and transistor form the reset mechanism for the oscillator.

### ✔ Corrected Wiring

- **U3A pin 2** connects **only to pin 7** of the integrator op-amp.  
- **R26/R32** form a divider from **U3B output** to the **transistor base**.  
  - The divider must reference the **integrator output**, *not a current-carrying node*.
- This ensures stable switching thresholds.

---

## 4067 Selector

The CD4067 selects which key (or CV level) feeds the integrator.

### ✔ Correction Applied

- Connection to **A3** was missing in earlier drafts — now restored.

---

## 2164-Based Pitch CV Input

The project adds an optional **pitch CV input**, following the principles used in Mutable Instruments’ *Ripples* and *Anushri*.

### How it works:

- **2164 input (Z)** receives a current proportional to the desired pitch.  
- **2164 output** goes directly to **pin 6 of the integrator**.  
  - This effectively creates a **variable current source** controlling frequency.

### ✔ Interpreting “Pin 6 connected to Z”
Correct interpretation:

- The 2164 allows a **variable amount of current** between Z and pin 6.  
- It does **not** connect pin 6 *directly* to Z electrically.  
- You are modulating the integrator’s charging current.

### CV Range

- Target: **0V → +9V control** works fine for banana-style workflows.  
- Eurorack ±5 V can be adapted using a simple offset circuit.

### 2164 Control Voltage Requirements

- VC must vary from **~4.5V (max current)** to **~6.0V (−9 octaves)**.  
- This is why VBias must be stable (buffered).

---

## Bias Buffering

VBias feeds:

- 2164 reference pins  
- Integrator reference  
- Several comparator thresholds

### ✔ Final Recommendation

- Use a **rail-to-rail op-amp** when buffering VBias or CV inputs.
- LM358 is fine for most of the circuit, but note:
  - Its output **cannot reach the positive rail**.
  - So buffers may top out ~1V below +9V.

---

## Known Behaviors & Tuning Considerations

Several behaviors are inherent to the integrator + Schmitt structure:

### 1. Switching-Time Constant Term
At higher frequencies, the Schmitt trigger introduces a “constant time” that affects pitch.  
A small offset current into the integrator can compensate for this.

### 2. 2164 Non-Idealities
- Slight temperature dependence  
- Slight deviation from perfect exponential behavior  

Pichenettes references the **Anushri page 3** technique of scaling the CV by a temperature-dependent factor for improved stability.  
This design keeps things simpler and omits this compensation.

### 3. Buffered VBias Effect on Tone  
Buffering VBias removes some of the original circuit’s “wobbly coupling,” slightly changing the tone — but keeping the 2164 stable.

### 4. Decay/X-Resistor Experiments
- LM358 limit near +9V explains loss of dynamic range in certain configurations.
- Simple non-inverting buffers work best.
- Variable resistance + variable bias creates many interesting behaviors (infinite drones, phantom notes, etc).

---

## Parts Notes / SMD Equivalents

### Op-amps
- **LM358** (single supply, low-end, non-rail-to-rail output)
- **TL072 / TL07x** (dual supply preferred; watch for input swing limitations)

For SMD:
- LM358 → **LM358DR**, **LM358DT**  
- TL072 → **TL072CDR**, etc.

For full rail-to-rail behavior (recommended for CV inputs & buffers):
- **OPA1678**
- **OPA2134**
- **MCP6002** (budget)

### Transistors
- 2N547/2N557 → SOT-23 equivalents: **MMBT3904 / MMBT3906**

### Diodes
- 1N4148 → **1N4148W (SMD)**  
- Power: 1N4007 → **1N4007FL**

Resistors and capacitors translate directly to 0603/0805 packages.

---

## Schematic

Images omitted for now, but the README includes the section:

```markdown
<!--
### Schematic Overview
![Main Schematic](docs/img/schematic.png)
-->
````

You can uncomment once you add exported schematic images.

---

## License

Open-source hardware; license included in repo.

---

## Contributions

PRs welcome for layout improvements, alternative 2164 CV converters, documentation, and demonstrations.
