# OpenESC-30x30

Open-source 4-in-1 BLDC ESC with a 30.5 x 30.5 mm mounting pattern, part of the incutec OpenDrone line. Four independent AT32F421G8U7 motor controllers, each with an NSG2065Q gate driver and six MOSFETs, run AM32 firmware and take DShot over the standard 8-pin connector. 3S-6S input, 6-layer PCB, designed in KiCad 10 for JLCPCB assembly. Full design detail: [hardware/docs/DESIGN.md](hardware/docs/DESIGN.md).

<p>
<img src="images/front.png" width="400" alt="OpenESC-30x30 top" />
<img src="images/back.png" width="400" alt="OpenESC-30x30 bottom" />
</p>

## Status

**Hardware validated**, Rev1, 2026-08-05.
Latest production exports are Rev1 (2026-06-05), generated with the KiCad Fabrication Toolkit for JLCPCB assembly: `Rev1-30x30` from `hardware/4in1.kicad_pcb` (this board only) and `Rev1-30x3020x20` from `hardware/4in1-panel.kicad_pcb` (combined panel with OpenESC-20x20).

## Certification

<a href="https://certification.oshwa.org/be000029.html">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/oshwa-certified-dark.svg" />
    <img src="images/oshwa-certified.svg" width="160" alt="OSHWA Certified Open Source Hardware, BE000029" />
  </picture>
</a>

OpenESC-30x30 is **certified open source hardware** by the [Open Source Hardware Association](https://www.oshwa.org/), OSHWA UID **[BE000029](https://certification.oshwa.org/be000029.html)**.

## Specifications

Full specifications and part-level detail (MCU, gate driver, power stage, current sense, protection) are in [hardware/docs/DESIGN.md](hardware/docs/DESIGN.md).

## Repository layout

| Path | Contents |
|---|---|
| `hardware/` | KiCad 10 project: schematics, PCB, panel, project-local libraries |
| `hardware/docs/` | Design documentation ([DESIGN.md](hardware/docs/DESIGN.md)) |
| `hardware/production/` | Fabrication exports per revision (generated, not tracked in git) |
| `libs/KiCad-Library` | Shared Incutec symbol/footprint/3D library (git submodule) |
| `images/` | Board renders and certification marks |
| `licensing/` | License notes, third-party notices, trademark policy |

## Design entry points

- Top schematic: `hardware/4in1.kicad_sch` (power, current sense, connector)
- Channel schematic: `hardware/ESC.kicad_sch`, instantiated 4 times
- Board layout: `hardware/4in1.kicad_pcb`, 6 copper layers
- Combined production panel: `hardware/4in1-panel.kicad_pro` / `hardware/4in1-panel.kicad_pcb`, this 30x30 board plus the OpenESC-20x20 board on one panel

Symbols and footprints are embedded in the design files, so the schematics and board open without any external library. The project-local libraries are `hardware/components.kicad_sym` and `hardware/4in1ESC-30x30.pretty/`; the project lib tables also reference the shared `Incutec` library from the `libs/KiCad-Library` submodule, used for new parts. Some legacy references (`ESCLibrary`, `PCM_*`, `4in1ESC:`) and standard KiCad library parts resolve only through their embedded copies.

## Build and export

```
git clone --recursive https://github.com/OpenDrone-hw/OpenESC-30x30.git
```

Open `hardware/4in1.kicad_pro` in KiCad 10. Production exports (gerbers, BOM, CPL) are generated with the [KiCad Fabrication Toolkit](https://github.com/bennymeg/Fabrication-Toolkit) plugin. Headless checks and exports use `kicad-cli`:

```
kicad-cli sch erc --exit-code-violations hardware/4in1.kicad_sch
kicad-cli pcb drc --exit-code-violations hardware/4in1.kicad_pcb
kicad-cli pcb export gerbers -o out/ hardware/4in1.kicad_pcb
```

## Manufacturing

Fabricated and assembled at JLCPCB: 6-layer, 1.69 mm board, LCSC parts. Per-revision BOM, CPL, and gerber sets are generated into `hardware/production/` (gitignored) with the Fabrication Toolkit. Two sets come out of this repo:

- `hardware/4in1.kicad_pcb`: this board only, archived as `Rev1-30x30`.
- `hardware/4in1-panel.kicad_pcb`: combined 30x30 + 20x20 panel. The tracked `hardware/fabrication-toolkit-options.json` holds that export's settings, `ARCHIVE_NAME` `Rev1-30x30+20x20`, written out as `Rev1-30x3020x20`.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Links

- Product page: [opendrone.be/products/openesc](https://opendrone.be/products/openesc)
- Video channel: [JustFPV on YouTube](https://www.youtube.com/@justfpv1432)

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt). See [LICENSE](LICENSE), [licensing/README.md](licensing/README.md), [licensing/THIRD_PARTY.md](licensing/THIRD_PARTY.md), and [licensing/TRADEMARKS.md](licensing/TRADEMARKS.md). Some bundled 3D model assets carry their own upstream notices (CC-BY-SA-4.0 / GPL).
