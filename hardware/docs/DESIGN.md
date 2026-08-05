# OpenESC-30x30 Design Notes

Detailed design description of the OpenESC-30x30. Values are extracted from the KiCad design files (`hardware/4in1.kicad_sch`, `hardware/ESC.kicad_sch`, `hardware/4in1.kicad_pcb`) and the production BOM (`hardware/production/Rev1-30x30_bom.csv`).

## Architecture

Four fully independent ESC channels share a common power input and telemetry connector. Each channel has its own MCU and gate driver; the high-current stage is six MOSFETs per channel (three half-bridges). This is the distributed-MCU AM32 topology rather than a single-MCU design.

| Block | Part | LCSC | Per board |
|---|---|---|---|
| Motor MCU | AT32F421G8U7 (QFN-28) | C2765098 | 4 |
| Gate driver | NSG2065Q (QFN-24) | C41414478 | 4 |
| Power MOSFETs | SP40N01GHNK PDFN-8L (5x6) | C22385416 | 24 (6 per channel) |

## Full specifications

| Parameter | Value |
|---|---|
| Channels | 4 independent BLDC channels |
| MCU | AT32F421G8U7 (ARM Cortex-M4, QFN-28), one per channel |
| Gate driver | NSG2065Q (QFN-24, FD6288Q-compatible), one per channel |
| Power MOSFETs | SP40N01GHNK, N-channel, PDFN-8L (5x6), 24 total (XRS280N03C evaluated as drop-in replacement, not used in Rev1) |
| Current sense | Board-level high-side: INA186A3IDCKR (100 V/V, SC-70-6) across 2x 0.2 mOhm 2512 shunts in parallel (0.1 mOhm) in the +BATT feed, 10 mV/A, ~330 A full-scale at 3.3 V ADC |
| Input | +BATT direct from connector/pads, 3S-6S |
| Input protection | 3x SMBJ24A TVS (24 V standoff) |
| Buck regulator | LMR54406DBVR (SOT-23-6) + FTC160808S4R7MBCA 4.7 uH inductor, produces the +10 V gate-drive rail (FB 115k/10k, Vref 0.8 V, 10.0 V out) |
| LDO | TLV76733DRVR (WSON-6), +10 V in, +3V3 out (MCUs, sensing) |
| Signal protocol | DShot (4 independent signal lines, one per channel) |
| Firmware | AM32 (per-channel AT32F421 target, flashed individually) |
| PCB | 6-layer, 1.69 mm |
| Mounting pattern | 30.5 x 30.5 mm, 4x 4.0 mm holes (M3) |

Current and voltage ratings are not printed in the design files. The input clamp is set by the SMBJ24A TVS (24 V standoff, 3S-6S); the MOSFET (SP40N01GHNK) and current-sense full-scale (~330 A) bound the practical envelope. Characterize before quoting a hard rating.

## Power tree

+BATT (3S-6S) feeds the MOSFET drains directly, the current shunts, and the LMR54406DBVR buck. The buck produces +10 V for the four gate drivers; the TLV76733DRVR LDO drops +10 V to +3V3 for the four MCUs and the current-sense amplifier. Input clamp: 3x SMBJ24A TVS.

## Connector

8-pin JST **SM08B-SRSS-TB** (J1). Pin-to-net mapping extracted from the schematic (net labels at the connector pins):

| Pin | Net | Function |
|---|---|---|
| 1 | +BATT | Battery positive |
| 2 | GND | Ground |
| 3 | /CURR | Current-sense telemetry (INA186 output) |
| 4 | *(unconnected)* | No dedicated telemetry pin, telemetry handled by extended DShot |
| 5 | /M1 | DShot signal, channel 1 |
| 6 | /M2 | DShot signal, channel 2 |
| 7 | /M3 | DShot signal, channel 3 |
| 8 | /M4 | DShot signal, channel 4 |

Connector ground returns on the shield/mounting pads P1/P2 (both GND). The same eight signals are also broken out as direct solder pads (U3). Pin 4, the dedicated telemetry pin on the Betaflight 8-pin standard, is intentionally unconnected: ESC to FC telemetry is carried over the motor signal lines via the bidirectional **extended DShot** protocol.

## Variants and revisions

| File | Description |
|---|---|
| `hardware/4in1.kicad_pro` / `.kicad_pcb` / `.kicad_sch` | Main design (30x30) |
| `hardware/4in1-panel.kicad_pro` / `.kicad_pcb` | Panelized version for production fabrication |

This repo is the 30x30 member of the OpenESC family. A smaller sibling, [OpenESC_20X20](https://github.com/incutec-hw/OpenESC_20X20) (20x20 mm), shares this design and mirrors this repo; the two differ only in board/mounting size and a few power-stage parts. Production exports in `hardware/production/` are versioned `V0.1` through `V0.4` and `Rev1`. `Rev1-30x30` is the 30x30-only fabrication set; `Rev1-30x3020x20` is a combined shared-fab set produced alongside the 20x20.

## Firmware

[AM32](https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware) is incutec's default ESC firmware. Boards ship with the AM32 bootloader pre-loaded; firmware is flashed and configured in-browser at [am32.ca](https://am32.ca). Each channel's AT32F421G8U7 is an independent AM32 target. The AT32F421 + NSG2065Q per-channel topology and the DShot signal nets are the standard AM32 hardware target for this board class. Works with Betaflight and other DShot-capable flight controllers.
