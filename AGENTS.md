# OpenESC-30x30

Open source 4-in-1 BLDC ESC, 30.5 x 30.5 mm mounting pattern. Four fully
independent channels, each with its own MCU, gate driver and six MOSFETs: the
distributed-MCU AM32 topology, not a single shared MCU. Control is DShot over
the standard 8-pin connector.

## Repo

| | |
|---|---|
| Maintainer | @Just4Stan |
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| KiCad project | `hardware/4in1.kicad_pro` |
| Root schematic | `hardware/4in1.kicad_sch` (power, current sense, connector) plus `hardware/ESC.kicad_sch`, one channel instantiated 4x |
| Board | `hardware/4in1.kicad_pcb`, 6 layers, 1.6 mm, 2 oz outer copper (the stackup field reads 1.69 mm; JLC ships 1.6) |
| Fixtures | [OpenDrone-Fixtures](https://github.com/OpenDrone-hw/OpenDrone-Fixtures): `OpenESC-30x30-QC/` press-contact QC fixture, `OpenESC-30x30-Flashing/` ST-LINK pogo-pin jig, both unrouted |
| Production panel | `hardware/4in1-panel.kicad_pcb`, carries this board **and** the OpenESC-20x20, not a panel of this board alone |
| Fab exports | `hardware/production/`, Fabrication Toolkit sets named by revision |
| Local library | `hardware/components.kicad_sym`, `hardware/4in1ESC-30x30.pretty/`, `hardware/4in1ESC-30x30.3dshapes/`. Frozen pre-consolidation libraries: use them, do not add to them |
| Shared library | [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), catalogue only; every library this board uses is local to the repo |
| Design rules | `hardware/4in1.kicad_dru` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | Line standard: 6 layers, 0.09 mm clearance and track, via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

The project is named `4in1`, not after the repo. Renaming it would break the
fab archive names, the release assets and the website board art, so it stays.

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
- **Reuse before you draw.** Check
  [KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library) and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it: copy the symbol and footprint into this repo's
  `lib` library and use it. Draw a new part only when the library has nothing
  that fits, and import it with `easyeda2kicad` from its LCSC number.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **ERC and DRC clean before every pull request.** Commands below.

## Environment

```sh
# schematic and board checks
kicad-cli sch erc --exit-code-violations hardware/4in1.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones --exit-code-violations hardware/4in1.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/4in1.net hardware/4in1.kicad_sch
```

`--refill-zones` stops stale fills inventing clearance errors. Keep
`--schematic-parity` on, but read its output against Layout rules below: the
board-only bulk bank makes parity noisy, and a real error hides easily in it.

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python. Shared scripts (renders, STEP export,
packaging art) live in `OpenDrone-Scripts`.

## Architecture

Four independent channels share one power input and one connector. Per channel:
an **AT32F421G8U7** (Cortex-M4, QFN-28) drives an **NSG2065Q** three-phase
half-bridge gate driver, which drives six **SP40N01GHNK** MOSFETs, two per
phase. One channel is drawn once in `ESC.kicad_sch` and instantiated four times.

Current sensing is **board level, not per channel**: a single INA186A3IDCKR at
100 V/V sits across two 0.2 mOhm 2512 shunts in parallel, 0.1 mOhm total, in the
+BATT feed. That gives 10 mV/A and roughly 330 A full scale against a 3.3 V ADC,
reported as `/CURR`.

**`/CURR` is expected unusable below full throttle.** The shunt is high side,
so both amplifier inputs ride the switching bus and the common-mode
feedthrough rectifies to a duty-dependent offset. Measured on the 20x20
sibling 2026-08-21, which shares this exact topology: errors from +164% to
-64% across the throttle range, a raw-ADC-level hard fault at 80% throttle,
honest only at 100% where AM32 stops chopping (+11 to +12% gain error there).
This board has not been measured itself, but nothing in its identical
high-side arrangement escapes the mechanism. The fix is a respin: shunt moved
low side, or a matched input network at the amplifier. Details and evidence
in the OpenESC-20x20 AGENTS.md and
`OpenDrone-Testing/Logs/esc-04-20x20-20260821T150430Z/`.

**There is no input protection.** The three clamp diodes (D1-D3) that earlier
revisions carried are gone: their 24 V standoff sits below the 33.6 V an 8S pack
reaches, so they were removing themselves. 2S-8S is qualified by bench and
flight testing, not by a clamp. The practical envelope is set by the MOSFET
(40 V VDSS), the buck (36 V rated, 45 V absolute maximum) and the current-sense
full scale.

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| Motor MCU, x4 | U2, U5, U7, U9 | AT32F421G8U7, QFN-28 | C2765098 | One per channel |
| Gate driver, x4 | U4, U6, U8, U10 | NSG2065Q, QFN-24 | C41414478 | FD6288Q compatible, integrated diodes |
| Power MOSFET, x24 | Q1-Q24 | SP40N01GHNK, PDFN-8L 5x6 | C22385416 | 6 per channel. XRS280N03C (C50314140) is a selected drop-in successor, not fitted |
| Current sense amp | U12 | INA186A3IDCKR, SC-70-6 | C2058245 | 100 V/V, board level high side |
| Current shunt, x2 parallel | Rsense1, Rsense2 | 0.2 mOhm 2512 | C695806 | 0.1 mOhm combined |
| Buck | U13 | LMR54406DBVR, SOT-23-6 | C5219316 | 1.1 MHz, 0.6 A; FB 115k/10k against 0.8 V for 10.0 V out |
| Buck inductor | U14 | FTC160808S4R7MBCA | C46594347 | 4.7 uH |
| LDO | U15 | TLV76733DRVR, WSON-6 | C2848334 | +10 V to +3V3 |
| Connector | J1 | SM08B-SRSS-TB, JST SH 8-pin | C160407 | Also broken out as solder pads (U3) |
| Bulk electrolytic, supplied | n/a | 470 uF | | Shipped with the board, not fitted to it. The user solders it across the battery terminals. Standard pairing of on-board ceramics with a pack-side elco; it dominates the bus capacitance once installed |

## Power

```
+BATT ─┬─► MOSFET drains, motor phases
       ├─► shunt pair ─► INA186A3 (U12) ─► /CURR
       └─► LMR54406DBVR buck (U13) + 4.7uH (U14) ─► +10V ─┬─► gate drivers U4/U6/U8/U10
                                                          └─► TLV76733DRVR (U15) ─► +3V3 ─► 4x MCU, INA186
```

## Connectors and I/O

8-pin JST SM08B-SRSS-TB (J1). The same eight signals are broken out as solder
pads (U3). Connector ground returns on pads P1 and P2.

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

## Firmware

[AM32](https://github.com/am32-firmware/AM32), one independent target per
channel. Boards ship with the AM32 bootloader pre-loaded; firmware is flashed
and configured in-browser at [am32.ca](https://am32.ca). The AT32F421 plus
NSG2065Q per-channel topology is the standard AM32 hardware target for this
board class, so it works with Betaflight and any other DShot-capable flight
controller.

## Layout rules

Bulk decoupling on +BATT and GND exists on the PCB without matching schematic
symbols. That is a deliberate board-only bank, and it is why DRC parity reports
are noisy here. Do not run update-from-schematic without checking what it would
delete.

The board outline follows the scalloped pad edges of the solder-pad breakout
footprint (U3). The extent past the 30.5 mm mounting square is real outline
geometry, not scratch: do not tidy it away.

## Revisions

| Rev | Date | Change |
|---|---|---|
| Rev3.1 | 2026-08-14 | Export `30x30-Rev3.1`, current. Bulk bank: 52 x 10 uF 1206 on +BATT/GND, 49 of them PCB-only (only C2, C3, C6 are in the schematic; 24 added since rev3). Board setup on the line standard. |
| Rev3 | 2026-08-11 | Input clamp diodes D1-D3 removed, C2 and C3 doubled. |
| Rev1 | 2026-06-05 | Validated build. Fab sets `Rev1-30x30` and `Rev1-30x3020x20`, the latter combined with OpenESC-20x20. |
| V0.4 | 2026-05-29 | Combined export `V0.4-20x20-30x30`. |
| V0.3 | 2026-05-06 | Export `V0.3`; combined `V0.3-20x20-30x30` on 2026-05-12. |
| V0.2 | 2026-05-05 | Export `V0.2`. |
| V0.1 | 2026-03-18 | First production export. |
