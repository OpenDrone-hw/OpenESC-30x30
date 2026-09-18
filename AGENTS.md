# OpenESC-30x30

Open-Source 4-in-1 sensorless BLDC motor Electronic Speed Controller (ESC), 30.5
x 30.5 mm mounting pattern (FPV Drone standard). Part of OpenDrone by Incutec
product lineup.

## Architecture

Four independent channels share one power input and one connector. Per channel:
an **AT32F421G8U7** (Cortex-M4, QFN-28) drives an **NSG2065Q** three-phase
half-bridge gate driver, which drives six **SP40N01GHNK** MOSFETs, two per
phase. One channel is drawn once in `ESC.kicad_sch` and instantiated four times.

Current sensing is **board level, not per motor**: a single INA186A3IDCKR at 100
V/V sits across two 0.2 mOhm 2512 shunts in parallel, 0.1 mOhm total, in the
+BATT feed. That gives 10 mV/A and roughly 330 A full scale against a 3.3 V ADC,
reported as `/CURR`.

## Power

```
Battery + (2S-8S) ─► 0.1mOhm shunt ─► +BATT
+BATT ─┬─► MOSFET drains, motor phases
       └─► LMR54406DBVR buck ─► +10V ─┬─► 4x gate driver
                                      └─► TLV76733DRVR ─► +3V3 ─► 4x MCU, INA186
```

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| Motor MCU, x4 | U2, U5, U7, U9 | AT32F421G8U7, QFN-28 | C2765098 | One per channel |
| Gate driver, x4 | U4, U6, U8, U10 | NSG2065Q, QFN-24 | C41414478 | Standard footprint, many alternatives exist. |
| Power MOSFET, x24 | Q1-Q24 | SP40N01GHNK, PDFN-8L 5x6 | C22385416 | 40 V, 6 per channel. Standard 5x6 DFN footprint, many alternatives exist. |
| Current sense amp | U12 | INA186A3IDCKR, SC-70-6 | C2058245 | 100 V/V, board level high side |
| Current shunt, x2 parallel | Rsense1, Rsense2 | 0.2 mOhm 2512 | C695806 | 0.1 mOhm combined |
| Buck | U13 | LMR54406DBVR, SOT-23-6 | C5219316 | 1.1 MHz, 0.6 A; FB 115k/10k against 0.8 V for 10.0 V out |
| Buck inductor | U14 | FTC160808S4R7MBCA | C46594347 | 4.7 uH |
| LDO | U15 | TLV76733DRVR, WSON-6 | C2848334 | +10 V to +3V3 |
| Connector | J1 | SM08B-SRSS-TB, JST SH 8-pin | C160407 | Also broken out as solder pads. |
| Bulk electrolytic. | / | 470 uF 50V | / | To be installed on the battery connector by the user. |
| Bulk ceramic | See PCB | 4.7 uF 1206, X5R 50 V | C380366 | 52 fitted |

## Connectors and I/O

Betaflight Standard:

| Pin | Net | Function |
|---|---|---|
| 1 | +BATT | Battery positive |
| 2 | GND | Ground |
| 3 | /CURR | Current sense telemetry, INA186 output |
| 4 | unconnected | See below |
| 5 | /M1 | DShot, channel 1 |
| 6 | /M2 | DShot, channel 2 |
| 7 | /M3 | DShot, channel 3 |
| 8 | /M4 | DShot, channel 4 |

Pin 4 is the dedicated telemetry pin in the Betaflight 8-pin standard and is
intentionally left unconnected: ESC to FC telemetry rides the motor signal lines
over bidirectional extended DShot instead.

## Layout rules

Bulk decoupling on +BATT and GND exists on the PCB without matching schematic
symbols. That is a deliberate board-only bank. Do not run update-from-schematic
without checking what it would delete.

## Firmware

