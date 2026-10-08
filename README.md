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

The schematic is hierarchical: the root sheet `EVO2513-0_5_0.kicad_sch` holds
one sheet symbol per sheet, and each sheet is its own file. Open it from
`EVO2513-0_5_0.kicad_pro`.

| Sheet | Function | Key parts |
|---|---|---|
| S01 External Connectors | Board connectors, mounting and pogo test pads | J1–J6, BON1–26 |
| S02 Input Protection | Over/under-voltage and reverse-voltage protection on the input | LTC4365, DMT4011LFG N-FETs |
| S03 BMS Input and Output Monitor | Input/output current sensing and power-path switching | LTC1960, DMP3018SFV P-FETs, 2512 shunts |
| S04 Charger Regulation | Charger switching stage | LTC1960, L1 4.7 µH |
| S05 Battery Management | Dual-battery selection and management | LTC1960 |
| S06 STEP-Up Step-Down Reg | Buck-boost output regulator | LM51772, L2 0.68 µH |
| S07 Load Switch and I-Sense | Output load switches and current-sense amplifiers | TPS22811 ×3, TLV9002 ×2 |
| S08 IO Expander | SPI I/O expander driving the rail enables and reading the faults | PCAL9714 |

The LTC1960 dual battery charger/selector is a single IC (U1) split across
sheets S03–S05.

Nets that cross sheets leave each sheet through a hierarchical label and a
pin on its sheet symbol, and the root sheet joins the pins with local
labels. There are no global labels; those nets are named from the root
(`/DCBMS`, `/POS1`, …). Labels between device sheets show the direction of
power or signal flow (e.g. DCFILTERED is an output of S02 and an input to
S03); links to the connector sheet, S01, stay passive. Labels used on only one sheet are local, and power
nets use power symbols.

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
- `EVO2513-0_5_0.kicad_sch` — root schematic sheet; `S0x *.kicad_sch` — the
  eight sheets
- `EVO2513-0_5_0.kicad_pcb` — PCB layout
- `EVO2513-0_5_0-import-fps.pretty/` — project footprint library (footprints
  extracted by the Altium import), registered in `fp-lib-table`. Chip resistors
  and capacitors instead use KiCad's standard `Resistor_SMD` and
  `Capacitor_SMD` libraries
- `EVO2513-0_5_0-lattech.kicad_sym` — project symbol library (nickname
  `Lattech Systems`): the Lattech symbols used by the board, extracted from
  the schematics, plus symbols added for the W16 changes (resistor values, PCAL9714). It is not the complete Lattech library, which was not supplied.
- `EVO2513-0_5_0-altium-import.kicad_sym` — the 4 power symbols created by the
  Altium import. Both symbol libraries are registered in `sym-lib-table`.
- `744393580068.stp` — STEP model for part 744393580068 (not currently
  referenced by any footprint in the PCB)

## Design changes (W16 review)

Part and value changes from the W16 schematic review. None changes a
connection or the board copper.

| Ref | Was | Now | Why |
|---|---|---|---|
| Q1, Q3 | DMT4011LFG 40 V | DMT6007LFGQ-7 60 V | Margin against D1's clamp voltage |
| D1 | 1.5SMB27CA | SMBJ26CA (Littelfuse) | Stand-off above the 24 V supply; 42.1 V clamp |
| L1 | 4.7 µH SRP1265A-4R7M | 10 µH SRP1265A-100M | LTC1960's 10 µH minimum |
| C10 | 100 nF 16 V | 100 nF 50 V CC0402KPX7R9BB104 | Sits on VPLUS at ~23 V |
| R54 | 1 kΩ | 768 Ω RC0402FR-07768RL | SBC load-switch current limit (R22, R53, R71 stay 1 kΩ) |
| C16 | 220 nF 50 V C0G 0805 | 220 nF 50 V X7R CC0805KFX7R9BB224 | No 220 nF C0G 0805 exists |
| C39 | 470 pF | 1 nF | SW2 snubber, boost-mode ringing |
| C3, C11, C12, C55 | 35 V | 50 V | DCFILTERED / DCBMS / DCSENSED at 26.3 V |
| C44, C45, C48–C50, C54 | 16 V | 50 V | +12 V and the switched outputs |
| R1 | 620K (description 619K) | 619K | Value and description now agree |
| D4 | Pins named as a series pair | Pins named for the fitted common-anode BAT54A | Drawing and wiring were already right |

