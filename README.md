# Buck Converter with Arduino PID Control

This is a 12V-to-5V power converter I designed, simulated, broke a couple times, and eventually got working with real closed-loop control. I'm writing this readme the way I'd actually explain the project to someone, mistakes included, because honestly the mistakes taught me more than the parts that worked on the first try.

## Why

I'm a sophomore studying EE at USF. Before this, basically everything I'd built existed only in simulation — I'd never had a component actually overheat on me, never had to figure out why a capacitor just popped. I wanted something that would force me out of that, and a buck converter seemed like a good pick: it's the kind of circuit hiding inside almost every charger and power adapter you own, so it felt like something worth actually understanding rather than just simulating. Adding PID control on top gave me an excuse to learn closed-loop feedback too, which shows up everywhere once you start looking — robotics, motor control, basically anything mechatronic.

## What it actually does

Takes 12V in, puts out a regulated 5V, using a P-channel MOSFET as the switch and an Arduino running both the switching signal and a PID loop that watches the output and corrects it on the fly.

## Specs

| Parameter | Value |
|---|---|
| Input voltage | 12V DC |
| Target output | 5V DC |
| Switching frequency | 31.25 kHz |
| Load | 10Ω resistor |
| Control | Digital PID on an ATmega328P |

## Design notes

### The MOSFET choice

I went with a P-channel MOSFET instead of the more typical N-channel, and this actually came from a mistake I almost made. High-side switching with an N-channel MOSFET works fine if you've got a gate driver IC, but I wanted to drive it straight off the Arduino, and once an N-channel MOSFET's source climbs up near 12V, a 5V gate signal just isn't enough anymore to hold it on. P-channel sidesteps the whole problem — I drive it through a small NPN transistor that flips the gate voltage the right way relative to the source.

Signal flow: Arduino PWM pin → 1kΩ resistor → NPN → pull-up/gate node → P-MOSFET → switch node → LC filter → output.

### Sizing everything

Inductor's 330µH, sized for 30% ripple current at 31.25kHz. Output cap is 100µF, sized for 1% ripple with some extra margin built in for how bad the ESR is on cheap electrolytics. The flyback diode has to be Schottky — a regular diode's forward drop and switching speed just aren't good enough at this frequency.

### Timer1

`analogWrite()` tops out around 490-980Hz depending on the pin, which is nowhere near fast enough. I ended up configuring Timer1's registers by hand — Fast PWM, no prescaler, TOP set to 511 — to actually hit 31.25kHz. This was probably the single most "sit down and actually read the datasheet" moment of the whole build.

## Parts list

| Ref | Part | Notes |
|---|---|---|
| V1 | 12V/2A adapter | Power in |
| M1 | IRF9540N | The switch |
| Q1 | 2N2222A | Drives the MOSFET gate |
| D3 | 1N5819 | Flyback diode |
| L1 | 330µH, 5.2A | |
| C1 | 100µF, 25V | Output filter |
| C2 | 0.1µF | Decoupling, right by M1 |
| R1 | 10Ω, 5W | The load — had to upgrade this one, more below |
| R2, R3 | 1kΩ each | Pull-up, base resistor |
| R4, R5 | 10kΩ, 5.1kΩ | Feedback divider |

## Simulation (PSpice)

Ran this open-loop first, fixed duty cycle around 41.7%. Output settled near 6.2V, which is higher than the 5V target but that's expected — real components eat into the ideal Vout = D×Vin relationship. Calculated efficiency came out to 80%.

That efficiency number gave me way more trouble than it should have. First attempt gave a result that was physically impossible — input power came out lower than output power, which obviously can't happen. Turned out PSpice's quick "average" cursor readout is just (Max+Min)/2, which works fine for smooth signals but falls apart for something as spiky as a switching converter's input current. Took a while to figure that out. Whole story's in `simulation/efficiency_analysis.md`, and I think it's honestly more interesting than the final number itself.

## Real hardware numbers

Got it running on breadboard with PID actually closing the loop:

| Metric | Value |
|---|---|
| Output voltage | 4.92V |
| Output current | 0.437A |
| Output power | ~1.91W |
| Input current | 0.2A |
| Input power | 2.4W |
| Efficiency | ~79.6% |

Lower than the simulated 82.5%, which tracks — breadboard connections aren't free, and my MOSFET's legs were too thick for the breadboard so I had to extend them with jumper wires, which definitely doesn't help.



## PCB

Once the breadboard version worked, I moved the design into KiCad and laid out an actual 2-layer board. Same circuit, same values, just meant to be fabricated instead of hand-wired.

A couple things I made sure to carry over from what the breadboard build taught me: R1's footprint is sized for a real 5W wirewound resistor this time, not the tiny 0.25W one I originally (wrongly) used. C2 sits as close as I could physically get it to M1's source pin on the layout. Power traces are routed wide, sized using KiCad's trace calculator instead of just leaving them at default.

Also caught a real bug doing this — an early routing pass had accidentally shorted VOUT straight to ground. Only found it because I pulled the actual netlist and read through it instead of trusting what the layout looked like visually. Good reminder that "looks right" and "is right" aren't the same thing.

Next step is actually getting it fabricated — probably JLCPCB or a local service here in India — and hand-soldering it once it shows up.

## Things I broke along the way

Figured this deserved its own section, because pretending none of this happened would be leaving out most of what I actually learned.

**Burned a resistor the first time I powered it on.** Got the resistance right (10Ω) but never checked the power rating — it needed to dissipate about 3.6W and I had it wired with a standard 0.25W part from an assorted kit. Obvious in hindsight. Always check wattage, not just ohms, for anything in the main current path.

**Popped two capacitors**, back to back. Root cause was a reversed transistor — I'd swapped Emitter and Collector on the NPN, so it never fully saturated, which left the MOSFET in some half-on state that spiked voltage past what the caps could take.

**Gate drive was too weak at first.** Had a 10kΩ pull-up that couldn't charge the MOSFET's gate fast enough in the time available between switching cycles, so the gate never fully swung and the MOSFET basically stayed half-on the whole time. Dropped it to 1kΩ and that fixed it.

**PID oscillation, which is what actually caused the burning smell.** My first tuning attempt had way too much gain — duty cycle was slamming from 0 to max every single cycle. Fixed by cutting Kp by 10x and adding a hard slew-rate limit on how much the duty cycle's allowed to change per loop, regardless of what the PID math says to do.

## Where it's at

- [x] Design and sizing calculations
- [x] PSpice schematic + simulation
- [x] Efficiency analysis
- [x] Timer1 PWM + PID code
- [x] Working breadboard build
- [x] Real hardware measurements
- [x] KiCad PCB design

## Author

Maneesh Reddy Goli — EE, University of South Florida

*Used AI for updating the documentation.
