# TOQSO-OTA-Analog-Design
# Third-Order Quadrature Sinusoidal Oscillator (TOQSO)
### OTA-Based Analog IC Design | Cadence Virtuoso 6.1.8 | Spectre Simulation

---

## Overview

This project presents the design and simulation of a **Third-Order Quadrature Sinusoidal Oscillator (TOQSO)** built using a custom transistor-level **Operational Transconductance Amplifier (OTA)** designed from scratch in Cadence Virtuoso on a generic 180nm CMOS process.

The oscillator generates three sinusoidal output voltages — **Vo1, Vo2, and Vo3** — with approximately **90° phase quadrature** between successive outputs. Two distinct TOQSO topologies (`Final_TOQSO_2` and `Final_TOQSO_3`) were implemented using the same `OTA_new` subcircuit and verified independently via transient Spectre simulation.

**Applications:**
- I/Q signal generation in communication systems
- Phase-sensitive detection and signal processing
- Analog waveform synthesis

---

## Key Specifications

| Parameter                  | Value                                    |
|----------------------------|------------------------------------------|
| Technology                 | 180nm CMOS (generic PDK)                 |
| Supply Voltage             | VDD = +900 mV, VSS = −900 mV (vdc = 900 mV) |
| Tail bias supply           | 900 mV (independent DC source)           |
| Oscillation Frequency      | ~10 MHz *(verify with Virtuoso markers)* |
| Steady-state amplitude     | ~±0.65 V (TOQSO_2), ~±0.4 V (TOQSO_3)  |
| Inter-output phase shift   | ~90° (quadrature)                        |
| Integrating capacitors     | C0, C1, C2 = 1 pF each                  |
| OTA instances per topology | 4                                        |
| Simulation type            | Transient (Spectre via ADE L)            |
| Simulation duration        | 1 µs                                     |
| Oscillation startup        | ~0.1–0.15 µs from initial conditions    |

---


---

## OTA Design (`OTA_new`)

The `OTA_new` cell is the core building block of both TOQSO topologies. It was designed at the transistor level from scratch, providing the transconductance (Gm) that satisfies the Barkhausen gain condition when embedded in the integrating feedback loop.

**Cell stats:** 19 instances · 12 nets · 3 pins (Vminus, Vplus, Iout)

---

### Architecture

The OTA uses a standard **differential pair with active current mirror load** topology:

| Stage                   | Devices        | Type  | Role                                          |
|-------------------------|----------------|-------|-----------------------------------------------|
| Differential input pair | NM_left, NM_right | **NMOS** | Converts differential input voltage to current |
| Active load / mirror    | PM0, PM1       | PMOS  | W=1.44µm — current mirror load               |
| Bias mirror (outer)     | PM2, PM3       | PMOS  | W=2.88µm — sets mirror reference             |
| Tail current source     | NM_tail (×2)   | NMOS  | W=5.4µm — sets differential pair bias current |
| Tail bias supply        | vdc = 900 mV   | DC    | Independent voltage source for tail gate bias  |
| Bias resistor           | R ≈ 50 kΩ      | R     | Sets tail gate VGS at the bias node           |
| Output                  | Iout pin       | —     | Single-ended current output from PM3 drain    |

---

### Device Sizing

| Device role             | Type  | W (µm) | L (nm) | m  |
|-------------------------|-------|---------|--------|----|
| NMOS differential pair  | NMOS  | 3.6     | 360    | 1  |
| PMOS inner load (PM0, PM1) | PMOS | 1.44  | 360    | 1  |
| PMOS outer mirror (PM2, PM3) | PMOS | 2.88 | 360   | 1  |
| NMOS tail current source (×2) | NMOS | 5.4 | 360  | 1  |
| Bias resistor           | R     | —       | —      | ~50 kΩ |

> All channel lengths are uniform at **L = 360 nm**.
> The tail NMOS uses a larger W = 5.4 µm to provide sufficient tail current for the chosen bias point.
> PM0 is diode-connected, setting the shared gate voltage for all four PMOS devices.
> Tail gate bias is supplied by an independent **900 mV DC source** through the bias resistor network.

---

### Supply Configuration

| Rail  | Value     | Connected to                              |
|-------|-----------|-------------------------------------------|
| VDD   | +900 mV   | Sources of all PMOS (PM0–PM3)             |
| VSS   | −900 mV   | Sources of NMOS diff pair and tail devices |
| vdc   | +900 mV   | Tail gate bias resistor network           |

The supply is symmetric (±900 mV), giving a total supply swing of 1.8 V.

---

### OTA Schematic
![OTA Transistor Level](schematics/OTA_transistor_level.png)

---

## TOQSO Topologies

Both topologies use **four `OTA_new` instances** and **three grounded 1 pF integrating capacitors** in a closed loop. The third-order loop provides a total phase shift of 360° at the oscillation frequency, with each OTA-C integrator stage contributing −90°, satisfying the **Barkhausen phase criterion**.

The two topologies differ in the **interconnection order** of OTA instances and the **input polarity** (Vminus/Vplus orientation) of individual OTAs — producing different amplitude characteristics while maintaining the same quadrature phase relationship.

---

### Topology 2 — `Final_TOQSO_2`

