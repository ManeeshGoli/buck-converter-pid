# Code

The actual Arduino sketch — Timer1 register configuration for real switching frequency PWM, plus the PID loop that reads the feedback voltage and adjusts duty cycle in real time.

## Status

Working. Tested on real hardware, closed-loop control confirmed converging (see main README for measured results).

## A few things worth knowing if you're reading this code

- **Why not `analogWrite()`:** it caps out around 490-980Hz by default, way too slow for a real power converter. This code configures Timer1's registers directly instead, to hit 31.25kHz.
- **The slew-rate limiter (`MAX_STEP`)** exists because my first tuning attempt was way too aggressive and the duty cycle was slamming between 0 and 511 every cycle — genuinely caused a burning smell at one point. This caps how much the duty cycle can change per loop, as a hard safety net independent of whatever the PID gains say to do.
- **The 8-sample ADC averaging** was added to reduce noise from reading a signal that's physically close to something switching at 31kHz — helped smooth things out and improved measured efficiency, though it didn't fully explain a small voltage offset noted in the main README.
