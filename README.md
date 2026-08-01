# Buck Converter with Arduino PID Control

I built a 12V-to-5V switching power converter from scratch — designed it, simulated it in PSpice, broke it a few times on a breadboard, fixed it, and got it running with real closed-loop feedback control. This README walks through the design, the math behind it, and honestly, a lot of the mistakes I made along the way, because I think those are just as worth documenting as the parts that worked.

## Why I Built This

I'm an EE sophomore at USF, and most of my prior work was simulation-only — I'd never actually built a power circuit with my own hands, dealt with a component overheating, or debugged something at 1am wondering why a capacitor just popped. I wanted a project that forced me to bridge circuit design, embedded programming, and control theory into something that actually exists as a physical object, not just a waveform on a screen.

A buck converter felt like the right scope: small enough to finish, but genuinely representative of what shows up in almost every piece of electronics you own — phone chargers, EV low-voltage systems, drone power distribution. And adding PID control on top of it meant I wasn't just building "a power supply," I was learning the same closed-loop feedback concept that shows up in robotics, motor control, and basically every mechatronics application.

## What It Does

Takes a 12V DC input and regulates it down to a steady 5V output, using a P-channel MOSFET as the switch, an Arduino Uno generating the switching signal, and a PID control loop reading the output and correcting it in real time.

## Specs

| Parameter | Value |
|---|---|
| Input voltage | 12V DC |
| Target output voltage | 5V DC |
| Switching frequency | 31.25 kHz |
| Load | 10Ω resistor (stand-in for a real device) |
| Control | Digital PID running on ATmega328P |

## The Design

### Why a P-channel MOSFET, not the more common N-channel

This tripped me up early on. A high-side switch (which is what you need here) is easy to drive with an N-channel MOSFET *if* you have a gate driver IC or bootstrap circuit — but I wanted to drive it directly from the Arduino. The problem: once an N-channel MOSFET's source rises up near the 12V rail, a 5V gate signal isn't enough to keep it fully on. A P-channel MOSFET sidesteps this — I drive it through a small NPN transistor acting as a low-side switch, which flips the P-MOSFET's gate voltage relative to its source in the way that actually turns it on.

**Signal path:** Arduino PWM pin → 1kΩ resistor → NPN transistor → pull-up resistor / P-MOSFET gate → switch node → inductor/capacitor

## Debugging Journey

- **Weak gate drive:** 10kΩ pull-up couldn't fully charge MOSFET gate capacitance within the switching OFF window — diagnosed via waveform analysis, fixed by reducing to 1kΩ
- **Diode orientation:** initial flyback diode placement was reversed — caught via schematic inspection, corrected
- **C2 A/B test:** confirmed experimentally (not assumed) that C2 has no measurable simulated effect, consistent with idealized SPICE physics

## PCB Design

Once the breadboard version was working, I moved the whole design into KiCad and laid out a proper 2-layer PCB — same schematic, same component values, but designed to actually be fabricated rather than wired by hand.

A few things I specifically carried over from what I learned on the breadboard build:
- **R1's footprint is sized for the real 5W wirewound resistor**, not the small 0.25W footprint I'd have used before learning that lesson the hard way (see "Things I Broke" below)
- **C2 sits as close as physically possible to M1's source pin** on the board, since that proximity is what actually matters for its decoupling job
- **Power path traces (VCC, switch node, VOUT) are routed wide** — sized using KiCad's trace width calculator for the real current levels, not left at default thickness
- Caught a real wiring bug during this process too — an early routing pass accidentally shorted my VOUT node to GND, which I only found by generating and reading the actual KiCad netlist rather than trusting the visual layout. Worth doing that check on any board before sending it off, honestly.

Ready for fabrication next — planning to order through JLCPCB or a local Indian PCB service and hand-assemble it once it arrives.

## Project Status

- [x] Design, simulation, and efficiency analysis complete
- [x] Arduino Timer1 PWM + PID implementation
- [x] Physical hardware build
- [x] Real-hardware measurement
- [x] PCB Design 

## Author

Maneesh Reddy Goli — Electrical Engineering, University of South Florida
