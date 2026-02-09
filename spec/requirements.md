# Requirements Document: SPDIF 1x4 Active Splitter

| Field       | Value                                      |
|-------------|--------------------------------------------|
| Document    | REQ-SPDIF-SPLITTER-001                     |
| Revision    | 0.3 (Draft)                                |
| Date        | 2025-02-22                                 |
| Author      |                                            |
| Schematic   | SPDIF_Splitter rev 1.0 (2025-02-22)       |
| Status      | Draft — pending design review              |

---

## 1. Purpose & Use Case

This device is an active 1-to-4 SPDIF coaxial audio splitter. It takes a single SPDIF RCA input, buffers and amplifies the signal, and distributes it to four independent SPDIF RCA outputs.

### 1.1 System Context

```
PC (SPDIF out) --> Digital A/B Switch --> [THIS DEVICE] --> 4x coax runs --> Receivers
                                                            (15-30m each)
```

- **Source:** PC digital audio output via SPDIF coaxial (RCA)
- **Upstream:** Signal may pass through a passive digital A/B switch before reaching this device
- **Outputs:** Four independent SPDIF coaxial outputs, each driving 15-30m (50-100ft) of RG-59 or RG-6 coaxial cable
- **Receivers:** Mixed — standalone DACs, AV receivers, and other SPDIF-capable equipment in different rooms

### 1.2 Critical Design Driver

The cable runs (15-30m) significantly exceed the IEC 60958-3 maximum specified distance of 10m. The splitter must compensate for the additional cable attenuation through active gain, while maintaining proper impedance matching and signal integrity across all paths.

---

## 2. Applicable Standards & References

| ID    | Document                          | Relevance                              |
|-------|-----------------------------------|----------------------------------------|
| STD-1 | IEC 60958-3                      | SPDIF consumer electrical spec         |
| STD-2 | USB 2.0 Specification            | Power supply (5V, 500mA max)           |
| REF-1 | TI THS7374 Datasheet (SLAS845)   | Buffer IC electrical characteristics   |
| REF-2 | TI AN-1468 / SBAA111            | Cable driving application notes        |

---

## 3. Functional Requirements

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| FR-01 | Shall accept one SPDIF coaxial input via RCA connector (J2)                | Must     |
| FR-02 | Shall provide four SPDIF coaxial outputs via RCA connectors (J3-J6)        | Must     |
| FR-03 | Shall be powered from a USB Type-A connection (J1), 5V DC                  | Must     |
| FR-04 | Shall include an LED power indicator (D1) visible during operation          | Must     |
| FR-05 | Shall support sample rates from 32 kHz through 192 kHz (all standard SPDIF PCM rates) | Must     |
| FR-06 | Shall operate as a transparent signal repeater — no decoding or re-encoding | Must     |
| FR-07 | Shall simultaneously drive all four outputs from a single input             | Must     |
| FR-08 | Shall not require user configuration or adjustment                          | Should   |
| FR-09 | Shall pass IEC 60958 consumer frames including compressed bitstreams (Dolby Digital, DTS) without alteration of channel status or user data bits | Must |
| FR-10 | Shall tolerate hot-plugging of any output connector without disrupting signal on remaining outputs | Should |

---

## 4. Electrical Requirements

### 4.1 Input Stage

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| ER-01 | Input shall be terminated with 75 ohm to ground (R6) at the RCA connector  | Must     |
| ER-02 | Input shall include a DC blocking capacitor (C7, 100nF ceramic, C0G/NP0) in series before the buffer | Must |
| ER-03 | Input shall accept SPDIF signal levels of 0.2-0.6 Vpp per IEC 60958-3     | Must     |
| ER-03a| Input shall not clip for signal levels up to 1.0 Vpp (headroom for hot sources) | Should |

