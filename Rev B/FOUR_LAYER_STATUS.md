# Four-layer engineering status

The active design direction is **four copper layers**, as requested. Nothing in
this folder is ready for fabrication or contest submission. Original EasyEDA
inputs and two-layer studies remain intact.

## Current placement

`design/four-layer/power-locality-4layer.kicad_pcb` is a 241-component,
34 x 38 mm, nominal 0.8 mm board. It has components on both sides, the manufacturer's
USB edge recesses and rounded DC-jack notch reliefs. The assembled bounding box
is larger than the bare-board outline and has not been validated.

| Layer | Intended use |
|---|---|
| F.Cu | Components, signals and local power |
| In1.Cu | Ground plane; no signal tracks |
| In2.Cu | Additional routing and power; regional ground/thermal review pending |
| B.Cu | Components, signals and local power |

The final supplier dielectric stack and USB impedance geometry are not selected.
The routing study uses 0.30 mm via pads / 0.15 mm drills, 0.075 mm annuli,
0.10 mm signal traces and 0.125 mm clearance. JLCPCB's published
[capabilities](https://jlcpcb.com/capabilities/Capabilities) list the small through
drill process. Uniform signal-width traces are **not approved power paths**.

## Changes and checks completed

- Restored all 17 bulk-capacitor packages to the original BOM sizes: thirteen
  0805 10 uF / 25 V parts and C29/C33/C34/C37 0402 10 uF / 10 V parts.
- Retained the 201-ball STM32 candidate, original GPIO identities, separate
  OPA197 channels and QS5K2 dual MOSFETs. The quad amplifier and UT6K3 studies
  are not used in this four-layer candidate.
- Used the Omron B3U-1000P pushbutton candidate and manufacturer's 0.30 mm
  USB rear-row lands.
- Component net-membership and changed terminal-role checks pass against the
  official source. These are not complete schematic/ERC or functional checks.
- Added 20 explicit power-stage placement constraints. Input bypass C2/C5 are
  now 1.152/1.150 mm from VIN pads; bootstrap C4/C9 are 1.088/1.132 mm from BST
  pads. All twenty constrained parts are on the same side as their targets.
  Distances are direct pad distances, not routed loop lengths.
- The current unrouted placement has no reported copper-clearance, board-edge,
  hole-clearance or courtyard violations. It still has 169 silkscreen/library
  findings and 499 unconnected items. Report:
  `work/four-layer/power-locality-placement-drc.json`.
- Prepared 50 BGA escape dogbones. Eight planned escapes remain obstructed.
  Escape copper/hole checks pass, but 49 dangling-via warnings are expected
  until their second-layer connections are routed. This is a routing input,
  not a completed board.
- Corrected the router's effective edge margin: an inward routing envelope is
  proved to lie at least 0.08 mm inside the actual board. The fabrication outline
  is unchanged. Its polygon was reduced from 252 to 80 vertices for routing.

## Measured routing studies

| Revision | Missing connections | Vias | Result |
|---|---:|---:|---|
| Source bulk capacitors, before power-locality correction | 275 | 123 | Incomplete; 21 router-created edge errors |
| Power locality plus conservative routing edge | 291 | 82 | No copper, edge, hole or short violations; incomplete |
| Power locality plus explicit BGA escapes | 291 | 124 | No completion improvement; rejected in favor of fewer vias |
| Finer signal rules, three routing passes | 260 | 96 | One router-created short; repaired in next revision |
| Finer rules, ground stitches and short repair | 246 | 109 | Copper/edge/hole/short checks pass; incomplete |

The latest checked candidate is
`design/four-layer/fine-ground-safe-repaired.kicad_pcb`. Its DRC report,
`work/four-layer/fine-ground-safe-repaired-drc.json`, has 246 missing connections,
169 silk/library findings and four dangling-trace warnings. A longer 20-pass
routing job has been started from this exact board. Completion is not implied.
The background review script will import its session and run KiCad DRC after
the router exits. Live/completion status is recorded in
`work/four-layer/long-route-status.json`; detailed output is in
`long-route-review.stdout.log` and `long-route-review.stderr.log` in that folder.
The script does not release, submit, order or automatically label the result
fabrication-ready.

The finer-rule branch uses 0.09 mm signal width / 0.10 mm clearance, supported by
JLCPCB's published 1 oz multilayer process. Its 0.35 mm via pads and 0.15 mm drills
give a 0.10 mm annulus; with the signal clearance this retains 0.20 mm
via-hole-to-copper spacing. Power and USB pair widths still require separate
engineering. The route boundary is inset by at least 0.105 mm for these rules.

Thirteen ground stitches were checked against copper on **all four layers**.
The first stitching draft omitted mixed-layer tracks in its geometric filter
and failed DRC; that draft and `fine-ground-repaired` are superseded. The
`fine-ground-safe-repaired` revision passes the relevant copper checks. No DRC
exclusions or weakened hole/edge rules were used to conceal these findings.

The earlier two-layer 261-connection result is not directly comparable: it used
a different placement, unqualified smaller bulk capacitors, and an incomplete
connector outline. None of these studies has a valid contest score.

## Source inconsistency awaiting resolution

The official U1/U2 symbol value is AP62300T, while the MPN field is AP62300WU-7.
The user's EasyEDA BOM specifies AP62300Z6-7. The ordinary AP62300 has a 0.8 V
reference; the T version has 0.763 V, per the
[Diodes data sheet](https://www.diodes.com/datasheet/download/AP62300.pdf).

For U1, R4/R5/R6 are three parallel 100 kohm resistors and R3 is 10 kohm.
The ordinary part therefore gives 0.8 x (1 + 100/3/10) = **3.4667 V nominal**,
despite the +3V3 label. The T part would give 3.3063 V nominal. U2's 52.3 kohm /
10 kohm divider gives 4.984 V with the ordinary part. This is a source conflict,
not a newly authorized voltage change. The user has been asked whether to target
nominal 3.3 V or preserve the original BOM/resistor values. No divider changes
have been applied.

Do not substitute the smaller SOT563 solely on equal current ratings: the data
sheet gives 110 C/W versus 70 C/W junction-to-ambient for TSOT26 under its test
conditions. The present candidate retains the original TSOT26 package. Its data
sheet recommends ground on both internal layers for best thermal performance;
the mixed third layer needs a specific thermal-ground layout before release.

## Score and unfinished work

Four layers alone cost 20,000 points, so a total below 20,000 is impossible.
At this outline and the jack model's approximately 9.1 mm vertical span,
34 x 38 x 9.1 + 20,000 = **31,757.2 before vias and other overhangs**. This is a
size-budget estimate, not an assembled-volume measurement or a contest score.
It does not beat the supplied leader's 26,362.

Required work: complete routing; engineered power copper and regulator loops;
MCU/hub bypass and clock review; USB differential impedance/return paths;
qualified remaining small-passive substitutions; source-voltage resolution;
thermal checks; connector mating/SD lid access; synchronized schematics and BOM;
self-contained verified assembly models/STEP; complete DRC/ERC and submission
package. No submission has been made.