Connection options on U5 (LM51772), as 0R 0402 links so either build can be
fitted (lower right of S06):

| Pin | Fitted | DNP (alternative) |
|---|---|---|
| MODE (7) | R86 to GND: power-save mode (W16 review) | R87 back to VCC2/ILIMCOMP: forced PWM, as built before |
| BIAS (40) | R88 to DCBMS, as built before | R89 to +12V, still under review |

U5 can also be configured over I2C instead of by its CFG resistors (right of
the MODE/BIAS links on S06, all DNP as built):

| Ref | Part | Function in the I2C build |
|---|---|---|
| R103 | 0R, CFG1 (ADDR/SLOPE, pin 9) to GND | Enables I2C at address 0x6A. R38 can stay; R103 shorts it |
| R104 | 0R, SDA/CFG3 (pin 5) to J3.34 `MCU_ENDAQ` | SDA to the MCU |
| R105 | 0R, SCL/CFG4 (pin 6) to J3.36 `MCU_ENEXT` | SCL to the MCU |
| R106, R107 | 2.2K to +3V3MCU | SDA and SCL pull-ups (pins rated to 5.8 V) |
| R30, R29 | 5.1K and 1.87K to GND | **Remove**: they are CFG3/CFG4 and would load the bus |

To build for I2C, fit R103–R107 and remove R29 and R30. J3.34 and J3.36 are
also the direct-enable fallback through R97 and R98 (S08): fit R97/R98 or the
I2C option, never both. The MCU pins on J3.34/J3.36 must support I2C.

In the I2C build U5 starts from its register defaults, not the resistor
settings, so firmware must write them before raising REGEN. They are latched
for as long as DCBMS is up, since nRST is tied to DCBMS. Defaults that differ
from the resistor build:
- slope compensation is 0.875 (R38 selects 1.0)
- VCC1, DRSS, hiccup mode and the current-limit options come from registers
  instead of R30 and R29
- the internal DAC becomes the current-limit reference (`ILIM_THRESHOLD`);
  ILIMCOMP is tied to VCC2, so confirm the average current limit stays
  disabled

Overvoltage lockout on the load switches U6, U7 and U9 (TPS22811, S07): a 1 MΩ
1% resistor (R90, R91, R92) from +12V to nEN/OVLO (pin 2), with the
existing 100K 1% pull-downs R51, R52 and R70. Each switch turns off above
13.2 V (12.60–13.81 V over the ±2% pin threshold, 1% resistors and ±0.1 µA pin
leakage) and turns back on below 12.0 V (11.50–12.60 V). To revert, leave
R90–R92 unfitted.
Undervoltage lockout (11.4 V) is deferred: EN/UVLO (pin 1) is also the MCU
enable, and a tight threshold there needs a FET.

Rail enables and faults through a PCAL9714 SPI I/O expander (U11, new sheet
S08). It shares CLOCK/MOSI/MISO with U1 (LTC1960) and has its own chip
select, nCSIO, on J3.32, the pin that was ENSBC. Address 40h (ADDR to GND);
INT has a pull-up but isn't routed off the board. Both supplies are
+3V3MCU. RESET is held low by R100 (100K) and pulled to +3V3MCU by Q23
(BSS138, the same FET as Q20-Q22) whenever +12V is up: R99 100K /
R101 1M put about 11 V on its gate. With the 12 V rail off the expander is
in reset with every port an input, and RESET can never rise above
+3V3MCU, so nothing back-powers the expander. The switch point depends on
Q23's gate threshold (roughly 3.5-4 V on +12V with a BSS138) and has no hysteresis, so
firmware should wait for +12V before configuring the expander.

Q20–Q23 are onsemi BSS138 (SOT-23) in place of the DMN65D8LFB-7
(DFN1006, 1.0 × 0.6 mm). Use the plain BSS138 or another part without a
gate protection zener: the gates sit on 100K pull-ups (R63, R65, R78) and
the R99/R101 divider, and ESD-protected parts such as the 2N7002K or
BSS138BK leak up to about 10 µA, which would pull the PG high level down
by about 1 V. BSS138: IGSS 100 nA, IDSS 0.5 µA, VGS(th) 0.8–1.5 V.

