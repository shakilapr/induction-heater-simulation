# 1000W Induction Heater Simulation

This repository contains the Proteus simulation files for a 1000W induction heater. The project calculates the required power, coil inductance, and resonant capacitance based on a set of heating specifications.

It then explores two different MOSFET driver circuits to power the induction coil:
1.  **A microcontroller-based driver** using an Arduino UNO.
2.  **An analog driver** using a self-oscillating MOSFET-based astable multivibrator.

***

## Project Goal & Specifications

The primary goal is to design an induction heater capable of heating 1 kg of steel from 50°C to 1000°C in 10 minutes (600 seconds).

### Specifications
* **Workpiece:** 1 kg Steel
* **Specific Heat Capacity (c):** $\sim 500 J/kg^{\circ}C$
* **Initial Temperature:** $50^{\circ}C$
* **Final Temperature:** $1000^{\circ}C$
* **Heating Time (t):** 600s (10 minutes)
* **Desired Resonant Frequency (f):** 50 kHz

### Coil Parameters
* **Coil Diameter:** 40 mm (0.04 m)
* **Number of Turns (N):** 10
* **Wire Type:** Copper coil pipe
* **Wire Outer Diameter:** 1/4 inch (6.35 mm)
* **Wire Wall Thickness:** 1 mm
* **Gap Between Turns:** 3 mm

***

## 1. Design Calculations

### Power Requirements

1.  **Temperature Change ($\Delta T$):**
    $\Delta T = 	ext{Final Temperature} - 	ext{Initial Temperature}$
    $\Delta T = 1000^{\circ}C - 50^{\circ}C = 950^{\circ}C$

2.  **Heat Energy Required (Q):**
    $Q = m \cdot c \cdot \Delta T$
    $Q = 1~kg 	imes 500~J/kg^{\circ}C 	imes 950^{\circ}C = 475,000~J$

3.  **Ideal Power ($P_i$):**
    $P_i = Q / t$
    $P_i = 475,000~J / 600~s = 791.67~W$

4.  **Actual Power (P) (Assuming 80% efficiency $\eta$):**
    $P = P_i / \eta$
    $P = 791.67~W / 0.80 = 989.59~W pprox 1000~W$

5.  **Supply Selection:**
    To achieve $\sim 1000W$, a suitable power supply is required.
    * For 12V: $I = 989.59W / 12V = 82.47~A$
    * For 24V: $I = 989.59W / 24V = 41.23~A$
    * For 48V: $I = 989.59W / 48V = 20.62~A$

    **Conclusion:** A **48V, 25A power supply** is the best choice.

### Coil & Resonant Tank

1.  **Coil Inductance (L):**
    Based on the coil parameters and the formula $L = (\mu_0 	imes N^2 	imes A_{coil}) / l_{coil}$, the calculated inductance is:
    $L = 3.95 	imes 10^{-6} H$ or **$3.95~\mu H$**

2.  **Required Capacitance (C):**
    To achieve the resonant frequency (f) of 50 kHz with the $3.95~\mu H$ coil:
    $C = 1 / ((2 \pi f)^2 	imes L)$
    $C = 1 / ((2 \pi 	imes 50 	imes 10^3)^2 	imes 3.95 	imes 10^{-6})$
    $C = 2.57 	imes 10^{-6} F$ or **$2.57~\mu F$**

### Heat Sink Design

1.  **MOSFET Power Dissipation ($P_D$):**
    Using the SNW60N15 MOSFET ($R_{DS(on)} = 33~m\Omega$) and the calculated current ($I_D pprox 25~A$):
    $P_D = I_D^2 	imes R_{DS(on)}$
    $P_D = (25)^2 	imes 0.033 = 20.625~W$

2.  **Required Thermal Resistance ($R_{	heta SA}$):**
    Assuming $T_{Jmax} = 150^{\circ}C$ and $T_A = 25^{\circ}C$:
    $R_{	heta SA} = ((T_{Jmax} - T_A) / P_D) - R_{	heta JC} - R_{	heta CS}$
    $R_{	heta SA} = ((150 - 25) / 20.625) - 0.45 - 0.25$
    $R_{	heta SA} = 6.06 - 0.7 = 5.36^{\circ}C/W$

    **Conclusion:** The heat sink must have a thermal resistance **less than or equal to $5.36^{\circ}C/W$**.