**ER-03a rationale:** Some SPDIF sources (pro equipment, certain sound cards) output higher than the nominal 0.5 Vpp. With 6 dB gain, a 1.0 Vpp input produces 2.0 Vpp at the buffer output — within the THS7374's output swing capability on a 5V supply (~3.5 Vpp max). Clipping at the buffer would introduce artifacts that receivers cannot correct.

**Note:** R6 and C7 are **not present** in the current schematic (rev 1.0) and must be added.

- **R6 (75 ohm input termination):** Required to properly terminate the source's 75 ohm output impedance. Without this, reflections on the input cable will degrade the signal, especially at 192 kHz. Place as close to J2 as possible.
- **C7 (100nF input DC blocking):** Prevents DC from the upstream source (or A/B switch) from biasing the THS7374 input. Also protects against ground potential differences between source and splitter.

**Upstream device interaction:** The system context includes a passive A/B switch upstream of this splitter. Some passive switches include their own termination resistors or series impedances. R6 combined with upstream termination creates a double-termination condition that attenuates the input signal by ~6 dB (halves the voltage). The signal level budget (section 4.4) already assumes worst-case input of 0.2 Vpp; double-termination of a nominal 0.5 Vpp source yields 0.25 Vpp — still within budget. However, the combination should be verified empirically. See test TV-10.

### 4.2 Buffer Stage (THS7374IPWR)

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| ER-04 | THS7374 shall be configured for 6 dB voltage gain                          | Must     |
| ER-05 | THS7374 internal low-pass filter shall be BYPASSED on all active channels  | Must     |
| ER-06 | Back-termination shall use THS7374 internal 75 ohm resistors (preferred) OR external R1-R4, not both | Must |
| ER-06a| If using internal 75 ohm, external R1-R4 shall be DNI (Do Not Install) or 0 ohm. If using external 75 ohm, internal termination shall be disabled via control pins. | Must |
| ER-17 | All THS7374 control pins shall be strapped to fixed logic levels via pull-up/pull-down resistors — no floating pins | Must |
| ER-18 | Default power-up state shall be: LPF bypassed, gain = 6 dB, back-termination = chosen topology per OQ-01 | Must |
| ER-19 | Power-up transient on outputs shall settle within 100 ms — no sustained oscillation or rail-latch that could disrupt downstream receiver lock | Should |

**ER-17 rationale:** The THS7374 control pins (SDA, SCL) use I2C-style names but are used as static configuration inputs in this design. Floating CMOS inputs can oscillate or settle to indeterminate states, potentially enabling the LPF or changing gain. Pull resistors ensure deterministic behavior.

**ER-04 rationale:** 6 dB gain is essential to compensate for the back-termination voltage divider (6 dB loss) and cable attenuation (1-2.5 dB). See signal level budget in section 4.4.

**ER-05 rationale:** The THS7374's internal selectable low-pass filter has a 9.5 MHz cutoff. SPDIF at 192 kHz has a biphase-mark bit rate of 12.288 Mbit/s. The 3rd and 5th harmonics of the fundamental (at ~18 MHz and ~30 MHz) contribute significantly to edge definition. The 9.5 MHz filter would attenuate these harmonics and cause jitter/bit errors. The filter **must** be bypassed.

**ER-06 rationale:** The THS7374 has internal 75 ohm output resistors that can be switched in via control pins. If these are enabled, external series resistors R1-R4 must be removed (or set to 0 ohm) to avoid doubling the source impedance to 150 ohm. The choice between internal and external back-termination is captured as open question OQ-01.

### 4.3 Output Stage

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| ER-07 | Each output shall present 75 ohm source impedance                          | Must     |
| ER-08 | Each output shall include a DC blocking capacitor (C3-C6, 100nF ceramic, C0G/NP0, 50V rated) in series | Must |
| ER-08a| Alternate value 220nF C0G may be substituted for C3-C6 if 100nF is unavailable (corner frequency shifts to ~9.6 kHz — still well below signal band) | Should |
| ER-09 | Output DC offset at the RCA connector shall be < 50 mV                     | Must     |

