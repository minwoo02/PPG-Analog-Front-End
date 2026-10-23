# PPG Analog Front-End

### LM358-Based Analog Signal Conditioning for Photoplethysmography

An analog front-end (AFE) circuit designed to amplify and filter photoplethysmography (PPG) signals using LM358 operational amplifiers and passive RC filters.

This project focuses on **analog circuit design, SPICE simulation, and hardware validation**. The goal is to acquire a PPG signal from an optical pulse sensor and condition it using a custom analog circuit without relying on a microcontroller or digital signal processing.

**Project Status:** PSpice Simulation Completed | Hardware Implementation Planned

---

## 1. Project Overview

Photoplethysmography (PPG) is an optical measurement technique that detects changes in blood volume through variations in light absorption or reflection.

A PPG sensor typically produces a signal containing a small pulsatile component along with a DC component and unwanted interference.

To extract the pulsatile signal, an analog front-end is designed to perform three main functions:

- **DC Removal:** Attenuate unwanted DC and low-frequency components.
- **Signal Amplification:** Amplify low-amplitude PPG signals using operational amplifiers.
- **Noise Filtering:** Attenuate high-frequency interference while preserving the desired pulse waveform.

The final objective is to construct the circuit using discrete components and observe the conditioned PPG waveform directly with an oscilloscope.

## 2. System Architecture

The proposed system consists of two high-pass filters, two non-inverting amplification stages, and one low-pass filter.

```text
       PPG Sensor (KY-039)
               |
               v
       High-Pass Filter 1
          fc = 0.482 Hz
               |
               v
       Non-Inverting Amp 1
            Gain = 11
               |
               v
       High-Pass Filter 2
          fc = 0.482 Hz
               |
               v
       Non-Inverting Amp 2
            Gain = 11
               |
               v
        Low-Pass Filter
          fc = 4.823 Hz
               |
               v
             VOUT
               |
               v
          Oscilloscope
```

A separate voltage reference circuit provides a stable bias for the analog signal-processing stages.

### Design Specifications

| Parameter | Design Value |
|---|---|
| Supply Voltage | 5 V DC |
| Operational Amplifier | LM358 |
| Reference Voltage | Approximately 1.667 V |
| HPF1 Cutoff Frequency | 0.482 Hz |
| First Amplifier Gain | 11 V/V |
| HPF2 Cutoff Frequency | 0.482 Hz |
| Second Amplifier Gain | 11 V/V |
| LPF Cutoff Frequency | 4.823 Hz |
| Nominal Cascaded Amplifier Gain | 121 V/V |
| Intended Sensor | KY-039 PPG Sensor |
| Output Measurement | Oscilloscope |

The actual overall gain depends on signal frequency due to attenuation introduced by the filters.

## 3. Circuit Design

### 3.1 Voltage Reference

Because the circuit operates from a single 5 V supply, a DC reference voltage is required for signal conditioning.

A resistor divider generates approximately 1.667 V:

\[
V_{REF}=V_{CC}\frac{R_2}{R_1+R_2}
\]

With \(R_1=20\,k\Omega\) and \(R_2=10\,k\Omega\):

\[
V_{REF}\approx1.667\,V
\]

An LM358 voltage follower buffers this reference for use by the subsequent stages.

### 3.2 High-Pass Filters

Two RC high-pass filters are used to attenuate DC components and low-frequency variations.

Each filter uses:

- Resistor: 330 kΩ
- Capacitor: 1 µF

The cutoff frequency is:

\[
f_c=\frac{1}{2\pi RC}\approx0.482\,Hz
\]

The second HPF also prevents the DC offset produced by the first amplification stage from being directly amplified by the second stage.

### 3.3 Amplification Stages

Two non-inverting LM358 amplifier stages provide signal amplification.

Each stage uses:

- Feedback resistor: 100 kΩ
- Gain-setting resistor: 10 kΩ

The nominal voltage gain is:

\[
A_v=1+\frac{R_f}{R_g}=11
\]

The two stages therefore provide a nominal cascaded gain of:

\[
A_{v,\mathrm{amp}}=11\times11=121
\]

The actual gain of the complete AFE is lower at 1.2 Hz because of the HPF and LPF frequency responses.

### 3.4 Low-Pass Filter

A passive RC low-pass filter attenuates higher-frequency interference.

- Resistor: 33 kΩ
- Capacitor: 1 µF

The cutoff frequency is:

\[
f_c\approx4.823\,Hz
\]

Together, the two HPFs and the LPF produce a band-pass-shaped overall frequency response.

## 4. Simulation Environment

The analog front-end was designed and verified using **PSpice for TI 23.1**.

| Item | Configuration |
|---|---|
| Simulation Software | PSpice for TI 23.1 |
| Op-Amp Model | STMicroelectronics LM358 |
| Power Supply | 5 V |
| Simulation Types | Transient, AC Sweep, FFT |
| Test Input | Simulated PPG waveform |

An external LM358 SPICE model and a custom schematic symbol were configured for simulation.

### Simulated PPG Input

A sinusoidal test signal was used to evaluate the circuit before connecting a physical sensor.

\[
V_{IN}(t)=2+0.005\sin(2\pi\cdot1.2t)
\]

| Parameter | Value |
|---|---|
| DC Offset | 2 V |
| AC Amplitude | 5 mV peak |
| Frequency | 1.2 Hz |
| Equivalent Heart Rate | 72 BPM |

**Note:** This signal is generated by an ideal simulation source, not measured from a physical KY-039 sensor.

## 5. Simulation Results

### 5.1 Stage-by-Stage Verification

