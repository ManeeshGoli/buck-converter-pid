# Schematic

PSpice/OrCAD Capture schematic for the closed-loop buck converter power stage, including the P-channel MOSFET high-side switch, NPN gate driver, LC output filter, and feedback divider.

## Files

- `buck_converter_schematic_final.png` — final verified schematic (correct D3 diode orientation, R2 = 1kΩ gate pull-up, C2 decoupling capacitor properly grounded)
- `netlist.txt` — corresponding PSpice netlist, confirmed clean and verified

## Key Nodes

| Net | Description |
|---|---|
| VCC | 12V input rail — V1 positive, R2 pull-up, M1 source, C2 |
| GND | Common ground reference |
| PWM_OUT | Arduino PWM signal into gate driver base |
| GATE_NODE | NPN collector / MOSFET gate, R2 pull-up junction |
| SWITCH_NODE | MOSFET drain / inductor input / diode cathode |
| VOUT | Inductor output / capacitor / load / feedback divider |

See root `README.md` for full design rationale and component values.
