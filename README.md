# OpenESC-30x30

Open source 4-in-1 BLDC ESC with a 30.5 x 30.5 mm mounting pattern, part of the
incutec OpenDrone line. Four independent motor controllers, each with its own
MCU, gate driver and six MOSFETs, running AM32 and taking DShot over the
standard 8-pin connector.

<p>
<img src="images/front.png" width="400" alt="OpenESC-30x30 top" />
<img src="images/back.png" width="400" alt="OpenESC-30x30 bottom" />
</p>

[![Status](https://img.shields.io/badge/status-alpha-e08c00)](https://github.com/OpenDrone-hw/.github/blob/main/CONTRIBUTING.md#the-life-of-a-project)
[![Shop](https://img.shields.io/badge/shop-opendrone.be-c89d2e)](https://opendrone.be/products/openesc)
[![Discord](https://img.shields.io/badge/Discord-%23esc-5865F2?logo=discord&logoColor=white)](https://discord.com/channels/1494019459822653512/1494782966302507118)
[![Video](https://img.shields.io/badge/YouTube-How%20Drone%20ESCs%20Work-FF0000?logo=youtube&logoColor=white)](https://www.youtube.com/watch?v=TwAmmPxOpTM)
[![OSHWA](https://img.shields.io/badge/OSHWA-BE000029-0099b0)](https://certification.oshwa.org/be000029.html)

## Specifications

| | |
|---|---|
| Firmware | AM32 |
| ESC protocol | DShot, bidirectional |
| Telemetry | Extended DShot |
| Input | 3-8S LiPo |
| BEC | None |
| MCU | One per motor |
| MOSFETs | 6 per motor |
| Current sense | On-board, 330 A |
| TVS protection | None |
| FC connector | JST-SH 8-pin |
| Mounting | 30.5 x 30.5 mm, 4.0 mm holes |
| PCB | 6-layer, 2 oz copper |

Technical write-up, part list and layout constraints: [AGENTS.md](AGENTS.md).

## In the line

What pairs with what, and what is available:
[opendrone.be](https://opendrone.be).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).
