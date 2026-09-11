# 3×3 Sagnac Interferometric Fiber-Optic Gyroscope PoC Using Off-the-Shelf Telecom Components

English | [日本語](README.md)

> **Goal:** Observe the Sagnac phase and the direction of rotation with a 3×3 configuration built mainly from off-the-shelf parts.<br>
> **Baseline configuration:** 1550 nm / 100 m SMF / nominal diameter 200 mm / 3×3 coupler / 3-port circulator / 3 detector channels<br>
> **Measurand:** Angular-rate component projected onto the normal of the coil plane<br>
> **PoC completion criterion:** Acquire the three returning interference outputs and, after calibration, recover positive and negative known angular rates with the correct sign<br>
> **Scope:** For proof of principle, education, and short-duration simple rotation measurements. The values below are design values and do not guarantee measured performance.

---

## Abstract

This document describes a simple fiber-optic Sagnac rotation sensor assembled from commercially available 1550 nm telecom components. Instead of building a ring resonator in free space, a long single-mode fiber is wound into a multi-turn coil so that the Sagnac phase can be observed with a comparatively simple optical setup.

A 3×3 coupler, a 3-port circulator, a 100 m coil, and three detector channels are combined to estimate the direction of rotation and the phase from the returning interference outputs of light that has propagated through the same coil in opposite directions. Using 100 m, a 200 mm diameter, and a 1550 nm wavelength as the baseline design, the document covers the component configuration, connections, signal processing, calibration, and the main error sources.

The ideal formulas apply only when the three detector values are 120° phase-shifted outputs of the same interference signal. Check the port definitions of the purchased parts and how the returning light is extracted; parts from which all three outputs cannot be obtained as interference outputs are not used in this PoC.

---

## 1. Background

One of the fundamental principles for detecting rotation optically is the **Sagnac effect**. When an entire closed optical path rotates, two light waves propagating around it in opposite directions have slightly different transit times even if they depart at the same moment. This difference can be observed as an optical phase difference, which is proportional to the angular rate.

In a ring laser gyro, the closed path itself is built as a laser resonator, and the frequency difference between the counter-propagating lasing modes is read out. In the approach described here, an external light source is fed into a 3×3 coupler, two of its ports are connected to the two ends of the same fiber coil, and the light propagates CW (clockwise) and CCW (counterclockwise) before recombining. The remaining port is optically terminated, and interference outputs are taken from the three returning ports. This makes the same Sagnac effect available as an interference phase without building a laser resonator.

The main reason for choosing a fiber-based approach is that **a long optical path and a large effective enclosed area can be obtained simply by winding a multi-turn coil, with no precise mirror alignment required**. In the 1550 nm band, telecom components such as single-mode fiber, couplers, light sources, and InGaAs detectors are also readily available off the shelf.

This setup does not aim directly at a high-precision inertial sensor. The goal is to establish an actual Sagnac rotation measurement in the following order:

1. Form the two optical paths, clockwise (CW) and counterclockwise (CCW), with the 3×3 coupler and the coil
2. Acquire the three returning interference outputs in the same ADC frame
3. Recover the phase, including the direction of rotation, from the three calibrated channels
4. Calibrate the scale, sign, and repeatability with known angular rates

---

## 2. Design Approach

### 2.1 Why 1550 nm

1550 nm is chosen because a complete set can easily be assembled from common telecom components without assuming special research-grade optics.

- Long single-mode fiber is easy to obtain
- 3×3 couplers and their port specifications can be selected
- A 3-port optical circulator can separate the transmitted and returning light
- InGaAs detectors detect it directly
- FC-type connectors allow a detachable setup

### 2.2 Division of Roles in the PoC

Below, `CW` for the light source (continuous wave) is kept distinct from `CW` for the propagation direction (clockwise).

In the 3×3 configuration, appropriate port wiring yields three returning interference outputs with different phases. Feeding the three channels into `atan2` recovers the phase and distinguishes clockwise from counterclockwise rotation.

In the PoC, light is launched from one port of the 3×3 coupler, two ports are connected to the two ends of the coil, and the remaining port is optically terminated. The returning light is taken from the three ports on the opposite side, and the port shared with the light source is separated to a detector by a 3-port circulator. Use only parts for which the manufacturer's port documentation or a measurement confirms that the three returning ports are outputs of the same interference signal.

---

## 3. Principle

Let $\boldsymbol{n}$ be the unit normal vector of the coil plane and $\boldsymbol{\Omega}$ the angular-rate vector. The measurand is defined as

$$
\Omega_n=\boldsymbol{\Omega}\cdot\boldsymbol{n}
$$

The sensor directly measures this normal component, not the full rotation vector.

When a closed optical path of area $A$ rotates, two counter-propagating light waves acquire a transit-time difference of

$$
\Delta t=\frac{4A\Omega_n}{c^2}
$$

With $\lambda$ as the vacuum wavelength, the Sagnac phase difference due to rotation is

