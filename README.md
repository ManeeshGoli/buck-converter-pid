# Closed-Loop Digitally-Controlled Buck Converter

A 12V-to-5V buck converter with Arduino-based PID closed-loop voltage regulation, direct Timer1 register-level PWM generation, and PSpice simulation validation prior to hardware implementation.

## Overview

Switching DC-DC step-down converter using a P-channel MOSFET high-side switch, NPN gate driver, and closed-loop PID control on an Arduino Uno. Switching frequency (31.25 kHz) is generated via direct Timer1 register configuration rather than `analogWrite()`, whose default PWM frequency (~490–980 Hz) is too slow for practical power conversion.

## Key Specifications

| Parameter | Value |
|---|---|
| Input voltage | 12V DC |
| Target output voltage | 5V DC |
| Switching frequency | 31.25 kHz |
| Steady-state duty cycle (ideal) | ~41.7% |
| Control method | Digital PID, ATmega328P |

## Circuit Design

A P-channel high-side MOSFET was chosen over N-channel, since an N-channel switch in that position can't be fully driven by a 5V microcontroller pin once its source rises toward the input rail. Driven via an NPN low-side transistor, standard for microcontroller-level control.

**Signal path:** Arduino PWM → base resistor → NPN driver → gate pull-up node → P-MOSFET gate → switch node → LC filter → output.

**Key design decisions:**
- Inductor (330µH) and output capacitor (100µF) sized via standard ripple-current/ripple-voltage formulas
- Gate pull-up resistor reduced from 10kΩ to 1kΩ after diagnosing an RC timing issue preventing full MOSFET turn-off (see Debugging)
- Decoupling capacitor (C2, 0.1µF) included for real-hardware switching noise suppression — shown to have no effect in idealized simulation, since ideal SPICE sources lack parasitic wiring inductance
- 1N5819 Schottky diode specified for hardware; D1N4148 substituted in simulation due to library availability, expected to make simulated efficiency conservative vs. real hardware

## Bill of Materials

| Ref | Component | Value | Notes |
|---|---|---|---|
| V1 | DC supply | 12V, 2A | Wall adapter |
| M1 | P-MOSFET | IRF9540N | High-side switch |
| Q1 | NPN transistor | 2N2222A | Gate driver |
| D3 | Schottky diode | 1N5819 | Flyback diode |
| L1 | Inductor | 330µH, 5.2A | Toroidal |
| C1 | Output cap | 100µF, 25V | Electrolytic |
| C2 | Decoupling cap | 0.1µF | Ceramic |
| R1 | Load | 10Ω | Test load |
| R2 | Gate pull-up | 1kΩ | — |
| R3 | Base resistor | 1kΩ | — |
| — | Feedback divider | 10kΩ + 5.1kΩ | — |
| — | MCU | Arduino Uno R3 | — |

## Simulation Results

Open-loop, fixed 41.7% duty cycle:

| Metric | Result |
|---|---|
| Output voltage | ~6.19V (vs. 5V target) |
| Efficiency | ~82.5% (Pout 3.838W / Pin ~4.65W) |

The steady-state voltage error at fixed duty cycle is the direct motivation for closed-loop PID control. Full efficiency calculation methodology, including a measurement pitfall found and corrected during analysis, documented in `simulation/efficiency_analysis.md`.

## Debugging Journey

- **Weak gate drive:** 10kΩ pull-up couldn't fully charge MOSFET gate capacitance within the switching OFF window — diagnosed via waveform analysis, fixed by reducing to 1kΩ
- **Diode orientation:** initial flyback diode placement was reversed — caught via schematic inspection, corrected
- **C2 A/B test:** confirmed experimentally (not assumed) that C2 has no measurable simulated effect, consistent with idealized SPICE physics

## Project Status

- [x] Design, simulation, and efficiency analysis complete
- [x] Arduino Timer1 PWM + PID implementation
- [ ] Physical hardware build
- [ ] Real-hardware measurement and demo

## Author

Maneesh Reddy Goli — Electrical Engineering, University of South Florida