**Note:** C3-C6 are **not present** in the current schematic (rev 1.0) and must be added.

- **C3-C6 (100nF output DC blocking, 50V, C0G/NP0):** The THS7374 adds approximately 150 mV of DC offset to the output. IEC 60958-3 requires < 50 mV DC at the receiver. AC coupling capacitors on each output are mandatory. At the lowest SPDIF frequency (32 kHz biphase = 2.048 MHz), a 100nF cap into 75 ohm gives a corner frequency of ~21 kHz — well below the signal band. The SPDIF biphase-mark waveform is inherently DC-balanced (equal positive and negative transition density), so AC coupling preserves duty cycle and edge timing integrity without baseline wander. C0G/NP0 dielectric is required to avoid the voltage-dependent capacitance shift of X7R/X5R, which would introduce asymmetric edge distortion.

### 4.4 Signal Level Budget

This is the critical calculation proving the design can drive 30m cables.

```
Stage                           Level       Notes
──────────────────────────────────────────────────────────────
SPDIF input (nominal)           0.5  Vpp    IEC 60958-3 typical
THS7374 with 6 dB gain          1.0  Vpp    2x voltage gain
Back-term divider (75 + 75)     0.5  Vpp    At cable input (matched)
30m RG-6 loss (~1-1.5 dB)       0.42-0.45 Vpp   At receiver
30m RG-59 loss (~1.5-2.5 dB)    0.35-0.42 Vpp   At receiver
IEC 60958-3 minimum             0.2  Vpp    Receiver sensitivity floor
──────────────────────────────────────────────────────────────
Result: PASS with 5-7 dB margin (RG-6) or 3-5 dB margin (RG-59)
```

**Without 6 dB gain (unity gain):**
```
Input: 0.5 Vpp -> divider: 0.25 Vpp -> 30m loss: 0.18-0.21 Vpp
Result: MARGINAL to FAIL — at or below 0.2 Vpp minimum
```

**Conclusion:** The 6 dB gain setting is **essential** for the long-cable-run use case.

**Cable recommendation:** RG-6 preferred over RG-59 for runs > 20m due to lower attenuation per meter.

### 4.5 Power Supply

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| ER-10 | Device shall operate from USB 5V (4.75-5.25V per USB spec)                 | Must     |
| ER-11 | Total current draw shall not exceed 100 mA (target: 40-60 mA)             | Must     |
| ER-12 | C1 (100nF ceramic) shall provide high-frequency bypass at THS7374 VCC pin | Must     |
| ER-13 | C2 (10uF) shall provide bulk decoupling                                    | Must     |
| ER-14a| USB power input shall include a resettable polyfuse (hold current ~200 mA, trip ~400 mA) for overcurrent protection | Should |
| ER-14b| USB power input shall include reverse polarity protection (Schottky diode or P-FET) | Should |
| ER-14c| J1 shall be a USB Type-A panel-mount receptacle with defined mechanical mounting | Must |

**ER-14a rationale:** Protects against short circuits or component failure exceeding USB 500 mA limit. A resettable polyfuse self-recovers, avoiding the need for user-replaceable fuses. Hold current set above normal draw (~50 mA) with margin for startup transients.

**ER-14b rationale:** USB Type-A connectors should always be correctly oriented, but cables, adapters, and non-standard power sources exist. A low-Vf Schottky diode (e.g., BAT54S) adds ~0.3V drop — acceptable since the THS7374 operates from 3.3-5V. A P-FET approach has near-zero drop but adds complexity.

**Power budget estimate:**
- THS7374 quiescent: ~15 mA (4 channels active)
- THS7374 output drive (4x 75 ohm loads): ~20 mA
- LED + resistor: ~5-10 mA
- **Total: ~40-50 mA** — well within USB 500 mA limit