$$
\Delta\phi_{\mathrm{S}}=\frac{8\pi A}{\lambda c}\Omega_n
$$

In a multi-turn fiber coil, the areas of the individual turns add up. For a circular coil of constant radius, with fiber length $L$ in the coil and radius $r$,

$$
A_{\mathrm{eff}}\approx\frac{Lr}{2}
$$

so that

$$
\boxed{
\Delta\phi_{\mathrm{S}}=
\frac{4\pi Lr}{\lambda c}\Omega_n
}
$$

In a real coil with multiple layers and leads, the radius differs from turn to turn, so the effective area is evaluated as

$$
A_{\mathrm{eff}}=\sum_k\pi r_k^2
$$

or approximated with the measured mean radius. Do not fix the scale factor from the nominal diameter alone.

The theoretical scale factor is

$$
\boxed{
K_{\phi,\mathrm{theory}}=\frac{4\pi Lr}{\lambda c}
}
$$

and the measured value is calibrated as $K_{\phi,\mathrm{meas}}$.

The actual demodulated value contains a static phase bias and drift:

$$
\phi_{\mathrm{raw}}=\phi_{\mathrm{bias}}+\Delta\phi_{\mathrm{S}}+\epsilon
$$

The angular rate is therefore obtained after correcting the phase bias and, if necessary, unwrapping the phase:

$$
\boxed{
\Omega_n=\frac{\mathrm{unwrap}(\phi_{\mathrm{raw}}-\phi_{\mathrm{bias}})}{K_{\phi,\mathrm{meas}}}
}
$$

### Baseline Design

| Parameter | Value |
|---|---:|
| Wavelength | 1550 nm |
| Fiber length in coil | nominal 100 m |
| Coil diameter | nominal 200 mm |
| Radius | 0.10 m (single-layer approximation) |
| Number of turns | approx. 159 (for constant radius) |
| Effective area | approx. 5.0 m² (for constant radius) |

The theoretical value is then

$$
K_{\phi,\mathrm{theory}}\approx0.2704\ \mathrm{s}
$$

1 rpm is converted as $2\pi/60$ rad/s.

| Rotation rate | Theoretical phase difference |
|---:|---:|
| 1 rpm | 0.028 rad |
| 5 rpm | 0.142 rad |
| 10 rpm | 0.283 rad |
| 30 rpm | 0.850 rad |
| 60 rpm | 1.70 rad |

**About 10–30 rpm** is convenient for the first functional check. Whether the signal is observable depends on the fringe visibility, phase bias, TIA saturation, and ADC noise.

---

## 4. PoC Specification

The only configuration built here is the 3×3 configuration that recovers the phase and the direction of rotation from the three returning interference outputs. The optics, detectors, ADC, and MCU are fixed on the same turntable, and the only connection to the stationary side is an electrical slip ring (power and UART).

The required blocks are:

- 1550 nm continuous-wave source → 3-port circulator → one port of the 3×3 single-mode coupler
- CW/CCW paths formed by two coupler ports, a manual polarization controller in the loop, and the 100 m SMF coil
- Optical termination of the unused port
- Three returning interference outputs → InGaAs detectors → TIAs → ADS1115
- Three-channel correction, phase recovery, unwrapping, and angular-rate conversion on the MCU
- Power and UART through the electrical slip ring, with frames and reference rate logged on a PC
- A turntable that can hold constant speeds from −120 to +120 rpm, with a rate reference

The PoC is accepted when all of the following are met:

1. The purchased part's port documentation and the measurement criteria in Section 11.2 confirm that the three returning ports are interference outputs carrying independent phase information.
2. After dark-offset correction, the three channels can be acquired as one frame without saturation.
3. The sign of the phase reverses between known positive and negative rotation, and the calibrated angular rate follows the same trend as the reference.
4. The scale factor obtained at multiple rates, the residuals, and the zero-point repeatability can be recorded.

---

## 5. Wiring and Direction Sensing

### 5.1 Wiring Conditions

The 3×3 coupler is treated as a bidirectional component with two sides of three ports each. `L0`–`L2` and `R0`–`R2` below are logical names for explanation, not actual port numbers. Map them to the datasheet of the purchased coupler and record the serial number and the mapping table.

The following configuration receives three **returning interference outputs**. A 3-port optical circulator is required because the source port is shared with the third detector port.

```text
                ┌────────────┐
  source ───────┤P1          │
                │ circulator │     ┌─────────────────────┐
  PD2 ──────────┤P3        P2├─────┤L2                 R2├── coil end B
                └────────────┘     │                     │
  PD1 ─────────────────────────────┤L1   3×3 coupler   R1├── terminator
                                   │                     │
  PD0 ─────────────────────────────┤L0                 R0├── polarization controller ── coil end A
                                   └─────────────────────┘
```

Each PD feeds the ADS1115 through a TIA. The actual connections are fixed as follows.

