# 6S LiPo to 5V Asynchronous Buck Converter

A discrete, analog-controlled asynchronous buck converter designed to step down a 6S LiPo battery pack (18.0V – 25.2V) to a regulated 5V output at 85% efficiency. Built from discrete components rather than an off-the-shelf regulator IC to demonstrate foundational power stage design, analog PWM generation, and closed-loop control stability.

---

## Key Specifications

| Parameter | Value | Details |
| :--- | :--- | :--- |
| **Input Voltage ($V_{in}$)** | 18.0V – 25.2V | Nominal 6S LiPo operating range (3.0V – 4.2V/cell) |
| **Output Voltage ($V_{out}$)** | 5.0V | Regulated output for microcontrollers and sensors |
| **Load Current ($I_{out}$)** | 100 mA – 500 mA | Designed for continuous conduction mode (CCM) across all loads |
| **Switching Frequency ($f_{sw}$)** | 100 kHz | Comparator inverting input sawtooth wave frequency |
| **Target Efficiency** | ~85% | Optimized power path and component selection |
| **Control Topology** | Voltage-Mode Control | Single-loop feedback with analog Type III compensation |

---

## Architectural & Design Choices

### 1. Discrete Control vs. Integrated Regulator IC
- Designed the power stage and PWM control circuitry from scratch using operational amplifiers, comparators, and standard discrete silicon.
- **Rationale:** Allows full visibility into gate drive dynamics, loop transfer functions, compensation tuning, and parasitic considerations that are typically abstracted inside integrated PMICs.

### 2. Voltage-Mode Control
- Utilizes a single voltage feedback loop comparing output error against a reference rather than a dual/nested current-mode loop.
- **Rationale:** Keeps the control architecture clean and deterministic for discrete analog implementation while providing robust regulation across steady-state steps.

### 3. Continuous Conduction Mode (CCM)
- Inductor sized ($470\,\mu\text{H}$) to prevent the inductor current from dropping to zero at the minimum rated output current ($100\text{ mA}$).
- **Rationale:** Prevents discontinuous conduction mode (DCM) dynamics, avoiding non-linear gain changes and output voltage peaking at light loads.

---

## Subsystem Details

### Power Stage
- **High-Side Switch:** N-channel MOSFET driven by an **IR2125** high-side gate driver with a dedicated bootstrap diode/capacitor circuit.
- **Freewheeling Diode:** **MBRS360** Schottky diode providing low forward voltage drop ($V_F$) and fast recovery during switch off-time.
- **Output Filter:** $470\,\mu\text{H}$ power inductor paired with low-ESR ceramic/electrolytic capacitance to suppress output voltage ripple.

### Analog PWM Generation
- **Sawtooth Oscillator:** An **LM334** constant-current source charges a timing capacitor into a **TLC555** timer to generate a linear 100 kHz ramp waveform.
- **PWM Modulation:** The ramp is compared against the compensator output using an **MCP6561** high-speed comparator to produce the pulse-width modulated gate drive signal.

### Loop Compensation (Type III Network)
- Built around an **MCP6021** rail-to-rail operational amplifier.
- Places two zeros and two poles to compensate for the second-order $LC$ resonance filter peaking ($f_o \approx 1.57\text{ kHz}$) and ESR zero.
- Maximizes phase margin under worst-case input voltage condition ($25.2\text{V}$) where loop gain peaks.

### Soft-Start & Inrush Protection
- Incorporates a $10\text{ ms}$ $RC$ soft-start circuit on the reference voltage line to clamp initial duty cycle expansion.
- Limits inrush current to $\le 10\text{ mA}$ and eliminates turn-on voltage overshoot.

---

## Layout & Simulation

- **Simulation:** Complete open-loop and closed-loop frequency response (Bode plots), soft-start verification, and transient load response modeled in **LTspice**.
- **PCB Design:** 4-layer board designed in **Altium Designer**:
  - Dedicated internal ground plane layer to shield noise and maintain low loop inductance.
  - Minimized loop area around the high $di/dt$ switching node (MOSFET Drain/Source and Schottky diode).
  - Isolated analog timing/feedback traces from the noisy switching node to prevent jitter in PWM ramp generation.