| Port | Net | Function |
|---|---|---|
| P0_0–P0_2 (out) | ENDAQ, ENSBC, ENEXT | U6, U7, U9 enables |
| P0_4–P0_6 (in) | PGDAQ, PGSBC, PGEXT | U6, U7, U9 power good (also still drive the LEDs) |
| P0_7 (in) | nFLT | U5 nFLT/nINT, which was unconnected |
| P0_3, P1_0–P1_5 | — | spare, not connected |

REGEN (U5 enable, J3.12) stays separate: it is still driven straight from
J3 and doesn't pass through the expander.

J3 pins 34 and 36 are now `MCU_ENDAQ` and `MCU_ENEXT`. Each reaches its
enable only through a DNP 0R link (R97, R98), and J3.32
`nCSIO` has R95 (fitted) to the expander CS and R96 (DNP) to ENSBC. To go
back to driving an enable straight from J3, fit its link and leave that
expander port an input. The expander powers up with every port an input
and pulls off, so the existing pull-downs keep the rails off until
firmware sets P0_0–P0_2 as outputs. The MCU firmware must change for this:
the three load-switch enables move from GPIO to SPI, and nCSIO must idle high (R93
pulls it up).

These parts aren't on the board yet: run Update PCB from Schematic, then
place and route R86–R101, Q23, U11, C58 and C59, and re-place Q20–Q22
(now SOT-23). The ENDAQ, ENSBC and ENEXT
tracks from J3 need cutting at the new links.

Still to confirm against what was fitted: C2 (100 µF on the schematic,
82 µF `35SVPF82M` ordered) and C47 (4.7 nF on the schematic, 1.8 nF ordered).

## BOM consolidation

Parts with the same value in different packages or ratings now share one BOM
line. Each change keeps or raises the voltage rating and tolerance.
All resistors are 1%: the OVLO dividers moved from 1M/102K 0.1% to
1M/100K 1%, which keeps the lowest trip point at 12.60 V.

| Refs | Was | Now |
|---|---|---|
| C8, C9, C55 | 1 µF 50 V X5R 0402 | 1 µF 50 V X5R **0603**, as C23, C44, C45, C48–C50, C54 |
| C7, C52, C53, C58, C59 | 100 nF 16 V X7R 0402 | 100 nF **50 V** X7R 0402 `CC0402KPX7R9BB104`, as C10 |
| C37, C38 | 100 nF 16 V X7R 0603 | the same, **0402** (U5 bootstrap caps, ~5 V) |
| C17, C18, C20, C21, C34 | 100 nF 50 V X7R 0402, no part number | `CC0402KPX7R9BB104` |
| C11–C14, C24–C26 | 10 µF 25 V/50 V 1206 | 10 µF 50 V X7R **1210** Murata `GRM32ER71H106KA12L`, as C27 (1210 is easier to source) |
| C27 | 10 µF 50 V X5R 1210 | the same part, X7R |
| C16 | 220 nF 50 V X7R 0805 | the same, **0603** Yageo `CC0603KRX7R9BB224`, as C22 (charger bootstrap cap) |
| C22 | 220 nF 50 V 0603, material field X5R | X7R, as its description already said |
| R43, R44 | 10R 1% 0603 | 10R 1% **0402**, as R4, R5, R24–R27 (U5 current-sense filter, no power) |

Three U5 (LM51772) fixes found during the review:

| Ref | Was | Now | Why |
|---|---|---|---|
| C30 → **R102** | 22 µF 6.3 V 0603 on VCC1 | 10K 1% 0402 to GND, as R7 | VCC1 is disabled (R30 5.1K on CFG3); the datasheet says tie a disabled VCC1 to GND through a resistor, not leave it floating |
| C32 | 22 µF 6.3 V X5R 0603 on VCC2 | 22 µF **16 V** X5R **0805** Murata `GRM21BR61C226ME44L` | VCC2 needs 6 µF minimum; the 0603 keeps only about 5–7 µF at 5 V |
| R29 | 1.9K | **1.87K** | CFG4 setting #3 accepts 1.814–1.926 kΩ (1.87K typical); 1.9K isn't a 1% value, and 1.91K can reach 1.929 kΩ |

