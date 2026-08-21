# ECU Control Loops and PWM Signal Extremes

The user successfully recognized that a flat Direct Current (DC) voltage signal represents the extremes of Pulse Width Modulation (0% and 100% duty cycle) and requested clarification on the inputs and logic the ECU uses to adjust these parameters.

## Evidence
- The user stated: "so DC usually has 0hz and duty 100%... and ECU changes the duty and frecuency based on soem lksdjfljdsfthat".
- This demonstrated an intuitive breakthrough regarding the physical shape of a continuous DC signal and a desire to connect signal parameters to engine bay sensors and control software.

## Learning Captured
- **DC as a PWM Limit**: 0% duty cycle (flat 0V) and 100% duty cycle (flat 12V) both have a frequency of 0 Hz because no switching transitions occur.
- **Closed-Loop Control**: The ECU determines the dynamic **Duty Cycle** output by checking sensor feedback (like engine coolant temperature), comparing it to a target value (setpoint), calculating the error, and using a lookup map or PID loop to adjust the output.
- **Fixed Frequency**: In vehicle design, the **Frequency** of switching is kept constant to match the electrical and mechanical limits of the actuator (to prevent audible humming or excessive transistor heat losses).

## Implications for Zone of Proximal Development
- Having understood the theoretical signal behaviors, physical circuit diagnostics, and ECU control loops, the user is now ready to explore how this logic is implemented programmatically.
- The next development block can focus on using microcontrollers (e.g., Arduino/ESP32) to generate PWM signals and read analog sensor inputs to build a custom fan speed controller.
