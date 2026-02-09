---
title: "SPDIF 1x4 Active Splitter"
type: docs
---

# SPDIF 1x4 Active Splitter

A USB-powered active 1-to-4 SPDIF coaxial audio splitter with buffered outputs for long cable runs.

---

## About This Project

This is a KiCad 9.0 PCB design for a device that takes a single SPDIF digital audio input and distributes it to four buffered outputs. It is designed specifically for driving 15-30m coaxial cable runs — well beyond the 10m IEC 60958-3 specification limit.

## Key Features

| Feature | Detail |
|---------|--------|
| Input | 1x SPDIF coaxial (RCA) |
| Outputs | 4x SPDIF coaxial (RCA), independently buffered |
| Buffer IC | TI THS7374IPWR (4-channel, 6 dB gain, internal 75 ohm) |
| Power | USB Type-A, 5V, ~50 mA draw |
| Sample Rates | 32 kHz - 192 kHz PCM, plus compressed bitstreams |
| Cable Support | Up to 30m RG-6 or RG-59 per output |

---

## System Context

```
PC (SPDIF out) --> Digital A/B Switch --> [THIS DEVICE] --> 4x coax runs --> Receivers
                                                            (15-30m each)
```

The splitter compensates for cable attenuation through 6 dB active gain, maintaining signal levels above the IEC 60958-3 minimum (0.2 Vpp) even at 30m with RG-59 cable.

---

## Documentation Sections

- **[Design]({{< relref "/docs/design" >}})** — Circuit architecture, signal path, and design decisions
- **[Specifications]({{< relref "/docs/specifications" >}})** — Electrical specs, signal budget, and BOM
- **[Reference]({{< relref "/docs/reference" >}})** — Quick reference tables, pinouts, and datasheets