### 4.6 ESD Protection

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| ER-15 | All RCA signal lines (J2-J6) shall have TVS ESD protection diodes (D2-D6)  | Should   |
| ER-15a| TVS diode capacitance shall be < 5 pF per device                          | Must (if TVS used) |
| ER-15b| TVS clamping voltage shall be compatible with THS7374 input/output range   | Must (if TVS used) |
| ER-16 | TVS diodes shall be placed at the connector, with the shortest possible return path to the ground plane — ESD current must not flow through signal traces or IC pins | Must (if TVS used) |
| ER-16a| TVS diodes shall clamp to the PCB signal ground plane, not to chassis/shield ground, to prevent ESD current from coupling into the signal return path | Must (if TVS used) |

**Note:** D2-D6 are **not present** in the current schematic (rev 1.0) and must be added.

**Rationale:** 15-30m coaxial cables act as effective antennas for ESD and transient events. Unlike short-cable consumer setups, these long runs have meaningful exposure. Low-capacitance TVS diodes (e.g., PESD0402-140 or similar) protect the THS7374 inputs/outputs without degrading the 75 ohm impedance match. Placement directly at the connector ensures the ESD pulse is clamped before it reaches any PCB trace or IC pin.

---

## 5. Signal Integrity Requirements

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| SI-01 | PCB traces carrying SPDIF signals shall be 75 ohm controlled impedance     | Must     |
| SI-02 | A continuous ground plane shall exist under all signal traces               | Must     |
| SI-03 | For 2-layer 1.6mm FR4 board: 75 ohm microstrip trace width ~1.1mm (43 mil) | Should   |
| SI-04 | C1 (100nF bypass) shall be placed within 5mm of THS7374 VCC pin           | Must     |
| SI-05 | Input and output traces shall avoid sharp 90-degree bends                  | Should   |
| SI-06 | Ground return path shall be unbroken between input and output connectors    | Must     |
| SI-07 | Output SPDIF traces shall maintain minimum 2x trace-width spacing between adjacent channels, or include ground pour guard between channels | Should |
| SI-08 | RCA connector shells (shields) shall connect to PCB ground plane at each connector. An optional ground-lift network footprint (0 ohm link in parallel with series RC: 10 ohm + 10nF to ground) shall be provided between chassis/shield ground and signal ground, populatable if ground loop issues arise during testing | Should |

---

## 6. Mechanical Requirements

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| MR-01 | PCB size shall be approximately 60mm x 40mm (±10mm)                        | Target   |
| MR-02 | Input (J2) and USB power (J1) shall be on one board edge                   | Should   |
| MR-03 | Outputs (J3-J6) shall be on the opposite board edge from input             | Should   |
| MR-04 | Board shall include M3 mounting holes at corners                           | Should   |
| MR-05 | Operating temperature range: 0-50 C (indoor residential use)               | Must     |
| MR-06 | Board shall be standard 1.6mm FR4 unless SI requirements demand otherwise  | Should   |

---

## 7. BOM Summary

### Existing Components (in schematic rev 1.0)

| Ref   | Value         | Description                        | Package       |
|-------|---------------|------------------------------------|---------------|
| U?    | THS7374IPWR   | 4-channel video/audio buffer       | TSSOP-14      |
| J1    | USB Type-A    | Panel mount, power input           | Through-hole  |
| J2    | RCA           | SPDIF input connector              | Through-hole  |
| J3-J6 | RCA           | SPDIF output connectors            | Through-hole  |
| R1-R4 | 75 ohm        | Output impedance matching          | 0402/0603     |
| R5    | TBD           | LED current limiting               | 0402/0603     |
| D1    | LED           | Power indicator                    | 0603/0805     |
| C1    | 100nF         | HF bypass decoupling               | 0402/0603     |
| C2    | 10uF          | Bulk decoupling                    | 0805          |

### New Components Required

