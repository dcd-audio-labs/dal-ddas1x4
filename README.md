# DAL-DDAS1x4

**Digital Distribution Active Splitter — 1-in, 4-out SPDIF over coax**

A KiCad 9.0 hardware project for an active SPDIF coaxial splitter. It takes a single SPDIF RCA input, buffers and amplifies the signal, and drives four independent SPDIF RCA outputs — each capable of reaching receivers at the end of 15–30 meter cable runs.

## Why this exists

Off-the-shelf passive SPDIF splitters work fine for short cables, but they fall apart over long runs. The IEC 60958-3 spec only guarantees 10 meters. This board uses a TI THS7374 quad buffer with 6 dB of gain to compensate for the voltage divider loss of proper 75 ohm back-termination and the attenuation of long coax cables, keeping signal levels well above the 0.2 Vpp receiver sensitivity floor even at 30 meters on RG-6.

Typical use case: a PC or media server in one room distributing SPDIF digital audio to DACs and AV receivers in multiple rooms throughout a house.

## How it works

```
                                ┌──── 75Ω ──── RCA Out 1 ──── 15-30m coax ──── DAC
SPDIF In ──── 75Ω term ──── THS7374 (6dB gain) ──── 75Ω ──── RCA Out 2 ──── 15-30m coax ──── AVR
   (RCA)                        ├──── 75Ω ──── RCA Out 3 ──── 15-30m coax ──── DAC
                                └──── 75Ω ──── RCA Out 4 ──── 15-30m coax ──── AVR
                   USB 5V power ──┘
```

- **Input:** 1x RCA SPDIF (0.2–0.6 Vpp per spec, tolerates up to 1.0 Vpp)
- **Outputs:** 4x RCA SPDIF, each with 75 ohm impedance-matched buffered drive
- **Power:** USB Type-A, 5V — draws roughly 40–50 mA
- **Transparent:** No decoding or re-encoding. PCM at any standard sample rate (32–192 kHz), Dolby Digital, and DTS bitstreams all pass through unmodified.
- **Buffer IC:** TI THS7374IPWR — four-channel video/audio buffer with 6 dB fixed gain, internal switchable 75 ohm back-termination, ~400 MHz bandwidth (LPF bypassed), single 3.3–5V supply, TSSOP-14

## Signal level budget

The 6 dB gain is essential. Without it, 30m cable runs land right at the edge of receiver sensitivity:

| Stage | Level |
|---|---|
| SPDIF input (nominal) | 0.5 Vpp |
| After THS7374 6 dB gain | 1.0 Vpp |
| After back-termination divider (75+75 ohm) | 0.5 Vpp |
| After 30m RG-6 (~1–1.5 dB loss) | 0.42–0.45 Vpp |
| IEC 60958-3 receiver minimum | 0.2 Vpp |
| **Margin** | **5–7 dB** |

## Project structure

```
SPDIF_Splitter/     KiCad 9.0 project — schematic, PCB layout, project config
spec/               Engineering specifications (requirements, design rationale)
docs/               Hugo site (hugo-book theme) for published documentation
datasheets/         Component datasheets
```

## Status

Schematic rev 1.0 — initial design complete, pending design review. Several additions identified in the requirements document (`spec/requirements.md`) are not yet in the schematic, including input termination, DC blocking caps, ESD protection, and control pin strapping.

## License

See repository for license details.
