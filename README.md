# EvoOne BMS PCB (EVO2513 v0.6.0)

KiCad project for the EvoOne battery management board, EVO2513 revision 0.6.0.
Revision 0.6.0 is the 0.5.0 design (the boards in field test) with the W16
review changes below. The project, schematic and PCB files are named
`EVO2513`; the project libraries keep their `EVO2513-0_5_0` names.

The design was converted to KiCad from the Altium sources supplied by Lattech.
The raw import needed several fixes before the schematic and PCB agreed; see
[Fixes after import](#fixes-after-import).

## Board

| | |
|---|---|
| Outline | 110 × 69 mm |
| Layers | 8 copper (F.Cu, In1–In6.Cu, B.Cu) |
| Thickness | 1.598 mm finished (1.59 mm laminate + ENIG); stackup as the Altium fab drawing: three 0.25 mm cores, 2116 prepregs, 1 oz copper |
| Components | ~230 footprints |
| Tool | KiCad 10.0 |

## Schematic sheets

The schematic is hierarchical: the root sheet `EVO2513.kicad_sch` holds
one sheet symbol per sheet, and each sheet is its own file. Open it from
`EVO2513.kicad_pro`.

| Sheet | Function | Key parts |
|---|---|---|
| S01 External Connectors | Board connectors, mounting and test points | J1–J6, TP1–TP26 |
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

The thermistor nets on J3 pins 24 and 28 are deliberately swapped from the
Altium design. The swap went in with `6ce5d68`, whose commit message doesn't
mention it. Firmware written for the earlier pinout must swap its BMSTEMP and
REGTEMP channels.

| J3 pin | Altium design | Now |
|---|---|---|
| 24 | BMSTEMP (TH1) | REGTEMP (TH2) |
| 28 | REGTEMP (TH2) | BMSTEMP (TH1) |

## Files

- `EVO2513.kicad_pro` — project
- `EVO2513.kicad_sch` — root schematic sheet; `S0x *.kicad_sch` — the
  eight sheets
- `EVO2513.kicad_pcb` — PCB layout
- `FIRMWARE_NOTES.md` — what the MCU firmware must know about this revision:
  SPI and expander set-up, the inverted DAQ enable, load-switch thresholds and
  current-monitor scaling
- `EVO2513-0_5_0-import-fps.pretty/` — project footprint library (footprints
  extracted by the Altium import), registered in `fp-lib-table`. Chip resistors
  and capacitors instead use KiCad's standard `Resistor_SMD` and
  `Capacitor_SMD` libraries
- `EVO2513-0_5_0-lattech.kicad_sym` — project symbol library (nickname
  `Lattech Systems`): the Lattech symbols used by the board, extracted from
  the schematics, plus the PCAL9714 added for the W16 changes. It is not the complete Lattech library, which was not supplied.
  Resistors and capacitors use KiCad's standard `Device:R`, `Device:C` and
  `Device:C_Polarized` symbols, with value, footprint and part number in the
  symbol fields, so a value change needs no new symbol
- `EVO2513-0_5_0-altium-import.kicad_sym` — the 4 power symbols created by the
  Altium import. Both symbol libraries are registered in `sym-lib-table`.

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
| C2 | 100 µF (part number `35SVPF82M`) | 82 µF `35SVPF82M` | Matches the ordered part |
| C46, C47, C51 | 4.7 nF 50 V X7R 0402 | 1.8 nF 50 V X7R 0402 `CC0402KRX7R9BB182` | TPS22811 DVDT caps (U6, U7, U9): shorter output ramp, as ordered for C47; the loads have little capacitance |
| R24–R27 | 10 Ω | 0 Ω | Pack comms pass straight through; protection moves to the MCU board |
| D9, D10 | DT1240V3-04LP fitted | DNP | Go with R24–R27: wrong clamp level for 3.3 V, and the protected MCU is on another board |
| R14 | 0.01 Ω | 0.012 Ω Yageo `PA2512FKF7W0R012E` | Adapter limit 8.33 A, below the 9.2 A supply, so charging throttles before the supply trips |
| R36 | 61.9K fitted | DNP | VIN-FB goes to ground through R37, as required when PCM is unused |
| R59, R61, R76 / R60, R62, R77 | 1M / 130K | 100K / 13K | TPS22811 PGTH dividers: same 10.43 V threshold, 10× lower impedance so pin leakage can't push it near the rail |
| R90–R92, R101 | 1M | 1.07M | OVLO trip 14.0 V, so a tripped switch recovers on the 12.03 V rail; R101 shares the BOM line |
| U6 EN/UVLO | ENDAQ from P0_0, R49 100K pull-down | R108 100K / R49 **12K** UVLO divider, Q24 shutdown FET with R109 100K / R110 27K gate bias; net `nENDAQ`, active low | DAQ 11 V undervoltage lockout (W16 handoff §4.4) |
| R41, R45 | 2.2R 1% 1206, 1/4 W | 2.2R 1% **1210**, 1/2 W (any standard thick film) | Snubber resistors. R41 (SW1) dissipates 0.31 W at 24 V and 0.38 W at the 26.3 V OV trip, 75% of 1/2 W: fine up to about 90 °C local ambient. R45 (SW2) 0.167 W, 33%. Power is set by C·V²·f, not the load, so it is continuous at 1.157 MHz; power-save mode lowers it at light load. Same length as 1206; the pads are 0.45 mm wider each side |
| Q4, Q5, Q12–Q15 | DMP3018SFV, PowerDI3333-8 (Type UX), imported Altium land | **DMP3007SCG-13** (Diodes, V-DFN3333-8 Type B, 6.8 mΩ max at −10 V), `Diodes_PowerDI3333-8`, copied from KiCad's standard `Package_SON` library into the project library so it can carry the source-drain keepout | PowerPath/battery FET thermal margin (W16 handoff). KiCad has no Type B footprint; the PowerDI3333-8 one matches Diodes' Type B suggested land except 0.1 mm more toe on each pad row, and adds segmented paste on the drain. Its drain is one pad, 5, so the project symbol `DMP3007SCG V-DFN3333-8` numbers D pins 5–8 and EP_D all 5. VGSS ±25 V as before. Diodes' datasheet (DS38990 rev 3-2) is still marked Advance Information |
| Q18 | CSD17304Q3 | unchanged (kept) | The W16 handoff suggested a lower-RDS(on) 30 V part for the 1.83 W boost-mode loss. Kept: it works and 15 units are in field test. Its thermal margin depends on copper: 46 °C/W on 1 in² against 158 °C/W on a minimum pad, so give it the full copper area in layout |
| R16 | 0.005R 2512 (`CRE2512-FZ-R005E-3`) | **0.003R** Bourns `CRE2512-FZ-R003E-3` | The LTC1960 latching breaker (90–115 mV for 15 ms) moves from 18–23 A to 30–38 A. 12 V peak load is about 7.5 A SBC (90 W) + 2.5 A J3 + DAQ, over the handoff's ~11 A threshold, and QE29 packs at ~21 A would sit at 105 mV, inside the trip window. Dissipation at 21 A drops from 2.2 W to 1.3 W. R16 feeds only the breaker comparator (SCP/SCN) |
| R53, R71 | 1K | unchanged | DAQ and EXT current limits (6595/R = 6.6 A, up to 7.1 A) sit above J6/J5's single 6.5 A contact. Left at 1K: DAQ's final load isn't known (stand-in device today) and EXT is unused. A note on S07 says to raise them to 1.2K or more once the loads are known |
| D4 | Pins named as a series pair | Pins named for the fitted common-anode BAT54A | Drawing and wiring were already right |

Connection options on U5 (LM51772), as 0R 0402 links so either build can be
fitted (lower right of S06):
nRST stays on DCBMS in both BIAS builds.

| Pin | Fitted | DNP (alternative) |
|---|---|---|
| MODE (7) | R86 to GND: power-save mode (W16 review) | R87 back to VCC2/ILIMCOMP: forced PWM, as built before |
| BIAS (40) | R89 to +12V (U5's own output): the LM51772 datasheet's standard configuration (§7.3.5). VSMART runs the VCC2 regulator from the lower of VIN and BIAS once both exceed the switch-over threshold, so on the 24 V supply it drops from 12 V instead of 24 V; at start-up or with +12V shorted it falls back to VIN | R88 to DCBMS, as built on the field units |

U5 is configured over I2C by default, not by its CFG resistors (parts right
of the MODE/BIAS links on S06):

| Ref | Part | Function in the I2C build (default) |
|---|---|---|
| R38 | 0R, CFG1 (ADDR/SLOPE, pin 9) to GND | Selects I2C at address 0x6A. 6.49K instead selects resistor configuration with slope compensation 1.0 |
| R104 | 0R, SDA/CFG3 (pin 5) to J3.34 `MCU_ENDAQ` | SDA to the MCU |
| R105 | 0R, SCL/CFG4 (pin 6) to J3.36 `MCU_ENEXT` | SCL to the MCU |
| R106, R107 | 10K to +3V3MCU | SDA and SCL pull-ups (pins rated to 5.8 V). 10K keeps edges slow for EMC and limits the bus to 100 kHz standard mode |
| R30, R29 | 5.1K and 1.87K to GND, **DNP** | CFG3/CFG4 for the resistor build; fitted, they would load the bus |

To return to the resistor build, remove R104–R107, change R38 to 6.49K and fit
R29 and R30. J3.34 and J3.36 are also the direct-enable fallback through R97
and R98 (S08), so R97 and R98 stay unfitted while the I2C build is fitted. The
MCU pins on J3.34/J3.36 must support I2C.

U5 starts from its register defaults, not the resistor
settings, so firmware must write them before raising REGEN. They are latched
for as long as DCBMS is up, since nRST is tied to DCBMS. Defaults that differ
from the resistor build:
- slope compensation is 0.875 (R38 selects 1.0)
- VCC1, DRSS, hiccup mode and the current-limit options come from registers
  instead of R30 and R29
- the internal DAC becomes the current-limit reference (`ILIM_THRESHOLD`);
  ILIMCOMP is tied to VCC2, so confirm the average current limit stays
  disabled
- constant dead time (`EN_CONST_TDEAD`) defaults off; R30 (CFG3 setting 7)
  turns it on

Overvoltage lockout on the load switches U6, U7 and U9 (TPS22811, S07): a
1.07 MΩ 1% resistor (R90, R91, R92) from +12V to nEN/OVLO (pin 2), with the
existing 100K 1% pull-downs R51, R52 and R70. Each switch turns off above
14.0 V (13.40–14.69 V over the ±2% pin threshold, 1% resistors and ±0.1 µA pin
leakage) and turns back on below 12.75 V (12.23–13.41 V), so a tripped switch
always recovers on U5's 12.03 V output. (1 MΩ gave 13.2 V, recovering at
11.50–12.60 V, which might not.) To revert, leave R90–R92 unfitted.
The power-good dividers on PGTH are R59, R61 and R76 (100K) over R60, R62 and
R77 (13K): PG rises at 10.43 V (10.93 V worst case) and falls at 9.47 V
(at least 9.05 V). They were 1M over 130K, the same ratio, but the ±1 µA PGTH
leakage across 1M could put the threshold at 11.84 V, within 0.2 V of the rail.
Undervoltage lockout on the DAQ switch U6 only: R108 100K from +12V over R49
12K on EN/UVLO (pin 1) turns U6 on at 11.2 V (10.77–11.69 V, including
±0.1 µA pin and 0.5 µA FET leakage) and off at 10.17 V (9.83–10.66 V). The
TPS22811's fixed hysteresis makes the turn-off point about 9% below turn-on,
so 11 V has to be the turn-on point. Because the divider now holds EN/UVLO,
the enable goes through Q24 (BSS138) from EN/UVLO to GND, and the expander net
is `nENDAQ`, active low: high or floating (POR, expander in reset or
unpowered) turns Q24 on and puts U6 in shutdown; low releases EN/UVLO to the
divider. R109 100K / R110 27K bias the gate from +12V to 2.55 V (3.1 V at the
14.7 V OVLO limit), so the channel stays off when the MCU board is
unpowered, and the gate stays under 3.3 V, so P0_1 can stay push-pull and
the R97 fallback can drive it from a 3.3 V MCU pin. SBC and EXT have no
UVLO. A note on S07 marks the DAQ circuit as deliberate.

Rail enables and faults through a PCAL9714 SPI I/O expander (U11, new sheet
S08). It shares CLOCK/MOSI/MISO with U1 (LTC1960) and has its own chip
select, nCSIO, on J3.32, the pin that was ENSBC. Address 40h (ADDR to GND);
INT has a pull-up but isn't routed off the board. Both supplies are
+3V3MCU. RESET is held low by R100 (100K) and pulled to +3V3MCU by Q23
(BSS138, the same FET as Q20-Q22) whenever +12V is up: R99 100K /
R101 1.07M put about 11 V on its gate. With the 12 V rail off the expander is
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
| P0_0 (out) | ENSBC | U7 enable |
| P0_1 (out) | nENDAQ | U6 enable through Q24, **active low** |
| P0_2 (out) | ENEXT | U9 enable |
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
and pulls off, so the pull-downs (R50, R69) and Q24's gate bias keep the
rails off until firmware sets P0_0–P0_2 as outputs. R97 now lands on
`nENDAQ`, so an MCU driving DAQ directly must drive it low to enable. The MCU firmware must change for this:
the three load-switch enables move from GPIO to SPI, and nCSIO must idle high (R93
pulls it up).

These parts aren't on the board yet: run Update PCB from Schematic, then
place and route R86–R110, Q23, Q24, U11, C58 and C59, and re-place Q20–Q22
(now SOT-23). The ENDAQ, ENSBC and ENEXT
tracks from J3 need cutting at the new links.

## BOM consolidation

Parts with the same value in different packages or ratings now share one BOM
line. Each change keeps or raises the voltage rating and tolerance.
All resistors are 1%: the OVLO dividers moved from 1M/102K 0.1% to
1.07M/100K 1% (see the overvoltage lockout above), and R101 is 1.07M to share
their BOM line.

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

Run Update PCB from Schematic to swap the footprints. Parts that grow overlap
their neighbours' courtyards and need moving: the 0603s for C8, C9 and C55 by
up to 0.9 mm (C55 against C5 and C7, C8 against R8, C9 against R15 and C10),
the 1210s by about 0.15 mm (C11/C12/C27, C24/C26, C25 against C23), and C32
by 0.3 mm against C38 (before C38 shrinks to 0402) and 0.1 mm against TH2.

## Fixes after import

Each fix is a separate commit on top of the initial import.

| Problem in the raw import | Fix |
|---|---|
| Every symbol's Value field was `${ALTIUM_VALUE}`, resolved through a per-symbol field | Copied the real value into Value (209 symbols); the 26 test points, which had no value, are `POGO-1` |
| 12 Footprint fields differed from the placed footprint only in letter case, so library lookup failed | Fields now name the exact footprint used on the PCB (J2, J5, J6, M2–M6, FL1, D9, D10, L2) |
| An imported `FOOTPRINT` field holding the bare package name (`0402`, `SMA`, …) overrode the real Footprint field, since KiCad matches field names case-insensitively | Renamed to `ALTIUM_FOOTPRINT` on all 237 symbols |
| Altium net labels became KiCad local labels, which don't connect across sheets, splitting 35 nets | Converted the 78 labels for those nets to global labels |
| 44 embedded library symbols drew Altium special strings `=Value` and `=FootPrint` as literal text | `=Value` replaced with `${VALUE}`; `=FootPrint` replaced by showing each part's own `ALTIUM_FOOTPRINT` field in the same place, so parts show their part number and package as in Altium |
| All 99 power symbols were unannotated (`#PWR?`) | Annotated `#PWR1`–`#PWR99` |
| The "Lattech Systems" symbol library wasn't available, so symbols existed only as copies embedded in the schematics, some in several slightly different versions | Exported the symbols to project libraries under their original library names, and made every schematic use one definition per symbol |
| Chip resistor and capacitor footprints had poor pads (0402s used 0.635 mm circular pads) and caused tombstoning | Replaced all 139 with KiCad standard IPC-7351 nominal footprints (`Resistor_SMD`, `Capacitor_SMD`) in place: same position, rotation and nets, zones refilled |
| PCB footprints were linked to the imported symbols by mismatched internal IDs, and nets used Altium names | Ran Update PCB from Schematic: no errors, placement and copper unchanged, all 147 nets match the schematic |
| KiCad design rules were defaults (0.2 mm clearance and track, 0.3 mm min hole), not the Altium rules | Set to the rules in the Altium `.PcbDoc`: 0.15 mm clearance, track and annular ring, 0.15 mm min hole, 0.3 mm hole-to-hole, and `EVO2513.kicad_dru` for the 0.3 mm pad-to-pad rule on test points TP1–TP26 |
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
| Every resistor and capacitor had its own per-value Lattech symbol (76 of them), and the schematic showed the `ALTIUM_VALUE` field, not Value, so R1, C39 and R54 still displayed their old values | Swapped all 160 for KiCad's `Device:R`, `Device:C` and `Device:C_Polarized` (C2, C28, C40–C42) in place, connections unchanged; Value is now the displayed field, and the 76 symbols are gone from the project library. `ALTIUM_VALUE` brought in line with Value on R1, C39, R54, D1, Q1, Q3 and L1 |
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
- Every power FET footprint (Q1, Q3–Q19) has a top-copper keepout (no pour, tracks or
  vias) across the gap between its source/gate row and its drain pad, so the gap is bare
  laminate as in the manufacturers' land patterns, not just solder mask over a pour.
  `EVO2513.kicad_dru` also keeps other nets' pours 0.3 mm from those FETs' pads. Both
  protect the source-drain gap during rework.
- The design relies on via-in-pad: over 700 vias sit in pads (FET tabs, QFN
  exposed pads, FL1, the 2512 shunts, and 0402/0603 capacitors). Order the
  board with vias filled and capped; open vias would wick solder from the
  small pads.

Generated outputs (netlists, backups, per-user `.kicad_prl` settings) are not
tracked; see `.gitignore`.