| Logical port | Connection | Notes |
|---|---|---|
| L0 | PD0 → TIA → ADS1115 | Returning interference output |
| L1 | PD1 → TIA → ADS1115 | Returning interference output |
| L2 | Circulator P2 | Reciprocal port shared by transmit and return |
| R0 | Polarization controller → coil end A | Adjusts fringe visibility in the loop |
| R1 | Optical terminator | Do not leave the unused port open |
| R2 | Coil end B | Other end of the coil |
| Circulator P1 | 1550 nm source | Continuous-wave mode |
| Circulator P3 | PD2 → TIA → ADS1115 | Returning interference output from L2 |

In this PoC, all three of L0, L1, and L2 are used in the phase calculation as outputs of the same interference signal. Products for which the datasheet or a measurement cannot confirm that all three returning outputs are interference signals, and products that include a monitor output, are not used.

The source, circulator, coupler, polarization controller, coil, detectors, TIAs, ADC, and MCU are fixed on the same turntable. The only connection to the stationary side is the electrical slip ring (power and UART); no optical fiber runs between the rotating and stationary parts.

### 5.2 Ideal Model

Only when the three detector values are 120° phase-shifted outputs of the same interference signal and their offsets and gains have been corrected, define the normalized values $J_0,J_1,J_2$ as

$$
J_0=C[1+V\cos\phi]
$$

$$
J_1=C\left[1+V\cos\left(\phi+\frac{2\pi}{3}\right)\right]
$$

$$
J_2=C\left[1+V\cos\left(\phi-\frac{2\pi}{3}\right)\right]
$$

With this wiring, the CW and CCW paths that leave L2 and return to L2 are the same path traversed in opposite directions. Apart from polarization effects, reciprocity makes the coupling coefficients of the two waves equal, so the L2 output is $\cos\Delta\phi_{\mathrm{S}}$ with zero phase bias and reaches its interference maximum at rest. For an ideal symmetric coupler, the L0 and L1 outputs are shifted by ±120° relative to it. Therefore, assign PD2 to $J_0$ and PD0 and PD1 to $J_1$ and $J_2$. The assignment of $J_1$ and $J_2$ is chosen so that $\phi$ increases for a known positive rotation.

Then, from

$$
X=2J_0-J_1-J_2
$$

$$
Y=\sqrt{3}(J_2-J_1)
$$

the principal-value phase is obtained as

$$
\boxed{
\phi_{\mathrm{raw}}=
\mathrm{atan2}
\left(
\sqrt{3}(J_2-J_1),
2J_0-J_1-J_2
\right)
}
$$

In practice, the general model is

$$
\boldsymbol{v}=\boldsymbol{b}+M
\begin{bmatrix}\cos\phi\\\sin\phi\end{bmatrix}+\boldsymbol{\epsilon}
$$

and the three-channel offset $\boldsymbol{b}$, the gain and phase relation $M$, and the noise $\boldsymbol{\epsilon}$ are calibrated. Using identical parts alone does not make $M$ an ideal matrix.

After calibration, least squares gives

$$
\boldsymbol{q}=M^{+}(\boldsymbol{v}-\boldsymbol{b}),\qquad
\phi_{\mathrm{raw}}=\mathrm{atan2}(q_y,q_x)
$$

where $M^{+}$ is the pseudo-inverse of the calibration matrix. The ideal formula above serves as a quick check of whether the measured $M$ is close enough to the ideal form.

The phase bias is then corrected with the formulas in Section 3, the principal-value wrapping is unwrapped, and the result is converted to angular rate. The positive direction of $\Omega_n$ follows the right-hand rule about the normal $\boldsymbol{n}$ in Section 3, so record which face of the coil $\boldsymbol{n}$ points to.

---

## 6. Bill of Materials

### 6.1 Optics, Detection, and Control

