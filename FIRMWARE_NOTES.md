# EVO2513 v0.6.0: notes for the MCU firmware

What the MCU firmware has to know about this board revision. The hardware is
described in [README.md](README.md); this file covers only what changes how
firmware drives or reads the board. Items marked **Changed** differ from the
0.5.0 boards in field test.

## SPI bus

U1 (LTC1960) and U11 (PCAL9714 I/O expander) share CLOCK (J3.16), MOSI
(J3.18) and MISO (J3.20). Each has its own chip select:

| Device | Chip select | J3 pin |
|---|---|---|
| U1 LTC1960 | nCS | J3.22 |
| U11 PCAL9714 | nCSIO, through R95 | J3.32, which was ENSBC on the field units |

- **Changed:** nCSIO must idle high. R93 (10K) pulls the expander's CS up.
- Keep the SPI clock at the rate already used for the LTC1960. At 5 MHz the
  10th harmonic lands at 50 MHz, in the same band as the converter's
  boost-mode switch-node ringing.

## I/O expander (U11, PCAL9714)

Address 40h (ADDR to GND). Both supplies are +3V3MCU.

### Reset and start-up

RESET is held low until +12V is up: Q23 pulls it to +3V3MCU once +12V
passes roughly 3.5–4 V, with no hysteresis. Wait for +12V before configuring
the expander, and reconfigure it whenever +12V has dropped, because every
port returns to an input.

At power-up every port is an input with pull-ups off, so the load switches
stay off until firmware sets P0_0–P0_2 as outputs.

### Port map

| Port | Direction | Net | Function | Active |
|---|---|---|---|---|
| P0_0 | out | ENSBC | SBC load switch (U7) enable | high |
| P0_1 | out | nENDAQ | DAQ load switch (U6) enable, through Q24 | **LOW (Changed)** |
| P0_2 | out | ENEXT | EXT load switch (U9) enable | high |
| P0_3 | — | — | spare, not connected | — |
| P0_4 | in | PGDAQ | U6 power good | high |
| P0_5 | in | PGSBC | U7 power good | high |
| P0_6 | in | PGEXT | U9 power good | high |
| P0_7 | in | nFLT | U5 (LM51772) fault/interrupt | low |
| P1_0–P1_5 | — | — | spare, not connected | — |

- **Changed: P0_1 is inverted.** Write 0 to enable DAQ and 1 to disable it.
  High or floating holds U6 in shutdown, including while the expander is in
  reset. The Q24 gate is biased to about 3.2 V through 286K, so P0_1 can stay
  push-pull: driving it high to 3.3 V sinks under 3 µA and still disables DAQ.
- All three enables move from MCU GPIO to SPI.
- Output drive strength: set the used outputs to 25 % (registers 40h–43h).
  Every output drives only a 100K-class load or lighter, so fast edges add noise and
  give nothing back.
- INT has a 10K pull-up (R94) but isn't routed off the board. Poll the input
  port for PG and nFLT changes.
- Unused ports P0_3 and P1_0–P1_5 are unconnected. Leave them as inputs with
  the internal pull-ups or pull-downs on, so they don't float.

### Direct-drive fallback (assembly option)

The links R96 (ENSBC), R97 (nENDAQ) and R98 (ENEXT) are unfitted as built.
If one is fitted to drive an enable straight from J3, leave that expander
port an input. **R97 lands on nENDAQ, so an MCU driving DAQ directly must
drive it low to enable** (3.3 V push-pull or open-drain both work).

### DAQ UVLO bypass (assembly option)

If R115 is fitted (with Q24, R108 and R109 removed), DAQ loses its
undervoltage lockout and its enable matches SBC and EXT: **P0_1 becomes
active high** (write 1 to enable), and so does the R97 fallback. A board built
this way needs the DAQ polarity flipped in firmware, and the undervoltage rows
in the table below no longer apply to DAQ.

## Load switch behaviour (TPS22811)

These thresholds are set in hardware. Firmware just has to expect them.

| | DAQ (U6) | SBC (U7) | EXT (U9) |
|---|---|---|---|
| Turns on above (undervoltage) | **11.1 V** (10.6–11.9 V), **Changed** | — | — |
| Turns off below (undervoltage) | **10.1 V** (9.7–10.9 V), **Changed** | — | — |
| Turns off above (overvoltage) | 14.0 V (13.4–14.7 V), **Changed** | same | same |
| Back on below (overvoltage) | 12.75 V (12.2–13.4 V) | same | same |
| PG asserts above | 10.38 V (at most 11.17 V), **Changed** | same | same |
| Current limit (6595 / R_IMON) | 6.6 A (R53 = 1K) | 8.6 A (R54 = 768R) | 6.6 A (R71 = 1K) |

An enabled switch can therefore be off with PG low for reasons other than a
fault: undervoltage on DAQ, or overvoltage on any channel. Read +12V status
before treating PG low as a load fault.

## Current monitor outputs

Each TPS22811 IMON pin drives a TLV9002 non-inverting stage with a gain of
1 + 21.5K/10K = 3.15, then 100R to J3.

V(IAMP) = I_OUT × G_IMON × R_IMON × 3.15, with G_IMON = 95.3 µA/A typical
(87.8–102.4 µA/A).

