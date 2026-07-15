# Efficiency Analysis

## Summary

| Metric | Value |
|---|---|
| Output power (Pout) | 3.838 W |
| Input power (Pin) | 4.652 W |
| **Efficiency** | **82.5%** |

Measured from open-loop PSpice transient simulation, fixed duty cycle (~41.7%), steady-state window (9.5–9.82ms).

## Methodology

Output power (`W(R1)`) was stable and consistent from the first measurement, cross-validated via V×I.

Input power required extra care: PSpice Probe's cursor-table "Avg Y" is simply `(Max+Min)/2`, which is accurate for flat signals but badly wrong for the sharply pulsed source current in a switching converter (current only flows during the MOSFET's ~13.3µs ON interval). This initially produced physically impossible results (Pin < Pout).

**Fix:** used PSpice's cumulative `AVG()` function with a weighted-subtraction formula to extract a true time-averaged value over the settled window:
Pin = (Y2×t2 − Y1×t1) / (t2 − t1) = 4.652 W

Reproducible across multiple runs (4.65W, 4.638W, 4.652W).

## C2 Decoupling Capacitor — A/B Test

| Config | Pin | Efficiency |
|---|---|---|
| Without C2 | 4.647 W | 82.6% |
| With C2 | 4.652 W | 82.5% |

No measurable difference — expected, since C2 sits across an ideal source with no parasitic wiring inductance modeled. C2's real benefit (suppressing switching noise from real wiring inductance) only applies to physical hardware, so it's retained in the build despite the null simulation result.

## Known Limitation

Simulation uses a D1N4148 diode (library substitute) instead of the real 1N5819 Schottky specified for hardware, due to higher series resistance/forward drop at these current levels. **Real hardware efficiency is expected to exceed this 82.5% simulated figure**, closer to the typical 85–95% range for Schottky-based buck converters.