- **OTA instances:** I0, I4, I5, I6
- **Capacitors:** C0, C1, C2 = 1 pF; C1 bootstrapped with `ic=1` to initiate startup
- **Output nodes:** Vo1 (top), Vo2 (bottom-left), Vo3 (bottom-right)
- All OTAs in standard orientation (Vplus/Vminus signal path consistent)

![Topology 2 Schematic](schematics/TOQSO_topology_2.png)

---

### Topology 3 — `Final_TOQSO_3`

- **OTA instances:** I0, I1, I2, I3
- **Capacitors:** C0, C1, C2 = 1 pF; C2 bootstrapped with `ic=1V`
- **Output nodes:** Vo1 (top-right), Vo2 (middle-left), Vo3 (top-left)
- Some OTAs have **Vminus/Vplus swapped** (inverting stage) to close the 360° phase loop correctly

![Topology 3 Schematic](schematics/TOQSO_topology_3.png)

---

## Simulation Results

Transient simulation was performed using **Cadence Spectre** via **ADE L** over 1 µs. Capacitor initial conditions (`ic`) were used to bootstrap oscillator startup.

---

### Topology 2 — Transient Response (`Final_TOQSO_2`)

**Signals:** Vo1 (red), Vo2 (green), Vo3 (hidden/red)

- Steady-state amplitude: **~±0.65–0.70 V**
- Startup time: **~0.1 µs**
- Period: **~0.1 µs → f₀ ≈ 10 MHz**
- Clear ~90° phase offset between Vo1 and Vo2 in steady state

![Waveform Topology 2](waveforms/transient_TOQSO_2.png)

---

### Topology 3 — Transient Response (`Final_TOQSO_3`)

**Signals:** Vo1 (hidden), Vo2 (red), Vo3 (orange)

- Steady-state amplitude: **~±0.4 V**
- Startup time: **~0.15 µs**
- Period: **~0.1 µs → f₀ ≈ 10 MHz**
- Quadrature phase relationship between Vo2 and Vo3 consistent with Topology 2

![Waveform Topology 3](waveforms/transient_TOQSO_3.png)

---

### Summary of Results

| Parameter              | Topology 2 (TOQSO_2) | Topology 3 (TOQSO_3) |
|------------------------|----------------------|----------------------|
| Steady-state amplitude | ~±0.65–0.70 V        | ~±0.4 V              |
| Oscillation frequency  | ~10 MHz              | ~10 MHz              |
| Startup time           | ~0.1 µs              | ~0.15 µs             |
| Phase shift (outputs)  | ~90°                 | ~90°                 |
| Bootstrap condition    | C1: ic=1             | C2: ic=1V            |

> The amplitude difference between topologies is due to the different OTA polarity arrangement and effective loop gain at steady state — both topologies confirm stable oscillation at the same frequency.

---

## Theory

### Oscillation Frequency

Each OTA-C integrator stage has a transfer function:

$$H(s) = \frac{G_m}{sC}$$

The oscillation frequency of the third-order loop is:

$$f_0 = \frac{G_m}{2\pi C}$$

where:
- **Gm** = OTA transconductance [A/V], set by the tail bias current and differential pair sizing
- **C** = integrating capacitor = **1 pF**

Primary tuning handle: adjusting the tail bias current (via the 900 mV source and 50 kΩ resistor) shifts Gm and directly shifts f₀.

---

### Barkhausen Criterion

For sustained sinusoidal oscillation, two conditions must hold simultaneously at ω₀:

1. **Phase condition:** Total loop phase = 360°
2. **Gain condition:** |Loop gain| = 1

In the third-order TOQSO, each OTA-C stage contributes −90° at ω₀. Three integrator stages × (−90°) = −270°. The remaining −90° is contributed by the inverting OTA stage(s) in the topology (achieved by swapping Vminus/Vplus), closing the full 360° loop.

---

### Sensitivity

| Parameter         | Effect on oscillator                          |
|-------------------|-----------------------------------------------|
| Gm (via Ibias)    | Directly sets f₀ — primary tuning knob        |
| Tail bias voltage | Controls Ibias → controls Gm → controls f₀   |
| Capacitor value   | Sets f₀; mismatch degrades phase accuracy     |
| Temperature       | Shifts µn, VT → shifts Gm → shifts f₀        |
| Process variation | Affects VT and µn/µp → changes Gm and Ibias  |

Iterative W/L sizing and bias resistor tuning were performed to achieve a stable operating point and consistent oscillation within the ±900 mV supply.

---

## Tools Used

| Tool                                       | Purpose                               |
|--------------------------------------------|---------------------------------------|
| Cadence Virtuoso Schematic Editor L 6.1.8  | Transistor-level schematic, hierarchy |
| Cadence Spectre (ADE L)                    | Transient simulation                  |
| Virtuoso Visualization & Analysis XL       | Waveform viewing, phase measurement   |

---

## Author

**Anushka Kashyap**
B.Tech, Electronics and Communication Engineering (2023–2027)
Netaji Subhas University of Technology (NSUT), Delhi
[LinkedIn](https://www.linkedin.com/in/anushka-kashyap-290158325) | [GitHub](https://github.com/anushkakashyap003)

---

## Notes

> Cadence Virtuoso library files (`.cdslck`, cellview database) are proprietary binary formats and are not included in this repository. The SPICE/CDL netlist exported from Virtuoso is provided under `/netlist/` for reference. All design details, schematics, and simulation results are documented via screenshots and this README.
