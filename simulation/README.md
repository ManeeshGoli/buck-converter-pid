# Simulation

PSpice validation of the power stage before I touched a breadboard. Given how many things went wrong once I did build it physically (see the main README's "Things I Broke" section), I'm genuinely glad I did this first — a couple of the mistakes I caught here would've been way more annoying to debug in copper or on a live circuit.

## What's here

- `efficiency_analysis.md` — the full efficiency calculation, including a genuinely embarrassing detour where I got a physically impossible result and had to figure out why (spoiler: PSpice's quick-look averaging tool doesn't work the way you'd assume for a switching waveform)

## tl;dr results

Fixed duty cycle, open-loop: output settled around 6.19V, calculated efficiency 82.5%. Full math and the debugging story in `efficiency_analysis.md`.