[AM32](https://github.com/am32-firmware/AM32). The bootloader
(`AM32_F421_BOOTLOADER_PB4_V19.hex`) is loaded first with an ST-LINK; the
firmware is then flashed and configured in-browser at
[am32.ca](https://am32.ca). Works with Betaflight and any other DShot-capable
flight controller.

## Repo

| | |
|---|---|
| Maintainer | @Just4Stan (Discord: juststan_) |
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| KiCad project | `hardware/4in1.kicad_pro` |
| Root schematic | `hardware/4in1.kicad_sch` (power, current sense, connector) plus `hardware/ESC.kicad_sch`, one channel instantiated 4x |
| Board | `hardware/4in1.kicad_pcb`, 6 layers, 1.6 mm, 2 oz outer copper, 1 oz inner copper. |
| Fixtures | [OpenDrone-Fixtures](https://github.com/OpenDrone-hw/OpenDrone-Fixtures): `OpenESC-30x30-QC/` press-contact QC fixture, `OpenESC-30x30-Flashing/` ST-LINK pogo-pin jig |
| Local library | `hardware/components.kicad_sym`, `hardware/4in1ESC-30x30.pretty/`, `hardware/4in1ESC-30x30.3dshapes/`. Frozen pre-consolidation libraries: use them, do not add to them |
| Shared library | `hardware/KiCad-Library/`, pinned submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; 3D models and exact component datasheets resolve through `OPENDRONE_LIB` |
| Design rules | `hardware/4in1.kicad_dru` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | 6 layers, 0.09 mm clearance and track, 0.16mm on outer layers (2 oz), via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

The project is named `4in1`, not after the repo. Renaming it would break the
fab archive names, the release assets and the website board art, so it stays.

## Environment

```sh
# schematic and board checks
kicad-cli sch erc hardware/4in1.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones hardware/4in1.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/4in1.net hardware/4in1.kicad_sch
```

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python. Reusable scripts for renders, STEP export,
and packaging art come from Incutec hardware tooling. The OpenDrone release
standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
Board-specific scripts, where a board has any, live in `hardware/tools/`.

## Rules

Identical in every OpenDrone board repo. Do not edit here; edit the template.

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields (MPN, Manufacturer, LCSC, Cost, Datasheet, text variables). An agent
  may not change nets, wiring, routing, placement, footprint assignment, or any
  value that changes the circuit.
- **Close KiCad before any write to a KiCad file.** KiCad caches library tables
  at process start and overwrites files on save.
- **Reuse before you draw.** Check the `OpenDrone` library and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it, and its symbol links to the exact committed
  datasheet: place it from `OpenDrone`. Draw a new part into `lib` only when
  the catalogue has nothing that fits, imported with
  `easyeda2kicad` from its LCSC number. Pulling a newer catalogue is a
  deliberate, reviewed commit: `git submodule update --remote
  hardware/KiCad-Library`, then DRC.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Run ERC and DRC before every pull request.** Existing approved findings
  may remain; a new type or increased count must be reviewed before merge.
  Commands are in Environment above.

## Revisions

| Rev | Date | Change |
|---|---|---|
| Rev3.3 | 2026-08-25 | Export `OpenESC-30x30-rev3.3`. Silkscreen rebranded OpenDrone -> incutec for export restriction reasons on flagging anything containing 'Drone'. First Incutec production run. |
| Rev3.2 | 2026-08-22 | Export `OpenESC-30x30-rev3.2`. Matched input network at the current-sense amplifier (R89/R90 1k, C40/C41 100n 50V, C42 1u) against the high-side common-mode feedthrough. |
| Rev3.1 | 2026-08-14 | Export `30x30-Rev3.1` |
| Rev3 | 2026-08-11 | Input clamp diodes D1-D3 removed, TVS diodes offer no protection when rail voltage is this close to the MOSFET Vds. |
| Rev1 | 2026-06-05 | Validated build. 15 pieces ordered by Incutec. |
| V0.4 | 2026-05-29 | Combined export `V0.4-20x20-30x30`. |
| V0.3 | 2026-05-06 | Export `V0.3`; combined `V0.3-20x20-30x30` on 2026-05-12. |
| V0.2 | 2026-05-05 | Export `V0.2`. |
| V0.1 | 2026-03-18 | First production export. |