| Ref   | Value         | Description                        | Package       | Requirement |
|-------|---------------|------------------------------------|---------------|-------------|
| R6    | 75 ohm        | Input termination                  | 0402/0603     | ER-01       |
| C3-C6 | 100nF C0G 50V| Output DC blocking (x4)            | 0402/0603     | ER-08       |
| C7    | 100nF C0G 50V| Input DC blocking                  | 0402/0603     | ER-02       |
| D2-D6 | TVS           | ESD protection, < 5pF (x5)        | SOD-882/0402  | ER-15       |
| F1    | Polyfuse      | Resettable fuse, 200mA hold        | 1206          | ER-14a      |
| D7    | Schottky      | Reverse polarity protection        | SOD-323       | ER-14b      |
| R7-R9 | Pull-up/down  | THS7374 control pin strapping (x3) | 0402/0603     | ER-17       |

**Note:** U? must be annotated to U1 in the schematic.

---

## 8. Testing & Verification

| ID    | Test                                    | Pass Criteria                                          |
|-------|-----------------------------------------|--------------------------------------------------------|
| TV-01 | Power-on                                | LED illuminates, 5V rail stable, quiescent current < 100 mA |
| TV-02 | Signal passthrough — 44.1 kHz           | All 4 outputs pass audio, no dropouts over 10-minute continuous play (1m cable) |
| TV-03 | Signal passthrough — 192 kHz            | All 4 outputs pass audio, no dropouts over 10-minute continuous play (1m cable) |
| TV-04 | Long cable — 30m RG-6 on all outputs    | All 4 outputs pass 44.1 kHz and 192 kHz audio, no dropouts over 10-minute continuous play |
| TV-05 | Long cable — 30m RG-59 on all outputs   | All 4 outputs pass 44.1 kHz and 192 kHz audio, no dropouts over 10-minute continuous play |
| TV-06 | Output level measurement                | 0.4-0.6 Vpp at cable input (before cable)              |
| TV-07 | Output DC offset                        | < 50 mV DC at each RCA output                          |
| TV-08 | Output impedance                        | 75 ohm ± 10% per channel (TDR or return loss)          |
| TV-09 | Simultaneous operation                  | All 4 outputs driven concurrently, no crosstalk artifacts |
| TV-10 | Upstream A/B switch compatibility       | Signal passes through upstream passive A/B switch + splitter to all 4 outputs; verify no dropouts and output level > 0.3 Vpp (accounts for possible double-termination) |
| TV-11 | Signal quality — edge integrity         | Oscilloscope at output connector: rise/fall time < 20 ns, overshoot/ringing < 20% of amplitude, clean eye opening on scope persistence mode |
| TV-12 | Receiver lock stability                 | Each output connected to a different receiver type (DAC, AVR); all maintain lock continuously over 1-hour play with no unlock/relock events |
| TV-13 | Hot-plug stress                         | While signal is active on all outputs, disconnect and reconnect each output cable 10 times; remaining outputs shall maintain uninterrupted signal |
| TV-14 | Compressed bitstream passthrough        | Dolby Digital and DTS bitstreams pass through and decode correctly on all 4 outputs |
| TV-15 | Multi-room ground differential          | Two outputs connected to receivers on different electrical branch circuits; verify no audible artifacts, no receiver unlock |
| TV-16 | Power-up behavior                       | Scope on outputs during power-up: no sustained oscillation or rail-latch; outputs settle to idle state within 100 ms |
| TV-17 | Hot source headroom                     | Apply 1.0 Vpp input signal; verify output waveform is not clipped (scope check) |

---

## 9. Open Design Questions

