# Routing studies — 3 October 2026

These are incomplete engineering experiments. None is a fabrication release or
a valid scored contest entry. Original EasyEDA inputs have not been overwritten.

| Study | Copper layers | Unconnected links, KiCad | Vias | Result |
|---|---:|---:|---:|---|
| 201-ball MCU, original dense placement, FreeRouting 2.4 | 2 | 332 | 54 | Incomplete |
| Same placement, FreeRouting 1.9 | 2 | 364 | 28 | Incomplete |
| 201-ball MCU with 50 explicit escape dogbones | 2 | 333 | 91 | No useful improvement |
| 2.4 routing plus ground pours on both faces | 2 | 289 | 54 | Ground connectivity improves |
| Ground-pour board, another five routing passes, pours refilled | 2 | 261 | 76 | Best measured completion so far |
| 100-ball MCU, automatic fanout and six routing passes | 2 | 285 | 419 | Rejected: excessive via penalty and incomplete |
| 100-ball MCU, annealed placement, three legacy-router passes | 2 | 304 | 66 | Incomplete; no ground pours yet |
| 238-part compact branch, four ports on one edge, explicit CAN trunks | 2 | 499 (CLI DRC) | 32 | Port buses pass copper/hole DRC; remaining routing in progress |

The best completed trial is
`design/experiments/ground-plane-routed.kicad_pcb`. Its DRC report is
`work/ground-plane-routed-drc.json`. Besides the 261 missing links, it has one
copper-to-edge violation, one dangling trace and silk/courtyard/library issues.
There are no reported copper shorts or copper-to-copper clearance violations.
That does not validate power delivery, impedance, thermal performance or assembly.

## Isolated smaller-MCU branch

`design/bga100/candidate-placed.kicad_pcb` retains 241 components and a 34 × 37 mm
outline. It uses STM32H725VGH6, an 8 × 8 mm, 0.8 mm-pitch TFBGA100, and an Omron
B3U-1000P push button. All 10 µF capacitors retain the original 0805-size
construction pending effective-capacitance qualification. USB4081 rear-row pads
are normalized to the manufacturer's 0.30 mm land width.

The branch passes component-net membership and changed terminal-role checks.
Its placement has no packing-envelope overlaps. KiCad still reports ten
courtyard issues that require review. Those statements are not functional signoff.

Firmware changes are explicitly allowed by the current
[competition README](https://github.com/commaai/PCBGolf). This branch needs the
following assignments; **firmware changes have not been implemented or tested**.

| Original GPIO | Replacement | BGA ball | Required peripheral/function |
|---|---|---|---|
| PF7 | PA2 | H3 | ADC1/2 channel 14 |
| PF8 | PA3 | G5 | ADC1/2 channel 15 |
| PF9 | PA7 | K3 | ADC1/2 channel 7 |
| PF10 | PB0 | K4 | ADC1/2 channel 9 |
| PF11 | PC5 | J4 | ADC1/2 channel 8, CH3 current monitor |
| PG9 | PD13 | H10 | FDCAN3 TX, AF5 |
| PG10 | PD12 | H9 | FDCAN3 RX, AF5 |
| PD12 | PE5 | A2 | CH3 SBU1 ignition GPIO |
| PD13 | PE6 | A1 | CH3 SBU2 relay GPIO |
| PE15 | PC13 | E4 | Button input; BOOT0 remains connected |

Pin functions are from [ST DS13311 Rev 5, tables 8 and 9](https://www.st.com/resource/en/datasheet/stm32h725ig.pdf).
ADC remapping must preserve sampling behavior and voltage scaling, including the
change from ADC3 for PF7–PF10. TFBGA100 has VDD33USB but no VDD50USB terminal;
VDD33USB remains on the existing 3.3 V rail. USB supply initialization requires
review. All unchanged interfaces retain their existing GPIO identities.

The complete machine-readable mapping is `work/bga100/gpio-remapping.json`.

## Structured routing study

`design/compact/can-trunks.kicad_pcb` is a separate 50 × 27 mm, 238-part placement.
It consolidates four OPA197 channels into one OPA4197IPWR and studies UT6K3TCR dual
MOSFETs. These substitutions save 61.792 mm² of summed component packing-envelope
area versus the BGA100 branch. They are **not fully qualified substitutions**;
see `work/compact/substitution-review.md` for the high-temperature bias-current,
gate-protection and thermal questions. Changed terminal mappings pass their audit.

All eight shared CAN signals now join the four OBD-C ports on straight back-layer
trunks with 32 through vias. The latest `work/compact/can-trunks-drc.json` has no
copper clearance, shorting, hole-clearance or solder-mask-bridge errors. It still
has 499 unconnected items, 13 courtyard issues and silk/library issues. Connections
to the CAN transceivers, activity-monitor inputs and header are still required.
Reserved routing regions may overlap component courtyards where actual copper
clears the pads; those intersections are separately recorded by the packer.

This longer board is a routing comparison, not the chosen final outline. It is
larger in area than 34 × 37 mm. No valid assembled-volume or contest score exists.

## Preserving power limits

The source MPN is TPS25944ARVCR, despite the TPS25942 symbol label. The candidate
retains TPS25944A auto-retry circuit-breaker behavior and all four original
28 kΩ current-setting resistors. [TI's data sheet, equation 4](https://www.ti.com/lit/ds/symlink/tps25944a.pdf)
gives 89 / 28 = 3.18 A nominal per-port fault threshold. Tolerance, the shared
input rating and thermal limits prevent treating that as four simultaneous
continuous 3.18 A loads. The four original 33 kΩ IMON resistors remain unchanged.

Uniform 0.10 mm tracks in autorouter studies are not approved power routing.
Input distribution, output paths, protection-device copper and regulator loops
must be engineered before release.

## Manufacturing and mechanical work still required

The 100-ball study uses 0.25 mm via pads and 0.10 mm holes, retaining a 0.075 mm
annulus. [JLCPCB lists 0.10 mm holes for two-layer boards](https://jlcpcb.com/capabilities/Capabilities)
at thickness ≤1 mm with ENIG/OSP. The study is 0.8 mm and requires that process.
Via count is still part of the contest penalty; these smaller holes do not
reduce it. Power vias require separate current/temperature sizing.

Outstanding: connector mating envelopes and header engagement, USB edge recesses,
rounded jack notch, correct model seating and self-contained model paths,
complete assembly STEP, power and USB routing, local bypass/clock layout,
thermal checks, synchronized schematics/BOM and final DRC/ERC.

The current header is not yet qualified as a low-height replacement. A shorter
pin header cannot be assumed compatible solely because it has the same pitch.
No score is calculated from an unverified header or incomplete assembly model.
