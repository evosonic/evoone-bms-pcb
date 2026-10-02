# EvoOne BMS PCB (EVO2513 v0.5.0)

KiCad project for the EvoOne battery management board, EVO2513 revision 0.5.0.

The design was converted to KiCad from the Altium sources supplied by Lattech.
The raw import needed several fixes before the schematic and PCB agreed; see
[Fixes after import](#fixes-after-import).

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

Nets that cross sheets use global labels; labels used on only one sheet are
local.

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
- `EVO2513-0_5_0-import-fps.pretty/` — project footprint library (footprints
  extracted by the Altium import), registered in `fp-lib-table`. Chip resistors
  and capacitors instead use KiCad's standard `Resistor_SMD` and
  `Capacitor_SMD` libraries
- `EVO2513-0_5_0-lattech.kicad_sym` — project symbol library (nickname
  `Lattech Systems`): the 96 Lattech symbols used by the board, extracted from
  the schematics. It is not the complete Lattech library, which was not supplied.
- `EVO2513-0_5_0-altium-import.kicad_sym` — the 4 power symbols created by the
  Altium import. Both symbol libraries are registered in `sym-lib-table`.
- `744393580068.stp` — STEP model for part 744393580068 (not currently
  referenced by any footprint in the PCB)

## Fixes after import

Each fix is a separate commit on top of the initial import.

| Problem in the raw import | Fix |
|---|---|
| Every symbol's Value field was `${ALTIUM_VALUE}`, resolved through a per-symbol field | Copied the real value into Value (209 symbols); the 26 BON pogo pads, which had no value, are `POGO-1` |
| 12 Footprint fields differed from the placed footprint only in letter case, so library lookup failed | Fields now name the exact footprint used on the PCB (J2, J5, J6, M2–M6, FL1, D9, D10, L2) |
| An imported `FOOTPRINT` field holding the bare package name (`0402`, `SMA`, …) overrode the real Footprint field, since KiCad matches field names case-insensitively | Renamed to `ALTIUM_FOOTPRINT` on all 237 symbols |
| Altium net labels became KiCad local labels, which don't connect across sheets, splitting 35 nets | Converted the 78 labels for those nets to global labels |
| 44 embedded library symbols drew Altium special strings `=Value` and `=FootPrint` as literal text | `=Value` replaced with `${VALUE}`; `=FootPrint` replaced by showing each part's own `ALTIUM_FOOTPRINT` field in the same place, so parts show their part number and package as in Altium |
| All 99 power symbols were unannotated (`#PWR?`) | Annotated `#PWR1`–`#PWR99` |
| The "Lattech Systems" symbol library wasn't available, so symbols existed only as copies embedded in the schematics, some in several slightly different versions | Exported the symbols to project libraries under their original library names, and made every schematic use one definition per symbol |
| Chip resistor and capacitor footprints had poor pads (0402s used 0.635 mm circular pads) and caused tombstoning | Replaced all 139 with KiCad standard IPC-7351 nominal footprints (`Resistor_SMD`, `Capacitor_SMD`) in place: same position, rotation and nets, zones refilled |
| PCB footprints were linked to the imported symbols by mismatched internal IDs, and nets used Altium names | Ran Update PCB from Schematic: no errors, placement and copper unchanged, all 147 nets match the schematic |

Symbols still carry their imported Altium fields (`ALTIUM_VALUE`,
`ALTIUM_FOOTPRINT`, `MANUFACTURER`, `MANUFACTURER_PN`, …) for reference.

### Known remaining issues

- The KiCad default title block is drawn over the imported Altium one, and
  variables such as `${PRODUCT_NAME}` and `${BOARD_NUMBER}` are not defined.
- The mounting spacers use two near-identical footprints: M1 uses
  `Mech Spacer Wurth 78614150960 Through M3`, and M2–M6 use the upper-case
  variant.
- The board's DRC reports many pre-existing violations (clearance, hole
  clearance, drill size, text size) from the imported design rules. With the
  standard R/C footprints, 11 courtyard overlaps now show where parts sit
  closer than IPC courtyards allow, e.g. TH1/TH2.
- `kicad-cli` only loads the first top-level sheet, so command-line ERC, BOM
  and DRC schematic-parity checks don't cover the whole design. Use the GUI.

Generated outputs (netlists, backups, per-user `.kicad_prl` settings) are not
tracked; see `.gitignore`.