| ID    | Question                                                                        | Recommendation                        | Status |
|-------|---------------------------------------------------------------------------------|---------------------------------------|--------|
| OQ-01 | **Back-termination topology:** Use THS7374 internal 75 ohm resistors (via control pins) or external R1-R4? | Use internal; remove R1-R4. Simpler layout, fewer components, matched resistors. | Open |
| OQ-02 | **Input DC blocking (C7):** Is it needed given the upstream source?              | Add C7. Defensive design — unknown upstream DC conditions, negligible cost. | Open |
| OQ-03 | **THS7374 control pin configuration:** What is the exact pin matrix for: LPF bypass, 6 dB gain, internal 75 ohm enable? | Verify against SLAS845 datasheet Table 1. Pins: SDA (pin 1), SCL (pin 2) — likely static logic levels, not I2C. **Partially addressed:** ER-17/ER-18 now require fixed pull resistors and defined power-up state. Exact pin-to-logic-level mapping still needs datasheet verification. | Open |
| OQ-04 | **PCB layer count:** 2-layer or 4-layer?                                        | Start with 2-layer. Adequate for this frequency range if trace widths respect 75 ohm microstrip. Move to 4-layer only if routing becomes constrained or return loss is poor. | Open |
| OQ-05 | **Enclosure grounding:** How to connect chassis/shield ground to circuit ground? | Direct connection — simplest approach, standard for SPDIF consumer equipment. Ground-lift network footprint added per SI-08 as fallback. Revisit only if ground loop issues arise in testing (TV-15). | Open |

**Decision recording:** When closing an open question, record the final decision, date, and resulting BOM/schematic changes in the revision history. This ensures traceability from question to implementation.

---

## 10. Risk Register

| ID    | Risk                                                  | Likelihood | Impact | Mitigation                                                |
|-------|-------------------------------------------------------|------------|--------|-----------------------------------------------------------|
| RK-01 | 192 kHz signal fails on 30m cable                     | Medium     | High   | 6 dB gain compensates loss; RG-6 recommended over RG-59   |
| RK-02 | THS7374 DC offset exceeds SPDIF spec at receiver      | High       | Medium | Output DC blocking caps C3-C6 (ER-08)                     |
| RK-03 | ESD damage from long cable runs                       | Medium     | High   | TVS diodes D2-D6 on all signal lines (ER-15), placed at connectors (ER-16) |
| RK-04 | Impedance mismatch causes reflections/jitter          | Medium     | High   | 75 ohm controlled impedance traces (SI-01), proper termination (ER-01, ER-07) |
| RK-05 | LPF not properly bypassed — attenuates high-rate SPDIF | Low        | High   | Control pin pull resistors (ER-17/ER-18), verify config (OQ-03), test at 192 kHz (TV-03) |
| RK-06 | Crosstalk between output channels on PCB              | Low        | Medium | Ground plane (SI-02), trace spacing (SI-07), guard pours   |
| RK-07 | Ground loops between rooms cause receiver unlock       | Medium     | Medium | Ground-lift network footprint (SI-08), test TV-15           |
| RK-08 | Double-termination with upstream A/B switch            | Low        | Low    | Signal budget has margin; verified by TV-10                 |
| RK-09 | USB power fault (short circuit, reverse polarity)      | Low        | High   | Polyfuse F1 (ER-14a), reverse protection D7 (ER-14b)       |

---

## 11. Non-Functional Requirements

| ID    | Requirement                                                                 | Priority |
|-------|-----------------------------------------------------------------------------|----------|
| NF-01 | Device shall support continuous 24/7 operation without thermal or reliability issues | Must |
| NF-02 | Device shall not require power cycling to recover from any normal operating condition (signal loss, source change, hot-plug) | Must |
| NF-03 | Conducted and radiated EMI shall not cause interference with nearby consumer equipment | Should |
| NF-04 | Device shall tolerate input signal loss and re-acquisition without manual intervention | Must |

**NF-01 rationale:** The splitter distributes audio to multiple rooms in a residential installation. It will be powered on continuously as part of a home entertainment stack. Thermal design must ensure component junction temperatures remain within rated limits at 50 C ambient (MR-05) under sustained operation.

**NF-03 rationale:** While formal EMI certification (FCC Part 15) is not required for a personal-use device, 30m coaxial runs can act as antennas for both emission and susceptibility. Good layout practices (ground plane, short traces, bypassing) address this implicitly.

---

## 12. Schematic Changes Implied by Requirements

