# Time Modulation and Binary Switching Dynamics

The user was confused about how the ECU physically "changes" the duty cycle. They needed to understand that a solid-state switch is binary, not analog or resistive.

## Evidence
- The user stated: "I still don't understand how ecu changes duty".
- This represents a common conceptual hurdle: assuming switches behave like variable flow valves (dimmers) rather than instantaneous on/off toggles.

## Learning Captured
- **Binary MOSFET Switching**: MOSFETs cannot open "halfway." They are either fully closed (low resistance, current flowing) or fully open (infinite resistance, no current flowing).
- **Duty Cycle as Time Division**: To deliver varying levels of power, the ECU modulates the ratio of **time** the switch is ON versus OFF during each cycle.
- **Physical Averaging**: Because high-power motors and fans have mechanical inertia and electrical inductance, they coast during the brief OFF cycles, blending the pulses into a smooth, intermediate speed.

## Implications for Zone of Proximal Development
- The user has now completely bridged the concept of how time division creates variable power.
- They are ready to move on to diagnosing actual signals on a vehicle (Lesson 3) or writing firmware code (Arduino) to toggle a pin to see how this time division works in real code.
