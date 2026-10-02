# EvoOne BMS PCB (EVO2513 v0.5.0)

KiCad project for the EvoOne battery management board, EVO2513 revision 0.5.0.

The design was converted to KiCad from the Altium sources supplied by Lattech.
Some symbol fields still carry Altium-era values (e.g. `${ALTIUM_VALUE}`) left
over from the import.

## Board

| | |
|---|---|
| Outline | 110 × 69 mm |
| Layers | 8 copper (F.Cu, In1–In6.Cu, B.Cu) |
| Thickness | 1.584 mm |
| Components | ~230 footprints |
| Tool | KiCad 10.0 |

## Schematic sheets

The schematic uses KiCad 10 top-level sheets (listed in the project file)
rather than a hierarchy, so open it from `EVO2513-0_5_0.kicad_pro`.

| Sheet | Function | Key parts |
|---|---|---|
| S01 External Connectors (`EVO2513-0_5_0.kicad_sch`) | Board connectors, mounting and pogo test pads | J1–J6, BON1–26 |
| S02 Input Protection | Over/under-voltage and reverse-voltage protection on the input | LTC4365, DMT4011LFG N-FETs |
| S03 BMS Input and Output Monitor | Input/output current sensing and power-path switching | LTC1960, DMP3018SFV P-FETs, 2512 shunts |
| S04 Charger Regulation | Charger switching stage | LTC1960, L1 4.7 µH |
| S05 Battery Management | Dual-battery selection and management | LTC1960 |
| S06 STEP-Up Step-Down Reg | Buck-boost output regulator | LM51772, L2 0.68 µH |
| S07 Load Switch and I-Sense | Output load switches and current-sense amplifiers | TPS22811 ×3, TLV9002 ×2 |

The LTC1960 dual battery charger/selector is a single IC (U1) split across
sheets S03–S05.

## Connectors

| Ref | Part |
|---|---|
| J1 | Molex 1053101204, 2.5 mm pitch |
| J2 | Molex 1053101114, 14-way 2.5 mm pitch, vertical |
| J3 | Amphenol BergStak 10144517-043802LF, 40-pin 0.8 mm receptacle |
| J4 | Molex 1053102204, 2.5 mm pitch |
| J5, J6 | Molex 1053091102, 2-way 2.5 mm pitch |

## Files

- `EVO2513-0_5_0.kicad_pro` — project
- `EVO2513-0_5_0.kicad_sch` and `S0x *.kicad_sch` — schematics
- `EVO2513-0_5_0.kicad_pcb` — PCB layout
- `EVO2513-0_5_0-import-fps.pretty/` — project footprint library (all
  footprints used by the board, extracted by the Altium import), registered in
  `fp-lib-table`
- `744393580068.stp` — STEP model for part 744393580068 (not currently
  referenced by any footprint in the PCB)

Generated outputs (netlists, backups, per-user `.kicad_prl` settings) are not
tracked; see `.gitignore`.
