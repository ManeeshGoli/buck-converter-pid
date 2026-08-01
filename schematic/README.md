# Schematic

This is the actual circuit — captured in both PSpice (for simulation) and KiCad (for the PCB I'm working on now). Same design, two tools, because I wanted to validate everything before committing to a board layout.

## What's here

- `netlist.txt` — the verified netlist, confirming every net is wired correctly (no shorts, no floating nodes). I went through a few rounds of catching wiring bugs this way before trusting the design.

## Key nets, if you're trying to follow the wiring

| Net | What it is |
|---|---|
| VCC | 12V rail — adapter, MOSFET source, pull-up resistor |
| GND | Common ground |
| PWM_OUT | Arduino's switching signal into the driver |
| GATE_NODE | Between the NPN driver and the MOSFET gate |
| SWITCH_NODE | MOSFET drain / inductor input / diode |
| VOUT | The actual regulated output |

Full design reasoning is in the main README.
