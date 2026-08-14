# 30x30-ESC-QC: press-contact QC fixture

Bench test/QC jig for the **OpenESC-30x30** (`../hardware/4in1`). Same fixture as
`OpenESC-20x20/20x20-ESC-QC`, rebuilt against the 30x30 pad geometry.

The board is a **negative** of the ESC contact face. The ESC drops into an
ESC-shaped pocket cut through the board, and its protruding edge pads land on
contact features around the rim. Power, three phases per channel and the signal
row fan out to solder pads at the board edge.

## Layout / conventions (mirrors the 20x20)

- KiCad 10 project `30x30-ESC-QC.kicad_{pro,sch,pcb}`, 100 x 100 mm, 4 layer.
  The schematic is an empty stub: the fixture is laid out on the board, no netlist.
- Project-local libraries in `ESC-QC.pretty/`:
  - `4in1ESC30x30-negative`, 34 contact pads plus all the board geometry:
    the 100 x 100 outline, four M3 corner holes and the ESC pocket, all on Edge.Cuts.
  - `TP_Pad_{2.0,3.0,4.0}x…mm`, the edge solder lands. Generated locally rather
    than pulled from KiCad's global `TestPoint` library, so the repo stays
    self-contained.
- 44 edge pads, F.Cu and B.Cu at the same spots: 6 per side for the phases
  (3.0 mm), 8 across the top for the signal row (2.0 mm), 2 at the bottom for
  the battery (4.0 mm). Positions are copied from the 20x20 board, which works
  because the fixture is 100 x 100 either way.
- **Nothing is routed.** Placement, pads, pocket and outline only.

## Generated, not drawn

Both the negative and the board come out of
`~/OpenDrone/software/OpenDrone-Scripts/esc/esc_qc_gen.py`, which reads the ESC design read-only,
so the contact geometry cannot drift from the board it tests:

```bash
KPY=/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3
$KPY ~/OpenDrone/software/OpenDrone-Scripts/esc/esc_qc_gen.py ../hardware/4in1.kicad_pcb \
    --dut 4in1ESC30x30 --lib ESC-QC.pretty --out 30x30-ESC-QC.kicad_pcb
```

Regenerating overwrites the board, so route last. Run against
`../../OpenESC-20x20/hardware/4in1-mini.kicad_pcb --dut 4in1ESC` and it
reproduces the 20x20 fixture, pocket included, to within 0.05 mm.

## Reference geometry

Taken read-only from `../hardware/4in1.kicad_pcb`; nothing in the ESC design is
modified. ESC outline 41.58 x 42.50 mm. Battery pads 4.5 x 15 mm. Phase pads
2.2 x 3.6 mm, two per phase. Signal row 8 x (4 x 1 mm) on 1.5 mm pitch.
Mounting 4.0 mm drill on 30.5 mm pitch. Pocket comes out 37.06 x 38.15 mm with a
3.8 mm corner radius, sized so the body drops in and the edge pads sit on the rim.

## Contacts

- **+BATT / GND**: high-current press contacts, the ones that matter most.
- **Motor phases**: 3 per channel x 4 channels.
- **Signal**: /M1-/M4 plus +BATT, GND and /CURR from the 8-pin row.
- SWD is not on this board. It is on `../30x30-ESC-Flashing`, because the SWD
  test points are on the ESC's other face.

License: hardware CERN-OHL-S-2.0, same as the parent repo.
