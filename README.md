# OpenESC-30x30

Open source 4-in-1 BLDC ESC with a 30.5 x 30.5 mm mounting pattern, part of the
incutec OpenDrone line. Four independent motor controllers, each with its own
MCU, gate driver and six MOSFETs, running AM32 and taking DShot over the
standard 8-pin connector.

<p>
<img src="images/front.png" width="400" alt="OpenESC-30x30 top" />
<img src="images/back.png" width="400" alt="OpenESC-30x30 bottom" />
</p>

|  |  |
|---|---|
| Size | 30.5 x 30.5 mm mounting pattern, 4x M3 |
| Input | 3S-8S |
| Interface | 8-pin JST SH, DShot |
| Firmware | [AM32](https://github.com/am32-firmware/AM32) |

<a href="https://certification.oshwa.org/be000029.html">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/oshwa-certified-dark.svg" />
    <img src="images/oshwa-certified.svg" width="160" alt="OSHWA Certified Open Source Hardware, BE000029" />
  </picture>
</a>

Certified open source hardware by the [Open Source Hardware Association](https://www.oshwa.org/),
OSHWA UID **[BE000029](https://certification.oshwa.org/be000029.html)**.

## In the line

- [OpenESC-20x20](https://github.com/OpenDrone-hw/OpenESC-20x20): the same
  design at 20 x 20 mm, for smaller builds. The two mirror each other and
  differ in board size and a few power-stage parts.
- [OpenFC-Lite](https://github.com/OpenDrone-hw/OpenFC-Lite): the 30.5 x 30.5 mm
  flight controller this stacks with. It has no onboard motor drivers, so it
  needs an ESC like this one.

## Get one

[opendrone.be/products/openesc](https://opendrone.be/products/openesc)

Build videos and teardowns: [JustFPV](https://www.youtube.com/@justfpv1432)

## Contributing

Issues and pull requests are welcome on any repo. KiCad files cannot be merged,
so say what you intend to change before you do, on
[Discord](https://discord.gg/v3sWmTcx3R).

The design itself, the part list and the layout constraints are in
[AGENTS.md](AGENTS.md). How everything works:
[CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).

Two bundled 3D models carry their own upstream licences in a notice embedded in
the file, and those still apply: `IND-SMD_L1.6-W0.8_FTC160865SR47MBCA.step`
is CC-BY-SA-4.0 and `QFN-28_L4.0-W4.0-P0.40-TL-EP2.4.step` is GPL, both in
`hardware/4in1ESC-30x30.3dshapes/`.