| Output | J3 pin | R_IMON | Scale (typ.) | Range from G_IMON tolerance | At the current limit |
|---|---|---|---|---|---|
| IAMPDAQ | J3.19 | R53 1K | 300 mV/A | 277–323 mV/A | about 2.0 V at 6.6 A |
| IAMPSBC | J3.17 | R54 768R | 231 mV/A | 212–248 mV/A | about 2.0 V at 8.6 A |
| IAMPEXT | J3.21 | R71 1K | 300 mV/A | 277–323 mV/A | about 2.0 V at 6.6 A |
| IAMPREG | J3.23 | — | 2 × U5's CDC pin voltage (gain 1 + 10K/10K) | — | — |

Calibrate per board if better than about ±8 % is needed. R53 and R71 are
expected to rise to 1.2K or more once the final DAQ and EXT loads are known
(a note on S07 explains why). That changes their scale to 360 mV/A at 1.2K.

## Converter (U5, LM51772)

- **Changed:** MODE is tied to GND (R86), so U5 runs in power-save mode and
  skips pulses at light load. The field units ran forced PWM. Expect more
  output ripple at light load, at a lower switching rate.
- **Changed:** BIAS is on +12V (R89). Firmware sees no difference.
- REGEN (J3.12) still enables U5 directly; it doesn't go through the expander.
- nFLT is now readable on expander P0_7.

### I2C configuration (default build)

**Changed:** U5 is configured over I2C at address 0x6A on SDA = J3.34 and
SCL = J3.36 (R104–R107 fitted, R38 0R, R29/R30 unfitted). The MCU pins must
support I2C. U5 starts from its register defaults, not the resistor settings,
so firmware must write them before raising REGEN.

- **Run the bus at 100 kHz (standard mode) or slower.** The pull-ups are 10K
  (R106, R107) to keep edges slow for EMC. Rise time is about
  0.85 × 10K × C_bus: about 850 ns at 100 pF, inside standard mode's 1000 ns
  limit, but well over fast mode's 300 ns. Don't add strong pull-ups on the
  MCU side; they would undo the slow edges. The settings stay latched
while DCBMS is up (nRST is tied to DCBMS). Defaults that differ from the
resistor build:

- slope compensation is 0.875 (R38 selects 1.0)
- VCC1, DRSS, hiccup mode and the current-limit options come from registers
  instead of R30 and R29
- the internal DAC becomes the current-limit reference (`ILIM_THRESHOLD`);
  ILIMCOMP is tied to VCC2, so confirm the average current limit stays
  disabled
- constant dead time (`EN_CONST_TDEAD`) defaults off; R30 (CFG3 setting 7)
  turns it on

**Set `DRSS_EN` = 1 (MFR_SPECIFIC_D0, register 0xD0, bit 2).** It resets to 0,
so without this write U5 switches at a fixed 1.157 MHz. The 0.5.0 boards had
spread spectrum on through R29 (1.87K, CFG4 setting 4). DRSS dithers the
switching frequency between f_SW × 0.922 and f_SW × 1.078 (about 1.067 to
1.247 MHz). That spreads the switching harmonics and the 51–81 MHz SW2 ring,
which is retriggered every cycle in boost mode (10–11 V packs), so its
radiated peaks drop. SYNC (U5 pin 3) is tied to GND here, so nothing
overrides it. Use a read-modify-write:
- MFR_SPECIFIC_D0 resets to 0x32 (EN_VCC1, IMON_LIMITER_EN and USLEEP_EN
  set). Setting bit 2 gives 0x36.
- Bit 0 is CONV_EN, so set DRSS_EN before or together with enabling the
  converter.
- CFG4 setting 4 on the 0.5.0 boards also enabled hiccup short-circuit
  protection (`HICCUP_EN`, bit 3, reset 0). To match those boards, set bit 3
  as well (0x3E with CONV_EN still clear).

The I2C build uses the same J3 pins as the R97/R98 direct-drive fallback, so
R97 and R98 must stay unfitted. The resistor build (R104–R107 removed, R38
6.49K, R29/R30 fitted) is still available as an assembly option.

## Battery manager (U1, LTC1960)

- **Changed:** R16 is 3 mΩ, so the latching breaker trips at 30–38 A
  (90–115 mV for 15 ms) instead of 18–23 A. When it trips, every PowerPath
  switch opens, including DC, and FA is set. It clears only when all sources
  are removed and VPLUS falls below UVLO, so firmware can report FA but
  can't clear it.
- **Changed:** R14 is 12 mΩ (was 10 mΩ). The adapter current limit drops to
  8.33 A, below the 9.2 A supply, so charging throttles before the supply
  trips. Scale any reading or setting that assumed 10 mΩ by 10/12.
- ED29 test packs trip at 8 A each (measured); QE29 production packs at
  10.5 A. Any limit keyed to 10.5 A applies to QE29 only.

## Pack communication

**Changed:** R24–R27 are 0R and D9/D10 are unfitted. The pack SMBus/SAFE
lines (J2) now pass straight through to J3 with no series resistance or
clamp on this board, so the MCU board must provide the protection.
