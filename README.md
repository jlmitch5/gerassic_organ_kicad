# Gerassic Organ – Technical Documentation
**Version:** Based on commit `d98319451b52165d5c0c8d84ce281333d11cbb31`  
**Author:** jlmitch5  

The **Gerassic Organ** is a CMOS-based synthesizer voice inspired by Peter Blasser’s instruments, extended with:

- Voltage-controlled pitch using a **2164 VCA as a voltage-controlled resistor (VCR)**  
- Bias-controlled decay, timbre, and “growl” behavior  
- Buffered bias supply for improved tuning stability  
- Compatibility with banana or 3.5 mm jack CV workflows  
- Preservation of original idiosyncratic behavior and tone  

This document explains the function of the circuit, design corrections, tuning notes, and how to interpret the KiCad project.

---

## 1. Repository Overview

This repository contains:

- **KiCad project files**  
- **Schematic (`*.kicad_sch`) and PCB (`*.kicad_pcb`)**  
- **Supporting library files**  
- This README.md as the **primary design documentation**

---

## 2. Schematic Overview

> **Note:** Export your schematics as PNG/SVG and replace the URLs below with the correct GitHub links.

### 2.1 Main Schematic
```

![Main Schematic](https://github.com/jlmitch5/gerassic_organ_kicad/blob/d98319451b52165d5c0c8d84ce281333d11cbb31/<ADD_SCHEMATIC_FILE>)

```

### 2.2 Oscillator (Integrator + Schmitt Trigger)
```

![Oscillator](https://github.com/jlmitch5/gerassic_organ_kicad/blob/d98319451b52165d5c0c8d84ce281333d11cbb31/<ADD_SCHEMATIC_FILE>)

```

### 2.3 2164 Voltage-Controlled Resistor (Pitch Control)
```

![2164 VCR](https://github.com/jlmitch5/gerassic_organ_kicad/blob/d98319451b52165d5c0c8d84ce281333d11cbb31/<ADD_SCHEMATIC_FILE>)

```

### 2.4 Bias Supply + Buffer
```

![Bias Buffer](https://github.com/jlmitch5/gerassic_organ_kicad/blob/d98319451b52165d5c0c8d84ce281333d11cbb31/<ADD_SCHEMATIC_FILE>)

```

### 2.5 Pitch CV and Decay CV Inputs
```

![CV Inputs](https://github.com/jlmitch5/gerassic_organ_kicad/blob/d98319451b52165d5c0c8d84ce281333d11cbb31/<ADD_SCHEMATIC_FILE>)

```

---

## 3. Functional Description

### 3.1 CMOS Relaxation Oscillator
The core of the instrument is a CMOS-based relaxation oscillator:

- An **integrator** ramps voltage up  
- A **comparator/Schmitt trigger** flips state at thresholds  
- A **transistor** discharges the timing capacitor  
- Oscillation frequency is set by the **current into the timing capacitor**

The innovation here is replacing fixed resistors with a **2164 VCA used as a voltage-controlled resistor**.

---

### 3.2 2164 as a Voltage-Controlled Resistor (VCR)

The 2164 sinks current exponentially with respect to its control voltage:

```

I_out = I_in * 2^(-(V_control - 0V) / 33mV)

```

In this circuit:

- 2164 **GND pin sits at Vbias (~4.1–4.5 V)**  
- 2164 **Vee = GND**, **Vcc = +9 V**  
- Pitch CV is scaled into the 2164’s required control range: **4.5–6 V**  

This gives:

- Smooth exponential pitch response  
- Limited V/oct range (sufficient for musical play)  
- Characterful nonlinearities at extremes

---

### 3.3 Bias Supply and Buffering

A bias voltage (~4.1 V) forms the reference point for:

- Oscillator comparators  
- CV scaling  
- 2164 ground reference  

Originally this bias drooped during button presses, causing pitch instability.  
A dedicated **op-amp buffer** now stabilizes bias while retaining the original tone.

---

### 3.4 Pitch CV Input

Pitch CV enters through an inverting op-amp stage scaled to the 2164’s 4.5–6 V control range.

Design notes:

- Rail-to-rail op-amps are recommended for accurate low-voltage operation  
- 100k input impedance keeps banana-jack behavior stable  
- Can be adapted to Eurorack (±5 V) with minor scaling changes

---

### 3.5 Decay / X-Resistor Control

The “X-resistor” node affects:

- Decay time  
- Droning behavior  
- Harmonic content

Because the decay control operates by pushing current into a floating node, pots **interact** in nonlinear and musically expressive ways.

Improvement:

- Replace the LM358 with a **rail-to-rail op-amp** for full 9 V swing  
- Remove unnecessary inversion stages and stray trimmers  
- Follow the 2164-style current-source topology for clean CV control

---

## 4. Full List of Design Corrections

### Oscillator Section
- Feedback capacitor must be wired **strictly across op-amp pins 6 and 7**  
- Pin 5 connects **only to Vbias**  
- Comparator input must not be tied to other nodes  
- R25 terminates at the correct op-amp pin after correction

### R26 / R32 Divider
Correct divider:

```

U3B output → R26 → node → R32 → GND

```

The node drives the base of the discharge transistor.

### 2164 Wiring
- **GND pin → Vbias** (virtual ground)  
- **Vee → GND**  
- **Vcc → +9 V**  
- Unused inputs/outputs grounded per datasheet  
- Control voltage must remain **4.5–6.0 V**

### Bias Supply
- Bias must be **buffered**  
- Prevents pitch shifts from transient current draw

### CV Inputs
- Prefer a single inverting gain stage  
- Rail-to-rail op-amps recommended  
- No unnecessary buffering

### X-Resistor / Decay CV
- LM358 replaced with rail-to-rail part  
- Remove redundant inversion stage  
- Remove unused trimmer (RV6)  
- Interactive pot behavior is expected

### V/Oct Considerations
- Comparator introduces a constant timing error  
- 2164 is temperature-sensitive  
- Expect limited but usable V/oct range

---

## 5. Bill of Materials (Simplified)

| Component | Description | Notes |
|----------|-------------|-------|
| 2164 VCA | Exponential VCR | Core of pitch control |
| LM358 (replace) | Dual op-amp | Use rail-to-rail for CV sections |
| TL07x | Audio op-amps | Integrator / comparator |
| CD4067 | Analog mux | Check A0–A3 ordering |
| BC547/BC557 | Transistors | SMD: BC846/BC856 equivalents |
| Diodes | 1N4148 / 1N400x | Signal + power |
| Resistors | 1% metal film | Standard values |
| Capacitors | Electrolytic + film | Timing cap C3 may be switchable |

Export a full BOM via KiCad as needed.

---

## 6. Usage Notes

### Power Requirements
- +9 V supply  
- GND  
- Vbias internally generated and buffered

### Calibration
- Tune V/oct trimmer  
- Select suitable timing capacitor values  
- Expect temperature influence on tuning (due to 2164 exponential cell)

### Patch Notes
- Works with banana or 3.5 mm jacks  
- Inputs are high-impedance  
- Bias buffering prevents cross-interference between controls

---

## 7. Files

KiCad project and schematics:  
https://github.com/jlmitch5/gerassic_organ_kicad/tree/d98319451b52165d5c0c8d84ce281333d11cbb31

---

## 8. Revision History

| Version | Changes |
|---------|---------|
| v1.0 | Initial full documentation in README.md |