New library symbols: `Capacitor 10uF 50V X7R 1210`, `Capacitor 22uF 16V X5R
0805` and `Resistor 1.87K 1% 0402`.
Run Update PCB from Schematic to swap the footprints. Parts that grow overlap
their neighbours' courtyards and need moving: the 0603s for C8, C9 and C55 by
up to 0.9 mm (C55 against C5 and C7, C8 against R8, C9 against R15 and C10),
the 1210s by about 0.15 mm (C11/C12/C27, C24/C26, C25 against C23), and C32
by 0.3 mm against C38 (before C38 shrinks to 0402) and 0.1 mm against TH2.

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
| KiCad design rules were defaults (0.2 mm clearance and track, 0.3 mm min hole), not the Altium rules | Set to the rules in the Altium `.PcbDoc`: 0.15 mm clearance, track and annular ring, 0.15 mm min hole, 0.3 mm hole-to-hole, and `EVO2513-0_5_0.kicad_dru` for the 0.3 mm BON pad-to-pad rule |
| FL1's ground land had no pad number, so its 38 GND vias showed as shorts | Numbered pad 4 (GND) on the board and in the project footprint library |
| The two mounting holes had no reference designator | Named H1 and H2, board-only, excluded from BOM and position files |
| 14 zero-length track segments | Removed |
| FL1's courtyard had a centre cross drawn on F.CrtYd, so KiCad saw an open (malformed) courtyard | Moved the cross to F.Fab on the board and in the project footprint library |
| Two vias sat in 0402 pads (R13 pad 2, R54 pad 2), connected on F.Cu only, and could wick solder | Removed |
| 9 tracks ran within 0.15 mm of the new R/C pad corners, B.Cu tracks passed 0.10 mm from J2 pins 9/10, and the ENEXT track 0.21 mm from J3's peg hole | Rerouted; 15 small R/C parts and 3 GND vias moved 0.1 mm |
| Net class patterns named Altium's auto-generated nets, so 13 power nets were in the Default class | Patterns point at the current net names |
| Every sheet sat 43 mil off the 50 mil connection grid in Y | Shifted each sheet's contents 7 mil onto the grid, and moved the few remaining off-grid wire corners on S06 onto it |
| 9 wires ran past their labels, leaving dangling ends | Trimmed back to the label |
| GND, +12V, +3V3MCU and +5V had no power source for ERC | Added PWR_FLAG symbols |
| Title blocks used the Altium formula `${COPY(DOCUMENTNAME,…)}` for the sheet title | Replaced with each sheet's title |
| The import made seven KiCad 10 top-level sheets, so `kicad-cli` only loaded S01, and every title block read "sheet 7 of 7" | Added a root sheet with the seven as sub-sheets (S01's content moved to `S01 External Connectors.kicad_sch`); sheet numbers now come from the page number |

Symbols still carry their imported Altium fields (`ALTIUM_VALUE`,
`ALTIUM_FOOTPRINT`, `MANUFACTURER`, `MANUFACTURER_PN`, …) for reference.

### Known remaining issues

- The KiCad default title block is drawn over the imported Altium one.
- The mounting spacers use two near-identical footprints: M1 uses
  `Mech Spacer Wurth 78614150960 Through M3`, and M2–M6 use the upper-case
  variant.
- DRC still reports: 80 hole-clearance hits, each Molex connector's own NPTH
  locating peg 0.18 mm from its pin pads (J1, J2, J4–J6; Altium had no
  hole-clearance rule); 199 silkscreen texts at 0.6 mm,
  below KiCad's 0.8 mm minimum; 16 board-edge clearance hits from the
  layer-order markers, which cross the edge on purpose; and 11 courtyard
  overlaps where parts sit closer than IPC courtyards allow, e.g. TH1/TH2.
- The design relies on via-in-pad: over 700 vias sit in pads (FET tabs, QFN
  exposed pads, FL1, the 2512 shunts, and 0402/0603 capacitors). Order the
  board with vias filled and capped; open vias would wick solder from the
  small pads.

Generated outputs (netlists, backups, per-user `.kicad_prl` settings) are not
tracked; see `.gitignore`.
