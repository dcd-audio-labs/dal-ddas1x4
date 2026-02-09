# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **hardware electronics project** — a KiCad 9.0 PCB design for an active 1-to-4 SPDIF RCA audio splitter. It takes a single SPDIF digital audio input and distributes it to 4 buffered outputs with proper impedance matching. USB powered.

## Architecture

- **Main IC:** THS7374IPWR (TI video/audio buffer amplifier, TSSOP-14) provides active buffering for all 4 output channels
- **Input:** Single RCA SPDIF jack (J2)
- **Outputs:** 4x RCA SPDIF jacks (J3-J6), each with 75Ω impedance matching resistors (R1-R4)
- **Power:** USB Type-A panel mount (J1) providing 5V, with LED power indicator (D1) and current limiting resistor (R5)
- **Decoupling:** C1 (0.1μF high-frequency bypass) and C2 (10μF bulk filtering)

## Directory Structure

- `SPDIF_Splitter/` — Main KiCad project (schematic, PCB layout, project config)
- `docs/` — Hugo static site (hugo-book theme) for published documentation, deployed to GitHub Pages
- `spec/` — Engineering specification documents (requirements.md, etc.) — source-of-truth markdown files
- `enclosure/` — Physical enclosure design (placeholder)
- `fabrication/` — Gerber/manufacturing output files (placeholder)
- `footprints/` — Custom KiCad footprints
- `symbols/` — Custom KiCad schematic symbols

## Documentation

- **`spec/`** contains the canonical engineering specs (requirements, design rationale, etc.) as plain markdown
- **`docs/`** is a Hugo site using the [hugo-book](https://github.com/alex-shpak/hugo-book) theme — run `hugo --source docs server` to preview locally
- Hugo content lives in `docs/content/docs/` organized by section (design, specifications, reference)
- The hugo-book theme is a git submodule in `docs/themes/hugo-book`
- GitHub Actions workflow (`.github/workflows/hugo.yml`) builds and deploys to GitHub Pages on push to main

## Key Files

- `SPDIF_Splitter/SPDIF_Splitter.kicad_sch` — Main schematic
- `SPDIF_Splitter/SPDIF_Splitter.kicad_pcb` — PCB layout
- `SPDIF_Splitter/SPDIF_Splitter.kicad_pro` — Project config (includes ERC settings, design rules, BOM format)

## Working with KiCad Files

KiCad files are S-expression text format. Schematics and PCB layouts can be read/parsed as text, but visual editing requires KiCad GUI. The project targets KiCad 9.0 (schematic format version 20250114).

## Design Constraints

- SPDIF requires 75Ω impedance matching on all output paths
- USB 5V power only — no external power supply
- Schematic revision 1.0 (dated 2025-02-22), paper size A4