| Part | Qty | Candidate |
|---|---:|---|
| 1550 nm CW source | 1 | [Joinwit JW3109](https://english.joinwit.com/product-info/96.html) |
| 3×3 SM coupler | 1 | [AC Photonics 3×3 Single Mode Fused Coupler](https://acphotonics.com/1x3-and-3x3-single-mode-fused-coupler) |
| 1550 nm 3-port optical circulator | 1 | [Thorlabs 6015-3-FC](https://www.thorlabs.com/newgrouppage9.cfm?objectgroup_id=373&pn=6015-3-FC) |
| Manual polarization controller | 1 | [Thorlabs FPC031](https://www.thorlabs.com/thorproduct.cfm?partnumber=FPC031) |
| 100 m SMF | 1 | [FS OS2 Custom Patch Cable](https://www.fs.com/products/12285.html) |
| Optical terminator (R1) | 1 | [Thorlabs FTFC1](https://www.thorlabs.com/thorproduct.cfm?partnumber=FTFC1) |
| FC/PC mating sleeves | as needed | Male-to-male connections (P2–L2, R0–polarization controller, polarization controller–coil end A, R2–coil end B, etc.) |
| InGaAs photodiode | 3 | [Thorlabs FGA01FC](https://www.thorlabs.com/item/FGA01FC) |
| TIA | 3 | [TI OPA381](https://www.ti.com/product/OPA381) |
| ADC | 1 | [ADS1115 (Adafruit 1085)](https://www.adafruit.com/product/1085) |
| MCU | 1 | [Arduino Nano Every](https://store.arduino.cc/products/nano-every) |
| Coil bobbin | 1 | φ200–300 mm |

### 6.2 Measurement, Mechanical, and Power

| Part | Qty | Purpose |
|---|---:|---|
| Dust caps (when idle) | as needed | Protect the terminator and connectors |
| 1550 nm optical power meter and attenuator | recommended | Check actual received power and prevent TIA saturation |
| TIA $R_f$, $C_f$, and photodiode bias circuit | per channel | Convert photocurrent to the ADC range |
| Power supply, regulators, decoupling parts | 1 set | Feed from a DC supply on the stationary side, regulate on the turntable, and supply the TIAs, ADC, and MCU |
| I²C wiring, pull-up resistors, board | 1 set | ADS1115 connection |
| Electrical slip ring | 1 | [Adafruit 736](https://www.adafruit.com/product/736). Two circuits for power and two for UART |
| USB–UART converter | 1 | Connects to the PC on the stationary side. Must support 5 V logic |
| Turntable | 1 | Holds constant speeds from −120 to +120 rpm in both directions and allows the slip ring to be mounted coaxially |
| Rotation-rate reference | 1 | Encoder, tachometer, or rate output of a calibrated turntable |
| Mechanical fixtures and protective cover | 1 set | Fix the coil and the optical and electrical parts to the same rigid body |

---

## 7. Component Notes

### Light Source — Joinwit JW3109

This is a candidate part whose configuration must be specified when ordering. Select a 1550 nm LD, continuous-wave mode, and no modulation. The JW3109 offers a choice of emitters such as FP-LD and LED, and a modulation mode is also available at 1550 nm, so never specify only "1550 nm". According to the manufacturer's specifications, the LD output is −7 dBm, the connector is FC/PC, and it runs on three AA batteries. Use it with the 10-minute auto-off function disabled. Also check the spectral width and the tolerance to back-reflected light.

---

### 3×3 Coupler — AC Photonics

Specify a 1550 nm, single-mode, 3×3 fused coupler. Wire the ports only after mapping the logical ports in Section 5 to the datasheet of the purchased part.

Ordering guideline:

```text
Configuration : 3x3
Wavelength    : 1550 nm
Split         : approximately 33:33:33
Fiber         : single mode
Connector     : FC/PC
Port report   : check insertion loss, uniformity, PDL, directivity, and port phase or its measurement method
```

Split ratio, PDL, and port phase characteristics of 3×3 couplers differ between products. If the standard specifications do not state a 120° phase relation, confirm it with the manufacturer's unit data or by measurement. Confirm that the same interference signal can be taken from all three returning ports; do not use a product for which this cannot be confirmed.

---

### 3-Port Optical Circulator — Thorlabs 6015-3-FC

It uses SM fiber with FC/PC connectors and is rated for 1525–1610 nm, which includes 1550 nm. The [manufacturer's V21 specification sheet](https://www.thorlabs.com/catalogpages/V21/1119.pdf) shows the P1→P2 and P2→P3 transmission paths. The manufacturer specifies an insertion loss of 0.8 dB (typ.) and an isolation above 40 dB. Light returning from the coupler is routed to P3 and barely reaches P1 on the source side, so no separate optical isolator is used. Check the current specifications and availability when ordering, and do not confuse it with the FC/APC version.

Connect source→P1, P2→coupler L2, and P3→PD2. Match the port numbers with the labels and documentation of the delivered unit, and before connecting the coupler, confirm P1→P2 and P2→P3 transmission with low-power light and a power meter.

---

### Polarization Controller — Thorlabs FPC031

A manual polarization controller with three paddles (Ø27 mm), rated for 1260–1625 nm, with built-in fiber terminated in FC/PC connectors. Insert it between coupler R0 and coil end A. In an SMF loop without polarization compensation, the birefringence of the loop makes the polarization states of the CW and CCW waves differ, which reduces the fringe visibility. In the worst case the interference almost disappears, so maximize the visibility with the procedure in Section 11.1 and then leave it fixed.

---

### 100 m Fiber — FS OS2 Custom Patch Cable

Ordering example:

```text
Fiber mode    : OS2
Fiber count   : Simplex
Connector A   : FC/PC (if FC/UPC is chosen, use UPC for all optical parts)
Connector B   : FC/PC (same as above)
Fiber grade   : G.657.A1 / A2
Length        : 100 m
Cable diameter: check the specification
Minimum bend  : check the specification
```

For winding at a diameter of about 200 mm, bend-insensitive G.657 fiber is easier to handle. Check the cable outer diameter, minimum bend radius, winding width, and number of layers, and calculate in advance whether 100 m fits on the bobbin.

---

### Optical Terminator — Thorlabs FTFC1

An FC/PC optical terminator with a return loss of 50 dB or more. Attach it to the R1 connector directly or through a mating sleeve, depending on the connector type.

---

### InGaAs Photodiode — Thorlabs FGA01FC

Fiber-coupled InGaAs PIN photodiode for 1550 nm.

It has no built-in amplifier, so a TIA is required after it. Estimate the photocurrent from the optical power and the wavelength-dependent responsivity, and keep it below the TIA saturation voltage. Take care with ESD.

---

### TIA — Texas Instruments OPA381

Converts the photodiode current to a voltage. Design the circuit including the OPA381 supply voltage, the photodiode reverse bias, output headroom, and input capacitance.

These are ranges for initial study and must not be used unconditionally.

```text
Rf : 10 kΩ – 100 kΩ
Cf : a few pF – a few tens of pF
```

During full-cycle calibration, all three channels reach the interference maximum (about 4/9 of the light entering the coupler). Assuming the JW3109's −7 dBm, the circulator's 0.8 dB insertion loss, and an ideal coupler, the upper limit of the received power is about 0.07 mW. At rest, the reciprocal-port detector PD2 receives the most power.

Estimate the output at this maximum received power with

$$
V_{\mathrm{TIA}}\approx I_{\mathrm{PD}}R_f,
\qquad I_{\mathrm{PD}}\approx P_{\mathrm{PD}}\mathcal{R}(\lambda)
$$

If the detector circuit saturates, lower $R_f$; if the signal is too small, raise it. Choose $C_f$ after checking stability from the photodiode capacitance, input parasitic capacitance, OPA381 GBW, and required bandwidth.

---

### ADC — ADS1115

4-channel, 16-bit, 8–860 SPS ADC with an I²C interface. Its inputs are multiplexed, so the three channels are not sampled simultaneously.

One device acquires all three detector channels. Record the PGA range, supply voltage, channel scan period, and analog bandwidth used. Manage the inter-channel time difference caused by sequential conversion with frame numbers and acquisition timestamps.

---

### MCU — Arduino Nano Every

The Nano Every's internal ADC is not used. The MCU reads the three channels from the ADS1115 and computes the phase and angular rate. Raw frames are sent to the PC over UART.

---

### Electrical Slip Ring — Adafruit 736

Six circuits, rated 2 A, up to 300 rpm. Mount it coaxially with the rotation axis and use two circuits for power and two for UART. Regulate the power from the slip ring on the turntable before use. On the stationary side, connect the UART to the PC through a USB–UART converter.

---

## 8. Pre-Purchase Checklist

| Item | Specification to unify |
|---|---|
| Wavelength | 1550 nm |
| Fiber | Single mode |
| Connectors | FC/PC throughout as a rule. If FC/UPC is chosen, use it for all optical parts |
| Optical configuration | 3×3 coupler, 3-port circulator, in-loop polarization controller, R1 optical terminator |
| 3×3 coupler | Approx. 33:33:33; check the unit's split ratio, PDL, and phase relation |
| Coil | OS2 / G.657; check the actual length, outer diameter, and minimum bend radius |
| Detectors | InGaAs |
| ADC | 3 ch or more. The ADS1115 converts sequentially |
| Light source | 1550 nm LD, continuous wave, no modulation, auto-off disabled |
| Returning light | Confirm port wiring in which all three outputs are interference outputs |
| Turntable | Holds constant speeds from −120 to +120 rpm in both directions, with a rate reference |
| Connection to rotating part | Electrical slip ring (power and UART) only |

Do not mix FC/PC and FC/UPC in the purchase specification, and never mix them with FC/APC. When connecting different polish types, check the combinations and return loss the manufacturer allows.

---

## 9. Coil Fabrication

Wind the 100 m fiber onto the bobbin. φ200 mm is the theoretical value for a single layer of constant radius; in practice, decide the winding pattern first from the cable outer diameter and the bobbin width.

Before winding, record:

- Inner diameter, outer diameter, and axial winding width of the bobbin
- Cable outer diameter, minimum bend radius, and actual fiber length
- Number of layers, turns per layer, and mean radius of each layer
- Normal direction of the coil plane and the viewing direction that defines positive rotation

Estimate the effective area of a multi-layer winding with $A_{\mathrm{eff}}=\sum_k\pi r_k^2$ from Section 3. After winding, record the inner diameter, outer diameter, and number of layers, and keep the theoretical value distinct from the measured calibration value.

Precautions:

- Do not pull the fiber hard
- Avoid sharp bends
- Do not clamp hard at a single point
- Keep the winding tension as constant as possible
- Do not go below the actual cable's minimum bend radius
- Do not force the coil end and connector leads against the coil face

---

## 10. Signal Processing

### 10.1 Three-Channel Frame Acquisition

Acquire the three returning interference outputs as one frame with the same ADC settings. Because the ADS1115 converts sequentially, tag each value with the same frame number and an acquisition timestamp, and record the inter-channel time difference. Do not use the Nano Every's `analogRead()`; read all channels from the ADS1115.

Frames are sent to the PC over UART through the slip ring, and the PC logs the reference rate together with the reception time. The calibration in Section 11 uses only steady-state segments, so this time alignment is sufficient. Choose the UART baud rate from the frame rate and the number of bytes per frame.

```cpp
// Pseudocode. Match the channel numbers to the port mapping table of the purchased part.
uint32_t frame = next_frame_number();
uint32_t t0 = timestamp_us();
float v0 = read_ads1115_single_ended(0);
uint32_t t1 = timestamp_us();
float v1 = read_ads1115_single_ended(1);
uint32_t t2 = timestamp_us();
float v2 = read_ads1115_single_ended(2);
send_frame_uart(frame, t0, t1, t2, v0, v1, v2);
```

---

### 10.2 Three-Channel Phase Recovery and Angular-Rate Conversion

Apply the offsets from Section 11.2 and the pseudo-inverse of the calibration matrix to the three channel values acquired in 10.1. The following is pseudocode showing the processing order, not executable firmware. Exclude frames from the angular-rate calculation if calibration is incomplete, or if they are clipped, failed to read, or have insufficient interference amplitude.

```cpp
// b[3] and P[2][3] are calibration values. P is the pseudo-inverse of M and includes gain correction.
float d0 = v0 - b[0];
float d1 = v1 - b[1];
float d2 = v2 - b[2];
float x = P[0][0] * d0 + P[0][1] * d1 + P[0][2] * d2;
float y = P[1][0] * d0 + P[1][1] * d1 + P[1][2] * d2;

float phi_raw = atan2(y, x);
float delta_phi_s = unwrap(phi_raw - phi_bias);
```

`x` and `y` are the two components of the corrected vector in Section 5.2. The ideal 120° formula is for initial diagnosis; do not apply a separate gain correction on top of the matrix correction.

The theoretical scale factor $K_{\phi,\mathrm{theory}}\approx0.2704$ s for 100 m, 200 mm diameter, and 1550 nm is for cross-checking only; the final calculation uses the calibrated value from Section 11.4.

```cpp
// Pseudocode. Kphi_measured is the calibration value saved in Section 11.4.
float Kphi_measured = load_calibrated_scale_factor();  // s

float omega_n = delta_phi_s / Kphi_measured;  // rad/s
float rpm     = omega_n * 60.0f / (2.0f * PI);
```

`phi_bias` is the phase bias obtained in Section 11.3, and `unwrap()` is phase unwrapping over time. In the implementation, choose the sampling period so that the phase difference before unwrapping does not exceed $\pi$ between samples.

Because `atan2()` wraps at $-\pi$ to $+\pi$, the simple measurement range of this baseline design is approximately

$$
-111\ \mathrm{rpm}
<
\mathrm{rpm}
<
+111\ \mathrm{rpm}
$$

This is the nominal range when the principal value is used centered on the bias; it does not mean the safe maximum rotation rate of the hardware. Without unwrapping, the phase repeats with a period of about 222 rpm.

---

## 11. Calibration and Functional Check

Carry out the steps in order from Section 11.1. Calibration is done on the raw frames logged on the PC; the calibration coefficients, port mapping, and ADC settings are saved as one set and written to the MCU.

### 11.1 Received Power, Polarization, and Saturation

1. With the source OFF, record the dark offset of each detector channel.
2. With the source ON and the setup at rest, adjust the polarization controller so that PD2 receives the maximum power and PD0 and PD1 become small, then leave it fixed.
3. Confirm that each TIA output is within the ADC input range. Choose $R_f$ for the maximum received power in Section 7, and confirm that no channel clips during the sweep in Section 11.2.
4. Record the source settings, received power, TIA $R_f$, ADC PGA, and sampling rate.
5. Confirm that there are no irregular fluctuations from unconnected ports.

### 11.2 3×3 Channel Calibration

Using the turntable and rate reference in Section 6.2, give all three channels a phase change of at least one full cycle. Fit the three channel values of each sample to

$$
\boldsymbol{v}=\boldsymbol{b}+M
\begin{bmatrix}\cos\phi\\\sin\phi\end{bmatrix}
$$

and obtain the offset $\boldsymbol{b}$ and the 3×2 calibration matrix $M$. If the phase is unknown, fit the ellipse traced by the three-dimensional detector values and map it to the unit circle. Fix the remaining freedom of phase origin and direction by setting the interference maximum of PD2 ($J_0$) to $\phi=0$ and taking the direction in which $\phi$ increases for a known positive rotation as positive. Do not use the dark offset alone as $\boldsymbol{b}$; include the DC component with the source ON.

The measurement criteria are evaluated as follows.

1. Fix the source settings, polarization controller, TIA gain, and ADC settings. At rest, obtain the short-term noise standard deviation $\sigma_i$ of each channel, and record the measurement time and effective bandwidth.
2. Step the turntable from −120 rpm to +120 rpm in increments of about 18 rpm, hold a constant speed at each step, and record the three raw values and the reference rate. Because the Sagnac phase is set by the angular rate, not the rotation angle, rotating at a constant speed for a long time does not sweep the phase. Confirm from two independent phase components that the changes in the three channels are not merely a common-mode fluctuation of the source intensity.
3. Estimate $\boldsymbol{b}$ and $M$ from data covering the full ellipse. Acquire at least 10 frames in each of at least 12 phase intervals, and keep a separate sweep for validation. Exclude transients during speed changes and use steady-state segments.
4. Compute the modulation amplitude of each row, $A_i=\sqrt{M_{i0}^2+M_{i1}^2}$, the singular values of the matrix, and the residuals of the validation data, and judge them with the table below.

These are provisional acceptance criteria for this PoC, not manufacturer guarantees or measured performance. Fix them together with the measurement conditions before testing, and do not relax them to fit a failing result.

| Item | Pass condition | On failure or when undeterminable |
|---|---|---|
| Three interference outputs | $A_i\geq5\sigma_i$ on all channels, no clipping | Check wiring, optical power, and the polarization controller. Do not feed an unmodulated output into demodulation |
| Phase independence | Rank of $M$ is 2, singular-value ratio $s_{\max}/s_{\min}\leq10$ | An in-phase intensity monitor or degraded interference signals alone do not pass |
| Repeatability | With coefficients fixed on a separate sweep, residual RMS of each channel $\leq0.1A_i$ | Redo calibration; isolate source fluctuation, timing skew, and drift |
| Phase range | Covers a full cycle and reproduces the same ellipse on a back-and-forth sweep | A short arc only is undeterminable; do not conclude that the three outputs are normal or abnormal |

The residual is the difference between the measured detector values and the values predicted from the fixed $\boldsymbol{b}$, $M$ and the phase recovered from the validation data. This test checks the consistency of the three-channel model; the angular-rate accuracy is checked separately in Sections 11.4 and 11.5.

In the baseline design, one full phase cycle requires an angular-rate span of about 222 rpm. A range of ±30 rpm covers only about 27% of a cycle, so if the −120 to +120 rpm sweep cannot be performed, treat the calibration as incomplete and do not declare the PoC complete with an uncalibrated angular rate.

### 11.3 Zero Point

Do not subtract the mean of the raw three-channel values as a phase offset. Compute $\phi_{\mathrm{raw}}$ with the coefficients from Section 11.2, and obtain the circular mean from short static data:

$$
\phi_{\mathrm{bias}}=\mathrm{atan2}\left(\mathrm{mean}(\sin\phi_{\mathrm{raw}}),
\mathrm{mean}(\cos\phi_{\mathrm{raw}})\right)
$$

With the phase origin of Section 11.2, $\phi_{\mathrm{bias}}\approx0$ for an ideal reciprocal port, so if it deviates significantly, check the polarization controller state and the port assignment. Also record the standard deviation or RMS at rest. The zero obtained here is a relative zero of the apparatus that includes the mounting orientation, Earth's rotation, and residual rotation of the mechanical stage.

### 11.4 Scale and Sign

Measure the actual angular rate $\Omega_n$ with the reference, and acquire the following rates several times in each direction.

```text
-30
-20
-10
0
+10
+20
+30 rpm
```

Values estimated by hand as "about" are for functional checks only and are not used for scale calibration. Convert rpm to rad/s with

$$
\Omega_n=\mathrm{rpm}\times\frac{2\pi}{60}
$$

subtract $\phi_{\mathrm{bias}}$ from Section 11.3, unwrap to obtain $\Delta\phi_{\mathrm{S}}$, and perform the linear regression

$$
\Delta\phi_{\mathrm{S}}=K_{\phi,\mathrm{meas}}\Omega_n+\delta_0
$$

The slope $K_{\phi,\mathrm{meas}}$ has units of s. Record the intercept $\delta_0$ as the residual zero point together with the zero-point repeatability; it is not used in the angular-rate conversion. Confirm that $\Delta\phi_{\mathrm{S}}$ is positive for a known positive rotation.

### 11.5 Functional Check After Calibration

Fix the coefficients from Sections 11.1–11.4 and check with separate measurements not used to derive them.

1. Confirm +10 rpm on the reference and measure for several seconds.
2. Stop the rotation and check the zero-point repeatability.
3. Confirm −10 rpm on the reference and check that the sign of the phase reverses.
4. Measure several rates between 10 and 30 rpm in both directions and check proportionality and residuals.
5. Repeat the same measurements at a different time or temperature and record the zero-point drift.

---

## 12. Main Errors and Simple Countermeasures

In the ideal formulas, optical path changes common to the clockwise and counterclockwise waves cancel. In a real device, however, temperature distribution, polarization, reflections, component imbalance, mechanical stress, and similar effects are not perfectly reciprocal and appear as phase changes unrelated to rotation. Rather than eliminating them completely, this simple device limits their effect through **short measurements, mechanical fixing, channel calibration, and zero-point calibration**.

| Source | Symptom | Countermeasure |
|---|---|---|
| Temperature change | Zero-point drift | Insulate the coil and measure after the temperature stabilizes |
| Polarization variation | Change in interference amplitude and zero point | Maximize the visibility with the polarization controller and fix it; reduce bending and pressure on the fiber |
| Connector reflections and back-reflection | Irregular interference fluctuation, reflection into the source | Clean end faces, terminate unused ports, isolate the source with the circulator |
| 3×3 coupler imbalance | Distorted phase calculation | Measure the unit's split ratio and phase, and correct with the calibration matrix |
| Detector and TIA gain mismatch | Offset and ellipse distortion | Calibrate dark offset, gain, and phase relation per channel |
| Source power fluctuation and coherent noise | Common-mode fluctuation, reduced interference contrast | Fix the continuous-wave and modulation settings; compare with another source if necessary |
| TIA and ADC saturation | Clipped waveform, phase jumps | Revisit optical power, $R_f$, $C_f$, PGA, and supply voltage |
| Sequential ADC conversion and aliasing | Inter-channel timing skew, aliasing | Record frame timestamps and limit the analog bandwidth |
| Slip-ring contact noise | Supply fluctuation, communication errors | Pass the supply through regulators and decoupling on the turntable; exclude frames with communication errors |
| Coil geometry and layers | Scale-factor deviation | Estimate the effective area and calibrate with known angular rates |
| Axis tilt | Projection error of the angular rate | Record the coil-plane normal and the positive rotation direction |
| Mechanical vibration and eccentricity | Signals other than rotation | Fix the coil to a rigid turntable and fit a protective cover |

---

## 13. Safety

1550 nm light is invisible to the naked eye.

- Never look into a fiber end face while the source is on
- During operation, fit optical terminators on unused ports; when idle, fit dust caps
- Do not clean end faces with the source ON
- Check the laser class of the product
- Do not use a source with more power than necessary
- Check the supply voltages and current limits of the TIAs, ADC, and source, and the ratings of the slip ring
- Fix all parts and cables to withstand the centripetal acceleration at the maximum speed (about 2.4 g at a radius of 0.15 m and 120 rpm), and check eccentricity and the protective cover
- If a fiber is cut, collect the glass fragments with adhesive tape or similar and do not touch them with bare hands

---

## 14. Summary

The core of this 3×3 PoC is **to gain effective area by winding a long fiber into many turns, and to recover the phase difference between light traveling in opposite directions through the same coil from three returning interference outputs**.

- Build the optics from a 3×3 coupler, a 3-port circulator, an in-loop polarization controller, and three detector channels
- Determine the three-channel offsets and calibration matrix with reference rotation from −120 to +120 rpm, and calibrate the phase bias and scale factor
- Check the sign, residuals, and zero-point repeatability with positive and negative known angular rates
- 100 m / φ200 mm / 1550 nm: theoretical value $K_{\phi,\mathrm{theory}}\approx0.2704\,\mathrm{s}$

The accuracy is limited less by the theory than by temperature, polarization, reflections, mechanical stress, coil geometry, and imbalance in the detection chain. Therefore, fix the calibrated coefficients and confirm with separate measurements at about 10–30 rpm that the optics and demodulation work correctly.

---

## 15. References

1. E. J. Post, “Sagnac Effect,” *Reviews of Modern Physics*, 39(2), 475–493, 1967. DOI: 10.1103/RevModPhys.39.475.
2. V. Vali and R. W. Shorthill, “Fiber ring interferometer,” *Applied Optics*, 15(5), 1099–1100, 1976. DOI: 10.1364/AO.15.001099.
3. S. K. Sheem, “Fiber-optic gyroscope with [3×3] directional coupler,” *Applied Physics Letters*, 37(10), 869–871, 1980. DOI: 10.1063/1.91867.
4. G. F. Trommer et al., “Passive fiber optic gyroscope,” *Applied Optics*, 29(36), 5360–5365, 1990. DOI: 10.1364/AO.29.005360.
5. Y. Jiang, P.-J. Liang, and T. Jiang, “Direct measurement of optical phase difference in a 3 × 3 fiber coupler,” *Optical Fiber Technology*, 16(3), 135–139, 2010. DOI: 10.1016/j.yofte.2010.02.005.
6. G. F. S. Nunes and J. M. S. Sakamoto, “Sliding Mode Observer with Gain Tuning Method for Passive Interferometric Fiber-Optic Gyroscope,” *Sensors*, 25(11), 3385, 2025. DOI: 10.3390/s25113385.

---

## Disclaimer

This document describes a simple setup intended for education and experiments. It is not intended as a calibrated inertial sensor or for safety-critical control applications.