***

## 2. Circuit Design 1: Arduino Driver

This design uses an Arduino UNO to generate alternating PWM signals to drive two MOSFETs in a half-bridge configuration.

### Components
* Arduino UNO
* MOSFET (60N15) $	imes$ 2
* Potentiometer (10K $\Omega$) $	imes$ 1
* 48V 25A Power Supply

### Control Logic & Code
The potentiometer on pin `A0` is used to control the period of the PWM signal. The code maps the analog value (0-1023) to a period from 17ms to 1000ms (a frequency range of $\sim 1Hz$ to $60Hz$). The two MOSFETs are driven in an alternating fashion.

```cpp
const int pwmPinA = 11;
const int pwmPinB = 10;
const int potPin = A0;

void setup() {
  pinMode(pwmPinA, OUTPUT);
  pinMode(pwmPinB, OUTPUT);
  pinMode(potPin, INPUT);
}

void loop() {
  int potValue = analogRead(potPin);  // to read the potentiometer
  // Map pot value (0-1023) to a period from 1000ms (1Hz) to 17ms (~60Hz)
  long period = map(potValue, 0, 1023, 1000, 17);
  long halfPeriod = period / 2;

  digitalWrite(pwmPinA, HIGH);
  digitalWrite(pwmPinB, LOW);
  delay(halfPeriod);

  digitalWrite(pwmPinA, LOW);
  digitalWrite(pwmPinB, HIGH);
  delay(halfPeriod);
}
```

### Simulation
**File:** `Arduino_Simulation.pdsprj`

The simulation file was modified to test the driver logic. The induction coil and capacitor were replaced with LEDs, as Proteus had issues simulating the full LC tank.

![Arduino Driver Simulation Circuit](images/arduino_simulation_circuit.png)


The oscilloscope results show the two gate signals (Channels C, D) and the resulting alternating square wave outputs at the drains of the two MOSFETs (Channels A, B).

![Arduino Driver Simulation Results](images/arduino_simulation_results.png)


***

## 3. Circuit Design 2: Astable Multivibrator

This design uses a MOSFET-based astable multivibrator to create a self-oscillating circuit, removing the need for a microcontroller.

### Design Note & Calculation
The project specification calls for a 50 **kHz** resonant frequency, but the astable multivibrator calculation was performed for 50-200 **Hz**.

The calculation shown is for **50 Hz** using 10k resistors (though the circuit diagram uses 20k resistors).
* **Formula:** $f = 1 / (\ln(2) 	imes (R_1C_1 + R_2C_2))$
* **Calculation for 50 Hz (using 10k resistors):**
    $50 = 1 / (\ln(2) 	imes (10 	imes 10^3 	imes C + 10 	imes 10^3 	imes C))$
    $C = 0.721 \mu F$
* **Selected Component:** A **$0.75 \mu F$** capacitor was selected for the astable timing.

### Components
* Astable Capacitors (0.75uF) $	imes$ 2
* MOSFET (60N15) $	imes$ 2
* Resistors:
    * $2.3 K\Omega 	imes 2$
    * $20 K\Omega 	imes 2$
    * $1 K\Omega 	imes 2$ (R6, R7)
* Potentiometer (1K $\Omega$) $	imes$ 1
* 48V 25A Power Supply

### Simulation
**File:** `Astable_Multivibrator_Simulation.pdsprj`

Similar to the Arduino design, the simulation file was modified to test the core oscillator logic. The heater coil, tank capacitor, and choke inductors were removed, as they prevented the simulation from running correctly.

![Astable Multivibrator Simulation Circuit](images/astable_simulation_circuit.png)


The oscilloscope results show the characteristic charging/discharging "sawtooth" wave of the astable multivibrator at the MOSFET gates, confirming the circuit is oscillating.

![Astable Multivibrator Simulation Results](images/astable_simulation_results.png)


***

## Simulation Notes

In both designs, the main load (the resonant LC tank) was removed for simulation purposes. The simulation software (Proteus) failed to run the simulation with the heater coil and capacitor connected. The circuits provided in the simulation files are therefore modified to only test and validate the **driver logic** of the MOSFETs.