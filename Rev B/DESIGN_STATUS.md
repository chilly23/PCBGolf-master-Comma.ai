# PCBGolf compact-board development

Status: experimental layout; NOT ready to order or submit.

The supplied EasyEDA Pro archive, BOM, and PDF remain unchanged. The competition's
official KiCad source is retained under work/upstream. The candidate is based on
that connectivity and has explicit package pin mappings under work/pinmap.json.

The working placement has 241 components, two copper layers, double-sided
assembly, a 34 x 37 mm board outline, and a nominal 0.8 mm laminate. This is not
the assembled bounding box and is not a contest score. Five USB-C receptacles,
four separately controlled/monitored 12 V device outputs, four CAN transceivers,
ignition circuits, USB hub, MCU, microSD, button and indicators are retained.

## Findings in the supplied EasyEDA variant

- Several 10 uF capacitors selected in 0201/0402 packages have 4 V or 6.3 V
  ratings while their nets reach 12 V. Those substitutions cannot be retained.
- The official symbol says TPS25942 but its manufacturer field specifies
  TPS25944ARVCR. The candidate now follows that manufacturer part number and
  preserves its circuit-breaker behavior, with all power contacts connected.
- The power inductors must be rated power inductors, not small signal inductors
  with a matching nominal inductance. Original MEKK2520T3R3M parts are 1.8 A.
- The original KiCad MCU is available in a smaller package with the same used
  GPIO names. The current candidate uses STM32H725IGK3 (10 x 10 mm BGA).
- The locally modified board had NCS20071 labels/pin numbers while the official
  reference uses OPA197IDR. The candidate now uses the same OPA197 silicon in
  SOT-23-5 (OPA197IDBVR), with its SOIC-to-SOT pin mapping explicitly checked.
- Eight single-color LEDs used lettered A/C pads in the official source.
  The smaller footprints use 1=K, 2=A; the mapping is now checked explicitly.
- Imported USB board-edge reference lines and switch body artwork were marked
  Edge.Cuts. The placed candidate moves this construction artwork off the cut layer.

## Release blockers

1. Complete routing with no shorts, missing connections, or manufacturing errors.
2. Engineered power widths/planes, eFuse and MCU thermal paths, regulator loops,
   local decoupling, USB differential-pair geometry and continuous return paths.
3. Qualify every smaller part's voltage/current/power/timing behavior. Buck
   identifiers now follow the official AP62300WU-7 manufacturer field (0.8 V
   reference), rather than the inconsistent AP62300T symbol value.
4. Confirm DC-jack mating fit and mid-mount plane; finalize rounded notch and
   USB board-edge setbacks; check all mating cable/access envelopes.
5. Complete, dimensionally accurate models for all components and header;
   collision-check the assembly and measure its bounding box from final STEP.
6. Synchronize the editable KiCad schematic, PCB, libraries and final BOM.
7. Calculate the final score only from validated final dimensions and via count.

The first router pass is a routing-feasibility experiment. Uniform thin tracks
in that experiment are not approved power routing, and no score from it is valid.
There is no guarantee that any score will remain first on the leaderboard.

## Reproducible working files

- work/prepare_candidate.py: package/pin-map construction from source board.
- work/custom_footprints.py: dimensioned candidate DC-jack footprint.
- work/pack_candidate.py: constrained two-face packing and preview generation.
- work/apply_placement.py: KiCad placement and router interchange export.
- design/placement-preview.svg: labeled placement preview.
- design/candidate-placed.kicad_pcb: unrouted engineering candidate.
- work/placed-drc-v2.json: initial placement DRC (no copper shorts/clearance errors;
  remaining courtyard, silk and library issues are not waived for production).
- work/connectivity-validation.json: official-reference component net membership
  and changed terminal-role audit. Earlier placement/router experiments predate
  the op-amp/LED correction and must not be used as electrically valid boards.

## References

- Competition: https://github.com/commaai/PCBGolf
- Fabrication: https://jlcpcb.com/capabilities/Capabilities
- MCU: https://www.st.com/resource/en/datasheet/stm32h725ig.pdf
- DC jack: https://cloud.kycon.com/Pub_Eng_Draw/KLDHCX-MM-0202-x.pdf
- USB receptacle: https://gct.co/connector/usb4081
- Bicolor LED: https://docs.broadcom.com/doc/AV02-0584EN
- Buck converter: https://www.diodes.com/datasheet/download/AP62300.pdf
