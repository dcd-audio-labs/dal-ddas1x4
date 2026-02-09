---
title: "Design"
weight: 1
bookCollapseSection: true
---

# Design

This section covers the circuit architecture and design decisions for the SPDIF 1x4 Active Splitter.

## Architecture Overview

The design uses a single TI THS7374IPWR 4-channel video/audio buffer amplifier to split and amplify one SPDIF input to four outputs:

```
                    ┌─────────────┐
  J2 (RCA In) ──>──┤  THS7374    ├──>── J3 (RCA Out 1)
                    │  4-channel  ├──>── J4 (RCA Out 2)
  J1 (USB 5V) ──>──┤  buffer     ├──>── J5 (RCA Out 3)
                    │  6 dB gain  ├──>── J6 (RCA Out 4)
                    └─────────────┘
```

## Design Principles

1. **Active gain is essential** — 6 dB gain compensates for the back-termination voltage divider and cable attenuation over 15-30m runs
2. **Transparent analog buffer** — no digital decoding/encoding; the SPDIF bitstream passes through unmodified
3. **75 ohm impedance matching** — input termination and output back-termination maintain signal integrity
4. **Single 5V supply** — USB powered, no external power supply needed

## Pages

Content pages covering specific design topics will be added here as the design progresses.