Each stage was simulated individually before integration into the complete circuit.

| Circuit Stage | Simulation Result |
|---|---|
| Voltage Reference | 1.66674 V |
| HPF1 | Approximately 92.8% amplitude transmission at 1.2 Hz |
| Amplifier 1 | Approximately 11× gain |
| HPF2 | Approximately 92.8% amplitude transmission at 1.2 Hz |
| Amplifier 2 | Approximately 11× gain |
| LPF | Approximately 97.0% amplitude transmission at 1.2 Hz |

The results closely matched the theoretical circuit calculations.

### 5.2 Complete AFE Output

For a 1.2 Hz sinusoidal input with an amplitude of 5 mV peak:

| Parameter | Input | Output |
|---|---:|---:|
| Signal Frequency | 1.2 Hz | 1.2 Hz |
| AC Amplitude | 5 mV peak | Approximately 505 mV peak |
| Peak-to-Peak Voltage | 10 mV | Approximately 1.01 V |
| AC Voltage Gain | — | Approximately 101.07× |

The complete circuit amplified the simulated PPG component by approximately **101×** while maintaining its waveform.

### 5.3 AC Sweep Analysis

An AC sweep from 0.01 Hz to 1 kHz was performed to evaluate the frequency response of the complete circuit.

| Frequency | Voltage Gain |
|---|---:|
| 1.2 Hz | 40.092 dB |
| 1.862 Hz | 40.486 dB |
| 50 Hz | 21.299 dB |

The maximum simulated gain occurred near 1.86 Hz.

The overall frequency response confirmed the combined filtering characteristics of the two HPFs and one LPF.

### 5.4 50 Hz Interference Test

A 50 Hz sinusoidal interference component was added to the simulated PPG signal.

\[
V_{IN}(t)=2
+0.005\sin(2\pi\cdot1.2t)
+0.001\sin(2\pi\cdot50t)
\]

The FFT results were:

| Frequency Component | Input Amplitude | Output Amplitude |
|---|---:|---:|
| 1.2 Hz PPG Signal | 5.00015 mV | 505.357 mV |
| 50 Hz Interference | 0.99970 mV | 11.607 mV |

The 50 Hz-to-1.2 Hz amplitude ratio decreased from approximately **19.99% at the input** to **2.30% at the output**.

This corresponds to an improvement of approximately **18.8 dB in the relative interference level**.

Although the absolute amplitude of the 50 Hz component increased, it became significantly smaller relative to the desired 1.2 Hz signal.

This result describes rejection of the applied sinusoidal interference relative to the desired signal, rather than a general broadband noise measurement.

## 6. Hardware Implementation

The next phase is to construct the simulated analog circuit on a breadboard.

### Planned Components

- KY-039 optical pulse sensor
- LM358N dual operational amplifier ICs (2 units)
- Resistors and capacitors
- Breadboard and jumper wires
- Regulated 5 V supply
- Digital multimeter
- Oscilloscope

### Hardware Verification Plan

1. Measure the actual output characteristics of the KY-039 sensor.
2. Construct and verify the voltage reference circuit.
3. Implement HPF1 and the first amplification stage.
4. Add HPF2 and the second amplification stage.
5. Implement the output LPF.
6. Observe the conditioned PPG signal using an oscilloscope.
7. Compare the experimental results with PSpice simulations.

The complete system will use analog signal conditioning only, without Arduino-based or other digital signal processing.

## 7. Limitations

The current results are based on circuit simulation.

The following aspects have not yet been experimentally verified:

- Actual KY-039 output amplitude and DC offset
- Hardware noise and electromagnetic interference
- Component tolerances and breadboard parasitics
- Differences between the simulated LM358 model and physical LM358N devices
- Amplifier saturation under real sensor conditions
- Actual PPG waveform acquisition from a finger

These limitations will be investigated during hardware implementation.

## 8. Project Files

The repository contains the PSpice project files and associated simulation settings.

```text
PPG_Analog_Front_End.opj
PPG_ANALOG_FRONT_END.DSN
PPG_ANALOG_FRONT_END.png
PPG_Analog_Front_End-PSpiceFiles/
```

The `.opj` and `.dsn` files contain the PSpice project configuration and schematic designs.

The `PSpiceFiles` directory contains the simulation configurations associated with individual circuit stages.

Simulation-generated output files are generally excluded from Git version control.

### External LM358 Model

This project uses an externally supplied STMicroelectronics LM358 SPICE model and a corresponding custom schematic symbol.

The model and symbol were configured outside the main project directory. Users opening the project on another computer may need to install the model and update the PSpice library references.

Third-party library files are not included unless their redistribution terms permit it.

## 9. Future Work

- [x] Analog front-end circuit design
- [x] Voltage reference verification
- [x] HPF and amplifier stage simulations
- [x] LPF simulation
- [x] Complete circuit transient simulation
- [x] AC frequency response analysis
- [x] 50 Hz interference test
- [x] FFT frequency analysis
- [ ] Breadboard circuit implementation
- [ ] KY-039 sensor characterization
- [ ] Oscilloscope measurement of real PPG signals
- [ ] Simulation-to-hardware comparison
- [ ] Final experimental documentation

## 10. Project Objective

This project aims to develop practical experience in **analog bioelectronic circuit design, small-signal amplification, filtering, SPICE simulation, and experimental verification**.

By progressing from theoretical calculations and circuit simulation to physical implementation, the project explores the engineering challenges involved in acquiring and conditioning biological signals.

---

**Current Stage:** Simulation validated; hardware experiments pending.