The following changes to the current schematic (rev 1.0) are required to meet the requirements in this document:

1. **Add R6 (75 ohm)** — Input termination resistor, J2 signal to GND (ER-01)
2. **Add C7 (100nF C0G)** — Input DC blocking cap, in series between J2 and U1 input (ER-02)
3. **Add C3-C6 (100nF C0G, 50V x4)** — Output DC blocking caps, in series between U1 outputs and J3-J6 (ER-08)
4. **Add D2-D6 (TVS x5)** — ESD protection diodes on all RCA signal lines, placed at connectors with short ground returns (ER-15, ER-16)
5. **Resolve back-termination topology** — Configure THS7374 internal 75 ohm or keep external R1-R4; if internal, mark R1-R4 as DNI (OQ-01, ER-06, ER-06a)
6. **Add THS7374 control pin pull resistors** — Strap all control pins to fixed logic levels for: LPF bypass, 6 dB gain, 75 ohm termination (ER-17, ER-18)
7. **Annotate U1** — Change reference from "U?" to "U1" (housekeeping)
8. **Add USB polyfuse** — Resettable fuse on 5V rail (ER-14a)
9. **Add reverse polarity protection** — Schottky diode or P-FET on USB power input (ER-14b)
10. **Add ground-lift network footprint** — Optional 0 ohm / RC network between shield and signal ground at RCA connectors (SI-08)

---

## Revision History

| Rev | Date       | Author | Changes           |
|-----|------------|--------|-------------------|
| 0.1 | 2025-02-22 |        | Initial draft     |
| 0.2 | 2025-02-22 |        | Added Appendix A: IC Selection Rationale |
| 0.3 | 2025-02-22 |        | Added: signal quality test metrics (TV-10 through TV-17), non-functional requirements (NF-01 through NF-04), USB power protection (ER-14a/b, F1, D7), control pin requirements (ER-17/ER-18/ER-19), ESD placement topology (ER-16/ER-16a), ground-lift network (SI-08), crosstalk spacing (SI-07), compressed bitstream passthrough (FR-09), hot-plug tolerance (FR-10), input clipping headroom (ER-03a), coupling cap voltage rating and alternate value (ER-08/ER-08a), output impedance topology clarity (ER-06a), upstream A/B switch interaction note, decision tracking process, expanded risk register (RK-07 through RK-09) |

---

## Appendix A: IC Selection Rationale

### A.1 Summary

The THS7374IPWR was selected as the active buffer IC because it is the only single-IC solution that provides all four channels with built-in 6 dB gain and internal 75 ohm back-termination, operating from a single 5V supply. No other evaluated IC matches this feature set in one package.

### A.2 Design Requirements Driving IC Selection

Any candidate IC must satisfy:

1. **4 output channels** from a single input (or minimum IC count)
2. **~6 dB voltage gain** — essential per the signal level budget (section 4.4); without it, 30m cable runs are marginal/failing at 0.18-0.21 Vpp vs 0.2 Vpp minimum
3. **75 ohm output impedance** — internal or with external series resistors
4. **Single 5V supply** — USB powered, no dual rails available
5. **Bandwidth > ~30 MHz** — SPDIF at 192 kHz = 12.288 Mbit/s biphase; need 3rd/5th harmonics for clean edges
6. **Low power** — target < 100 mA total device draw
7. **Small footprint** — fits on ~60x40mm board

### A.3 Candidates Evaluated

**THS7374IPWR (TI, current selection):** 4-channel video/audio buffer. 6 dB fixed gain, internal switchable 75 ohm + LPF. 9.5 MHz LPF per channel (bypassed; raw BW ~400 MHz). Single 3.3-5V supply, ~10 mA quiescent. TSSOP-14. ~$2-4.

**THS7373 (TI):** 4-channel, asymmetric (1x SD + 3x HD). Same 6 dB gain and internal 75 ohm. 16.2 mA quiescent. TSSOP-14. Since the LPF is bypassed on all channels, the asymmetry is irrelevant — functionally equivalent to THS7374 but draws more power. Viable as a second-source/backup.

