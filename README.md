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
* **Coil Diameter ($l_{coil}$):** 40 mm (0.04 m)
* **Coil Radius (r):** 20 mm (0.02 m)
* **Number of Turns (N):** 10

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
    * Calculated Current: $I = 989.59W / 48V = 20.62~A$
    * **Conclusion:** A **48V, 25A power supply** is a safe and appropriate choice.

### Coil & Resonant Tank

1.  **Coil Inductance (L):**
    Using the standard formula for a short solenoid, $L = rac{\mu_0 N^2 A}{l_{coil}}$, where $A = \pi r^2$.
    * $A = \pi 	imes (0.02~m)^2 = 0.001257~m^2$
    * $L = rac{(4\pi 	imes 10^{-7} 	imes 10^2 	imes 0.001257)}{0.04~m}$
    * $L = 3.947 	imes 10^{-6} H pprox$ **$3.95~\mu H$**

2.  **Required Capacitance (C):**
    To achieve the resonant frequency (f) of 50 kHz with the $3.95~\mu H$ coil:
    $C = 1 / ((2 \pi f)^2 	imes L)$
    $C = 1 / ((2 \pi 	imes 50 	imes 10^3)^2 	imes 3.95 	imes 10^{-6})$
    $C = 2.56 	imes 10^{-6} F pprox$ **$2.57~\mu F$**

### Heat Sink Design

1.  **MOSFET Power Dissipation ($P_D$):**
    Using a worst-case current $I_D$ of **25 A** (the max rating of the chosen power supply) and the SNW60N15 MOSFET ($R_{DS(on)} = 33~m\Omega$):
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

The calculation shown is for **50 Hz**. Using the **20k $\Omega$** resistors (as shown in the diagram and component list):
* **Formula:** $f = 1 / (\ln(2) 	imes (R_1C_1 + R_2C_2))$
* **Calculation:**
    $50 = 1 / (0.693 	imes (20 	imes 10^3 	imes C + 20 	imes 10^3 	imes C))$
    $50 = 1 / (0.693 	imes 40 	imes 10^3 	imes C)$
    $C = 1 / (50 	imes 0.693 	imes 40 	imes 10^3)$
    $C = 0.721 	imes 10^{-6} F$
* **Selected Component:** A **$0.75~\mu F$** capacitor was selected for the astable timing.

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

![Astable Multivivbrator Simulation Circuit](images/astable_simulation_circuit.png)


The oscilloscope results show the characteristic charging/discharging "sawtooth" wave of the astable multivibrator at the MOSFET gates, confirming the circuit is oscillating.

![Astable Multivivbrator Simulation Results](images/astable_simulation_results.png)


***

## Simulation Notes

In both designs, the main load (the resonant LC tank) was removed for simulation purposes. The simulation software (Proteus) failed to run the simulation with the heater coil and capacitor connected. The circuits provided in the simulation files are therefore modified to only test and validate the **driver logic** of the MOSFETs.