**THS7316 (TI):** 3 channels only — would need 2 ICs for 4 outputs. SOIC-8. Disqualified due to extra IC and board space.

**74AHCT125 / 74HC125 (Quad Logic Buffer):** 4 channels, very cheap (~$0.30-1.00). **No gain** — unity buffer only. Output impedance ~20-30 ohm (not 75 ohm). Disqualified: no gain means 30m cable runs receive 0.18-0.21 Vpp, at or below the 0.2 Vpp IEC 60958-3 minimum. Would only work for short cables.

**74HCU04 (Hex Inverter with feedback):** Can achieve ~1.5-2x gain with external feedback resistor network. Popular in DIY SPDIF circuits. Very cheap. Concerns: imprecise gain, signal inversion (pairs needed), feedback topology adds jitter, no 75 ohm support. Marginal for long cable runs.

**LMH6703 / ADA4857 (High-speed op-amps):** Single-channel ICs — need 4 for this application. Configurable gain, 600+ MHz bandwidth. Prefer dual ±5V supply. 4x IC cost ($16-32), complex layout, overkill bandwidth. Disqualified on practicality.

**MAX4396 (Maxim, quad op-amp):** 4 channels, 85 MHz, configurable gain. Requires ±2.25V to ±5.5V dual supply. Disqualified: incompatible with single USB 5V power.

**Dedicated SPDIF ICs (CS8416, DIR9001, WM8805):** All are receivers/decoders that demodulate SPDIF to I2S. Not applicable — this design needs a transparent analog buffer, not a decoder/encoder.

### A.4 Comparison Matrix

| Criterion              | THS7374       | THS7373       | 74AHCT125  | 74HCU04+FB | LMH6703 x4 |
|------------------------|---------------|---------------|------------|------------|-------------|
| Channels (1 IC)        | 4             | 4             | 4          | 6 (inv.)   | 1           |
| 6 dB gain              | Yes (built-in)| Yes (built-in)| **No**     | ~Approx    | Yes (ext R) |
| 75 ohm support         | Internal      | Internal      | No         | No         | Yes (ext R) |
| Single 5V supply       | Yes           | Yes           | Yes        | Yes        | Marginal    |
| BW (LPF bypassed)      | ~400 MHz      | ~400 MHz      | 20-50 MHz  | ~50 MHz    | 600 MHz     |
| Quiescent current      | ~10 mA        | ~16 mA        | ~5 mA      | ~3 mA      | ~20 mA      |
| IC count               | 1             | 1             | 1          | 1          | 4           |
| Package                | TSSOP-14      | TSSOP-14      | SOIC-14    | SOIC-14    | 4x SOIC-8   |
| External parts needed  | DC block caps | DC block caps | Gain + R   | R network  | R network   |
| Unit cost              | ~$2-4         | ~$2-5         | ~$0.30     | ~$0.20     | ~$16-32     |
| Long cable viability   | **Yes**       | **Yes**       | **No**     | Marginal   | **Yes**     |

### A.5 Conclusion

The THS7374IPWR remains the correct choice for this design:

1. **Only single-IC solution** providing 4 channels, 6 dB gain, and internal 75 ohm back-termination at 5V single supply.
2. **The "video buffer" concern is a non-issue.** With the 9.5 MHz LPF bypassed, raw amplifier bandwidth is ~400 MHz — more than sufficient for SPDIF at 12.288 Mbit/s.
3. **DC offset weakness is already addressed** by output DC blocking caps C3-C6 (ER-08).
4. **THS7373 is a viable second-source** if THS7374 becomes unavailable. Functionally equivalent with LPF bypassed, but draws slightly more power (16 mA vs 10 mA).
5. **No dedicated SPDIF buffer/distribution IC exists.** All SPDIF-specific ICs are decoders/encoders, not transparent analog buffers.
