> Translated from the Chinese version; if they differ, the Chinese version prevails.

# Hubo-Tack PCB Layout General Specification (SPEC-005 Public Edition V0.12)

> **Basis:** Chapter 10 sets the basic routing rules for all traces (corners, trace width and spacing, vias, pad exits, power trace width). The other chapters set routing, via and spacing requirements for USB, SPI, MIPI, SDIO, clocks and RF by signal grade. Additional chip-vendor requirements are listed in the appendices, per chip. Values are taken from chip-vendor hardware design guides, PCB-fabricator tutorials and industry application notes (searched and checked against the originals on 2026-10-08); where sources disagree, the choice made is explained in Appendix C.
>
> This is the public edition of the internal specification SPEC-005 V0.12 (draft). Section and clause numbers are kept unchanged; appendices outside the public scope and internal-process clauses are not part of the public edition, and their numbers are marked "(Not included in this public edition)". This specification is still a draft, and values may be revised after measurement and review.

## Revision history

(Not included in this public edition)

---

## 1. Scope and signal grades

### 1.1 Scope

This specification applies to Hubo-Tack related PCBs (mainboards, 30-Pin backpacks and so on), regardless of EDA tool. Hubo-Tack 30-Pin backpacks must also satisfy [SPEC-003](./30pin-backpack-spec.md); where the two conflict, SPEC-003 prevails.

### 1.2 Signal grades

Rules are determined by signal speed and edge rate, independent of chip brand. **The basic routing rules of Chapter 10 apply to all nets**; each board's design document must also list the **grades involved on that board**, and the corresponding chapters are applied per the table.

| Grade | Typical signals | Chapters to apply |
|---|---|---|
| L Low speed | I2C, UART, GPIO, LED data, SPI ≤ 20 MHz | Chapter 10 only |
| U1 USB full speed | USB 2.0 full speed 12 Mbps (e.g. S3 USB-OTG full speed) | 2, 3.1 |
| U2 USB high speed | USB 2.0 high speed 480 Mbps (e.g. the ESP32-P4 USB high-speed port) | 2, 3.2 |
| S High-speed SPI | SPI/QSPI/OPI > 20 MHz, including Flash and PSRAM | 2, 4 |
| M MIPI | MIPI CSI/DSI | 2, 5 |
| D SDIO | SDIO, SD card 4-wire | 2, 6 |
| C Clock | Passive crystals, active oscillator outputs | 7 |
| R RF | Antenna feed lines, RF module outputs | 8 |

The 20 MHz boundary is a value chosen by this specification (below this frequency the trace length is negligible relative to the rise time, following the method in Altium's "Is There an SPI Trace Impedance Requirement?"); it is not an industry standard.

### 1.3 Additional chip requirements

Where the chip vendor's hardware design guide is stricter than this specification, the chip vendor prevails. Chips already compiled are in Appendices A, B and D; **before using a chip that has not been compiled, read that chip's hardware design guide and add an appendix first**; routing must not start before the appendix is added.

---

## 2. General rules (high-speed grades other than U1)

| Clause | Requirement | Source |
|---|---|---|
| 2.1 Reference plane | The layer directly below a high-speed line is a complete GND; do not cross split gaps or plane boundaries | Espressif ESP32-P4/S3 hardware design guides (USB, MIPI, SDIO sections); SPEC-003 6.2.3 |
| 2.2 Return path on layer change | Avoid layer changes; when unavoidable, put a return ground via **≤ 1 mm** from each signal via, and a pair of them for a differential pair | Espressif ("add a pair of ground vias for return at every layer-change via"); SPEC-003 6.2.5 |
| 2.3 Differential via symmetry | The two lines of a differential pair have the same number of vias in symmetric positions | TI SPRAAR7J; JLC Wiki |
| 2.4 Corners | No 90° corners; use 45° (135° interior angle) or arcs | Espressif RF section; JLC; Altium |
| 2.5 Spacing | Center-to-center distance between a high-speed line and other signals **≥ 3W** (W = trace width), 5W recommended; do not run parallel to clocks or switching-supply traces over long distances | JLC 3W; Espressif MIPI section 3W; TI 5W. See Appendix C for the choice |
| 2.6 Test points | No test points and no stubs on differential lines of grades U2 and M | TI SPRAAR7J (0 test points on high-speed differential lines) |
| 2.7 Length-matching serpentines | Add serpentines only to the shorter line, close to the source of the mismatch (vias, bends, pins); serpentine segment spacing ≥ 3W | TI SPRAAR7J; JLCPCB blog |
| 2.8 Via stubs | **Grades ≤ 480 Mbps (U1, U2, L, S, D)**: 1.6 mm through-hole vias are fine; stubs are not limited and back-drilling is not needed. **Gbps-class (such as M MIPI)**: keep the whole pair on one layer; on a 4-layer board there is then no stub. When a layer change is necessary, the stub must be **≤ 15 mil**; anything longer needs back-drilling or routing on the other side | TI SPRAAR7J (the 15 mil limit applies to Gbps-class signals) |
| 2.9 Guard-ground vias | Guard copper of high-speed differential lines (ground copper on both sides and between the pair) must be tied to the reference ground plane **along its whole length** with ground vias; **tying it at the two ends only is forbidden**; narrow guard copper that cannot hold ground vias must be removed and ≥ 3W spacing used instead. Via spacing is taken as λ/20 (λ calculated at 2.5 times the bit rate, FR-4 effective dielectric constant 3.5–4.1), and no more than λ/10 where space is limited. Common values: MIPI (≤ 1 Gbps/lane) **≤ 3 mm**; HDMI TMDS (≤ 1080p) **≤ 4 mm**; USB 2.0 HS **≤ 6 mm**; for higher rates recompute with the formula and write the result into the design document. Chip fan-out areas, connector areas and ESD pad areas may be exempted, with the exempted segments listed in the design document | EDN Hageman "Via Spacing on High-Performance PCBs" (λ/20, citing Ott and Vizmuller); ProtoExpress, AtlasPCB (λ/20 to λ/10); Bogatin & Simonovich "Guard Traces: Love Them or Leave Them?" (grounding at the two ends only forms a resonant cavity and increases crosstalk); Panox MIPI DSI guide (≤ 200 mil, single source). Chip-vendor (TI, Renesas, Espressif) documents give no such value; see Appendix C for the choice |
| 2.10 Connector pin assignment | Differential pairs on a connector are arranged "**GND — pair — GND**", with one ground pin on each side of every pair; the fan-out from chip to connector uses smooth arcs, keeps P/N symmetric, and keeps the uncoupled P/N segment as short as possible. The "octopus" fan-out on a 2.54 mm header (P to one row, N to the other row, ground pins between pairs) is for debug headers only and is not recommended in production; no data was found on the effect of headers on impedance and stubs, and the risk for HDMI is high | Samtec FAQ (signal:ground 1:1 is usually best; ground shielding between pairs helps); Raspberry Pi CSI 22-pin connector has one ground between each pair (pin definitions compiled by Arducam, secondary). Recommendation level; FPC connector pin definitions follow the display or module vendor |
| 2.11 Low-speed signals on inner layers | The power layer of a 4-layer board (L3, including mixed layers of "power blocks + GND fill") **may carry a few low-speed signals** (UART, I2C, GPIO, SPI ≤ 8 MHz), provided all of the following hold: ① there is a complete GND under RF and crystals; ② the reference layers adjacent to the signal are complete and **do not cross split gaps**, and where a crossing is unavoidable a stitching capacitor is added at the crossing; ③ there is ground copper on both sides of the signal with ground vias along it; ④ record the signal's **actual nearest reference layer** (if the L3–L4 spacing is smaller than the L3–L2 spacing, the nearest is L4). **High-speed signals (grades ≥ 480 Mbps) do not use the power layer and do not reference a power plane**; where unavoidable, add stitching capacitors. Below sensitive modules such as GNSS, the top layer and the second layer must be ground-filled with no signal routing | Espressif ESP hardware design guide, PCB layout chapter (L3 "route power traces and a few signal traces ... provided that there is a complete GND plane under the RF and crystal"; ESP32-S31 "Signal traces are allowed"; UART "surrounded by ground copper and ground vias"); TI SLLA414 §3.4, SCAA082A §2.3 (do not cross splits; add stitching capacitors where crossing is unavoidable); ST AN4488 §8.4 (a PWR reference needs 10 nF stitching capacitors); u-blox MAX-M10S Integration Manual §4.4 (fill the top and second layers below the module with ground and route no signals); Altium blog (corroboration: UART may cross a split, the PWR layer is not suited as a general signal layer) |

**The 20H rule (pulling the power plane in by 20 times the dielectric thickness) is not required.** Its EMI suppression effect is unstable on modern multilayer boards (IEEE "Effects of 20-H Rule and Shielding Vias on EMI Radiation"; Clemson CVEL research). Board-edge GND stitching vias follow SPEC-003 6.2.7 and have nothing to do with 20H.

---

## 3. USB 2.0

### 3.1 Full speed (U1)

| Clause | Requirement | Source |
|---|---|---|
| 3.1.1 Routing | Route D+/D− as a parallel differential pair, as short as possible; **impedance control and length matching are not required** | Value chosen by this specification: edges at 12 Mbps are slow and short on-board lines have negligible effect (no primary source found) |
| 3.1.2 Protection | Place ESD/TVS devices close to the connector | JLC Wiki "USB module"; TI SPRAAR7J |

### 3.2 High speed (U2)

| Clause | Requirement | Source |
|---|---|---|
| 3.2.1 Impedance | 90 Ω differential ±10%; trace width and spacing per SPEC-003 6.5 | Espressif P4/S3; JLC |
| 3.2.2 Components | Resistors and capacitors reserved on the line are placed close to the chip; ESD/TVS close to the connector | Espressif; JLC "USB module"; TI |
| 3.2.3 Guard ground | Guard the differential pair with ground on both sides | Espressif |
| 3.2.4 Layer change | Avoid layer changes; **≤ 2 vias per line**; each layer change gets a pair of ground vias per 2.2 | Espressif (minimize); TI (USB 2.0 ≤ 4 per line). ≤ 2 is a value chosen by this specification |
| 3.2.5 Intra-pair matching | Intra-pair length difference **≤ 5 mil** (0.127 mm) | JLC "USB module", "Understanding impedance matching" |
| 3.2.6 Length | Total length from chip to connector **recommended ≤ 1800 mil (45.7 mm)**; beyond 4000 mil (101.6 mm) the reason must be stated in the design document | JLC 1800 mil; TI typical 4000 mil, maximum 12000 mil |
| 3.2.7 Test points | No test points (2.6) | TI |
| 3.2.8 Edge distance | Recommended ≥ 90 mil (2.29 mm) from the board edge and from the edge of copper pours | TI SPRAAR7J (differential vias ≥ 90 mil from the edge of the reference plane); secondary material. Single source, unverified, not a hard requirement |

---

## 4. High-speed SPI (S)

| Clause | Requirement | Source |
|---|---|---|
| 4.1 Series resistor | Series resistors on SPI lines (default 0 Ω) are placed close to the **driver end** (main-controller side) | Espressif S3; Infineon AN240090; AMD UG863; Altium |
| 4.2 Guard ground | Guard clock lines and data lines separately | Espressif S3 |
| 4.3 Topology | Point-to-point; for one-to-many use a star and explain it in the design document; do not use simple branches | Infineon AN240090. Single source |
| 4.4 Length | As short as possible; Flash and PSRAM right next to the main controller | Espressif S3 |
| 4.5 Length matching | Octal SPI must be length-matched. Espressif gives no values; **suggested** ±30 mil between data lines and ±200 mil for CLK to data | Espressif S3 (length matching required); Infineon AN240090 (133 MHz case). Values single-source, for reference |

---

## 5. MIPI (M)

| Clause | Requirement | Source |
|---|---|---|
| 5.1 Impedance | 100 Ω differential ±10%; trace width and spacing per SPEC-003 6.5 | Espressif P4 |
| 5.2 Length matching | Intra-pair length difference **≤ 10 mil**; between pairs **≤ 30 mil**, using serpentines when needed | Espressif P4 |
| 5.3 Isolation between pairs | Guard ground between pairs; where guarding is impossible, spacing **≥ 2W** | Espressif P4 |
| 5.4 Keep away | Distance **≥ 3W** from other high-speed and high-frequency signals, and never parallel; keep away from switching supplies | Espressif P4 |
| 5.5 Clock | Guard the MIPI CLK pair with ground | Espressif P4 |
| 5.6 Layer change | Per 2.2 | Espressif P4 |

---

## 6. SDIO (D)

| Clause | Requirement | Source |
|---|---|---|
| 6.1 Impedance | 50 Ω single-ended ±10%, per SPEC-003 6.5 | Espressif P4/S3 |
| 6.2 Length matching | CMD and DATA0–3 within **±50 mil** of the CLK length | Espressif P4/S3 |
| 6.3 Length | Total length from chip to connector preferably **≤ 2000 mil** (50.8 mm) | Espressif P4/S3 |
| 6.4 Clock | Guard CLK with ground on both sides | Espressif P4/S3 |
| 6.5 Planes | Do not cross planes; follow 2.1 | Espressif P4/S3 |

---

## 7. Clocks and crystals (C)

| Clause | Requirement | Source |
|---|---|---|
| 7.1 Vias | **No vias** on crystal clock traces | Espressif P4/S3 |
| 7.2 Guard ground | Surround clock traces with ground and dense ground vias; keep power vias on both sides away from the clock line | Espressif P4/S3 |
| 7.3 Component placement | Series components close to the chip; ground tuning capacitors close to both sides of the crystal, not connected directly to the series component | Espressif P4/S3 |
| 7.4 Underneath | No high-frequency signals under the crystal, preferably no signals at all | Espressif P4/S3 |
| 7.5 Ground plane | A complete ground plane between crystal and chip | Espressif P4/S3 |
| 7.6 Magnetic components | No large inductors or other magnetic components around the crystal | Espressif P4/S3 |
| 7.7 Distance | Minimum distance from crystal to the chip clock pin follows the chip appendix (P4 ≥ 4.5 mm, S3 ≥ 2.0 mm) | Espressif |

The crystal vendor ECS advocates "no ground directly under the crystal", which is the opposite of Espressif's "complete ground plane between crystal and chip"; with Espressif chips **Espressif prevails** and ECS's practice is not adopted.

---

## 8. RF (R)

Impedance, matching network, SMA and under-module requirements follow SPEC-003 6.3 and 6.5. In addition:

| Clause | Requirement | Source |
|---|---|---|
| 8.1 Layer and vias | Route RF lines on the surface layer, **no vias or layer changes** | Espressif S3 RF section |
| 8.2 Shape | Uniform width, no branches; 135° or arc corners | Espressif S3 |
| 8.3 Shielding | Dense ground vias on both sides; complete ground plane on the adjacent layer, with no routing below where possible | Espressif S3; SPEC-003 6.2.7 |
| 8.4 Keep away | Keep antennas and RF lines away from crystals, high-speed clocks, USB and UART (including vias, test points and headers) | Espressif S3 |

---

## 9. Checks

KiCad projects are checked with the scripts of the `hubo-kicad` skill in hubo-backpack-skill, and the results are written into the design document:

| Check item | Tool | Clauses |
|---|---|---|
| Differential trace width, spacing, breakout narrowing | `impedance_check.py` | 3.2.1, 5.1 |
| Corners, ground-via distance beside layer-change vias, reference plane, ground fill between pairs, distance to aggressors | `hs_review.py` | 2.1, 2.2, 2.4, 5.3, 5.4 |
| Via count per differential line, per-line via limit, intra-pair length difference, test points | `hs_review.py --max-vias --skew` | 2.3, 2.6, 3.2.4, 3.2.5, 5.2 |
| No vias on given nets, crystal-to-chip clock pin distance | `hs_review.py --no-via --xtal` | 7.1, 7.7, 8.1 |
| Trace width and spacing, pad-to-trace, hole size and annular ring, hole-to-hole, board edge, silkscreen, minimum width of power nets | KiCad DRC (`power_rules.py` generates rules from the project configuration) | 10.2–10.5, 10.8, 10.9 |
| Pads of different components overlapping or spaced < 0.1 mm (including same net) | `route_check.py` item 3 (DRC does not check same-net pad spacing) | 10.12 |
| Ground-via spacing along guard copper | `fence_check.py` | 2.9 |
| Right angles and acute angles over the whole board, via-in-pad | `route_check.py` | 10.1, 10.7 |

The power currents and trace widths in the design document and the board's check parameters are written into the project configuration `<board name>.hz_layout.json`. **When production files are exported, `prod_gate.py` enforces all checks in the table above, and any failure refuses the export**; where a release is truly necessary the reason must be stated in the design document.

Items the scripts cannot judge (component placement order, whether guard ground is in place, topology, reference-layer cutouts under pads, whether a power pour replaces thin traces) are checked manually in the KiCad interface.

---

## 10. Basic routing rules (all nets)

This chapter applies to all nets (including grade L). Process values are taken from the JLCPCB "Process parameters" page (jlc.com/portal/vtechnology.html, original checked on 2026-10-09); "recommended" values are stricter than the process lower limit, and traces below the recommended value must be listed with location and reason in the design document.

| Clause | Requirement | Source |
|---|---|---|
| 10.1 Corners | No 90° right-angle or acute-angle corners; use 45° or arcs; traces must not form acute-angle slots with pads or with other traces. A corner point that falls inside the copper of a same-net pad or via does not count (the corner is covered by copper) | Acute angle: risk of etchant trapping (acid trap), general process knowledge; 90°: a uniform requirement of this specification (high-speed lines have an additional electrical reason, see 2.4) |
| 10.2 Trace width and spacing | **Recommended ≥ 5/5 mil** (0.127 mm); process lower limit 4/4 mil for 1 oz double-sided boards and 3.5/3.5 mil for multilayer boards; below 3.5 mil on multilayer boards a 20% surcharge applies | JLC process parameters; JLC "PCB surcharge list" |
| 10.3 Pad to trace | Pad edge to trace edge ≥ 0.1 mm | JLC process parameters |
| 10.4 Vias | Normally hole 0.3 mm / outer diameter 0.6 mm (value chosen by this specification); hole ≥ 0.3 mm, outer diameter ≥ 0.4 mm and at least 0.15 mm larger than the hole (usual minimum 0.3/0.45 mm); vias with hole < 0.3 mm and outer diameter ≤ 0.4 mm incur a surcharge at JLC (TG155 material plus full four-wire low-resistance testing); where smaller is truly needed, state it in the design document and accept the surcharge; via hole-edge to hole-edge ≥ 0.2 mm; through-hole component holes edge to edge ≥ 0.45 mm; annular ring on multilayer boards ≥ 0.2 mm | JLC process parameters; JLC "PCB surcharge list"; the process lower limit of 0.15/0.25 mm is "non-standard, expensive" |
| 10.5 Board edge | Copper, pads and traces to board outline ≥ 0.3 mm; V-cut panels ≥ 0.4 mm. **With SMT, measure from the component body as well**: pad to board edge preferably **≥ 0.9 mm** (do not push to the limit; reduce only when space is short), body to board edge at least > 0.5 mm; board-edge connectors (USB-C, SMA, buttons and so on) that are designed to overhang the edge are excepted, to be confirmed on the order page. Standard SMT must have process rails (JLC 5–8 mm, minimum 3 mm, added by JLC by default); economy SMT does not need them | JLC routing ≥ 0.2 mm, V-cut ≥ 0.4 mm; SPEC-003 6.4.1 takes 0.3 mm. SMT body to board edge: JLC "SMT assembly quality responsibility statement" ("the body edge of components must be more than 0.5 mm from the board edge") (m.jlc.com/portal/m7i643.html, **search snippet only, original page not read, single source**); JLC "Process rail purpose and suggestions" recommends components ≥ 5 mm from the board edge and suggests process rails below 5 mm (jlc.com/portal/q7i49004.html, snippet); JLCPCB English site Assembly T&C (2026-09-08) "body of the components and the edge of the board must be equal or greater than 2.5mm"; "Process rail design specification" jlc-smt.com/technology/detail/62681.html (2026-08-14). The relationship between the three figures is not explained officially; the order page prevails |
| 10.6 Pad exits | Exit trace width no greater than the pad width, leaving from the pad edge; both pads of 0402 and smaller have the same exit width; pads connected to large copper areas use thermal-relief pads | Engineering practice: unbalanced heat dissipation at the two ends causes tombstoning and cold joints (no primary source found) |
| 10.7 Via-in-pad | No vias on SMD pads, and **signal vias are never allowed on pads**; where a via is necessary, choose "via-in-pad" on the order (resin-filled and plated over) and explain it in the design document. **Exception:** thermal/ground vias on a chip's bottom thermal pad that are on the same net as the pad and have a hole ≤ 0.3 mm may be left unfilled and need not be ordered with the via-in-pad process; preferably include them in the footprint and keep stencil openings clear of the holes | JLC process parameters (via-in-pad is a separate process); unfilled vias wick solder away; TI SLUA271C "QFN and SON PCB Attachment" (the smaller the hole, the less solder is wicked; unfilled vias still allow repeatable soldering) |
| 10.8 Power trace width | The design document lists the current **actually carried by the trace** of each power net (when the main path is a pour and the trace is only a branch, use the branch current); trace width is calculated per **IPC-2221**, temperature rise **10 °C**, outer 1 oz / inner 0.5 oz (reference: outer layer 1 A about 0.30 mm, 2 A about 0.78 mm; inner layer about 5 times wider for the same current); the narrowest point of the net over its whole length must not be smaller than this (via count is likewise checked against the current); the current is written into the project configuration, and scripts calculate it and generate DRC rules enforced per net | KiCad 10 Calculator Tools documentation (trace width uses the IPC-2221 formula); 10 °C rise is a value chosen by this specification |
| 10.9 Silkscreen | Text height ≥ 1.0 mm, line width ≥ 0.15 mm, ≥ 0.15 mm from exposed copper pads | SPEC-003 6.4.2; JLC process parameters (height ≥ 1 mm, stroke ≥ 0.1 mm, to pad ≥ 0.15 mm) |
| 10.10 Teardrops | Teardrops are suggested where thin traces meet vias and pads; not mandatory | Engineering practice |
| 10.11 Power pours | A power net with several loads (several decoupling capacitors, ferrite beads, multiple chip supply pins) **uses a copper pour block for its main path, not thin traces**; criterion: path resistance from supply pin to bypass capacitor ≤ 30 mΩ and inductance ≲ 4 nH. On a 4-layer board without a power layer (L2 and L3 both GND), put the pour on an outer layer and change layers with clusters of vias (about 1 per 1 A); pours do not run under high-speed lines and are not their reference; **when an inner layer has a power layer, multi-load supplies preferably use local copper blocks split on that layer, with each pad dropping into the plane through a nearby via, rather than long traces; power traces and pours must not pass under the top layer and second layer beneath the body of sensitive modules such as GNSS** (no clause limits copper blocks on the third layer); keep the SW-node pour of a DC-DC minimal, and use wider pours only on the VOUT side for distribution; branch currents of traces are still checked per 10.8. No dedicated literature was found for "a shared pour is better than separate traces for several components"; this is inference | Renesas R01AN5280 (trace resistance from pin to bypass capacitor ≤ 30 mΩ; inductance LVDS ≤ 3.5 nH, MIPI ≤ 4.6 nH); TI SLYT614, ROHM, MPS (DC-DC layout) |
| 10.12 Pad overlap | Pads of different components must not overlap or be shared; pad edge to edge ≥ 0.1 mm, **same net included**; where two components must be connected, connect them with a trace or a pour, not a shared pad; when library and self-made footprints have different pad sizes (JLC library pads are often longer than self-made ones), recheck after changing a footprint or moving a component | Two components on one pad cannot satisfy placement, stencil and reflow at the same time; JLC assembly follows "one component per pad" |
| 10.13 LDO output capacitor | For classic 1117-type LDOs whose datasheet relies on output-capacitor ESR for loop compensation (AMS1117, LM1117, LD1117 and so on), **do not fit only MLCCs and leave out a tantalum/electrolytic position**: the output must carry (or at least reserve as DNP) a **22 µF solid tantalum** (AMS1117 datasheet wording); TI LM1117 requires ESR of 0.3–22 Ω. After prototyping, measure no-load and full-load transients with an oscilloscope, and fit the tantalum if there is oscillation or droop. LDOs whose datasheet explicitly states "ceramic stable" (such as TPS7A20/TPS7A2033, Cout 1–200 µF) follow their own datasheet and are not covered by this clause. The purpose of the output tantalum is ESR stability, **not ripple filtering** | AMS1117 datasheet ds1117 (advanced-monolithic.com, Application Hints "Stability": 22 µF solid tantalum on the output ensures stability); TI LM1117 datasheet §9.2.2.1.3 (minimum 10 µF tantalum, ESR 0.3–22 Ω); TI AN-1482 (ESR zero and phase margin: about 68° with the zero, about 11° without; MLCC not suitable); Espressif ESP32-DevKitC V4 schematic output 22 µF; community: EEVblog 2025-01 "all-MLCC oscillated, stable after switching to aluminium electrolytic" (single source, no waveform). The AMS1117 datasheet gives no ESR range |
| 10.14 Re-check after changes | After changing a footprint, moving a component or editing routing, **re-export the Gerbers before auditing and checking** (using Gerbers from before the change gives outdated conclusions), and recheck: corners (10.1; moving a component often drags a former 45° line into a folded-back acute angle), pad overlap (10.12), board edge (10.5), via-in-pad (10.7), and DRC in detailed mode (the EasyEDA brief mode returns nothing). Library footprints may carry their own board-outline-layer lines (such as USB-C footprints), which end up in the Gerbers as cut lines and must be deleted from the footprint | — |

## Appendix A  ESP32-P4

Source: Espressif "ESP Hardware Design Guidelines" ESP32-P4, "PCB Layout" (docs.espressif.com, original checked on 2026-10-08).

| Item | Requirement |
|---|---|
| A.1 Layers | At least 4 layers: L1 signals and components; L2 complete ground, no routing; L3 power, which may carry high-speed lines when conditions allow, provided the reference is complete; L4 signals |
| A.2 Crystal | **≥ 4.5 mm** to the chip clock pin; otherwise per Chapter 7 |
| A.3 USB | Per 3.2; reserved resistor/capacitor placed close to the ESP32-P4 |
| A.4 MIPI | Per Chapter 5; with more layers, move to an inner layer right after leaving the chip and add a pair of return ground vias |
| A.5 SDIO | Per Chapter 6; with more layers, move to an inner layer right after leaving the chip |
| A.6 Power | 3.3 V main supply trace width ≥ 25 mil; VDD_HP main line ≥ 20 mil, DCDC close to the chip; 10 µF at each supply entry, 0.1 µF per pin; star distribution |
| A.7 Flash/PSRAM | The guide has no such section (PSRAM is in-package); external Flash per Chapter 4 |

## Appendix B  ESP32-S3

Source: Espressif "ESP Hardware Design Guidelines" ESP32-S3, "PCB Layout" (original checked on 2026-10-08).

| Item | Requirement |
|---|---|
| B.1 Crystal | **≥ 2.0 mm** to the chip clock pin; otherwise per Chapter 7 |
| B.2 Flash/PSRAM | 0 Ω series resistors close to the ESP32-S3; route SPI on an inner layer where possible (e.g. L3), guarding clock and data separately; octal SPI length-matched; when Flash and PSRAM are far from the chip, place decoupling capacitors on VDD_SPI and at the Flash and PSRAM supplies |
| B.3 RF | Per Chapter 8; add a stub of about 15 mil to the ground pad of the shunt capacitor on the chip side of the matching circuit (not needed when components are 0402 or larger); clear all layers under the IPEX connector |
| B.4 UART | U0TXD series resistor close to the chip and away from the crystal; guard UART with ground on both sides and add ground vias |
| B.5 Power | Main 3.3 V ≥ 25 mil, VDD3P3 branch ≥ 20 mil, others 10 mil; at least 2 vias per layer change; at least 9 ground vias under the chip's ground pad |

## Appendix C  Divergent values and choices

| Rule | Values from the sources | Value chosen by this specification and reason |
|---|---|---|
| High-speed line spacing | JLC 3W; HQ PCB 4W; TI 5W; Espressif MIPI 2W between pairs, 3W to other high-speed signals | Hard lower limit 3W (lowest consensus across sources), 5W recommended |
| Distance of return ground vias on layer change | SPEC-003 ≤ 1 mm; JLC Wiki ≤ 0.5 mm (the page states it was AI-assisted); TI ≤ 200 mil (5.08 mm, for Gbps-class) | Keep ≤ 1 mm |
| USB 2.0 total length | JLC ≤ 1800 mil; TI typical 4000 mil, maximum 12000 mil | Suggest ≤ 1800 mil; > 4000 mil must be explained |
| USB 2.0 intra-pair length difference | JLC ≤ 5 mil; TI general ≤ 50 mil; Atmel ≤ 150 mil; TI USB note ±2 mil (package level) | ≤ 5 mil; values with different bases are not compared |
| USB 2.0 vias per line | Espressif "minimize"; TI ≤ 4 | ≤ 2 |
| Via stubs | TI newer ≤ 15 mil (Gbps-class); TI USB note ≤ 200 mil | Not limited for grades ≤ 480 Mbps; for Gbps-class (such as MIPI) keep the whole pair on one layer, and ≤ 15 mil where a layer change is necessary (2.8) |
| Ground under the crystal | Espressif: complete ground plane; ECS: no ground | Espressif prevails |
| Corners | JLCPCB blog: 45° and arcs have no general advantage over each other; Zuken: 45° preferred for RF | Default 45°; RF follows Espressif's 135° or arc |
| USB trace width and spacing | HQ PCB rule of thumb 10/6 mil (stackup not stated) | Not adopted; use the calculator result per SPEC-003 6.5 |
| Guard-ground via spacing | Chip-vendor documents give no value; rules of thumb λ/20 to λ/10 (EDN, ProtoExpress, AtlasPCB); Panox ≤ 200 mil | λ/20 at 2.5 times the bit rate; grounding at the two ends only is forbidden (Bogatin) |
| 20H rule (pulling the power plane in by 20 times the dielectric thickness) | A SoC vendor's hardware design guide recommends it; IEEE "Effects of 20-H Rule and Shielding Vias on EMI Radiation" and Clemson CVEL: effect unstable on modern multilayer boards | **Not adopted** (last paragraph of Chapter 2) |

## Appendix D  ESP32-C3 (modules ESP32-C3-WROOM-02/02U)

Source: Espressif "ESP Hardware Design Guidelines" ESP32-C3, "PCB Layout" and "Schematic Design"; "ESP32-C3-WROOM-02/02U Datasheet" v1.7 (originals checked on 2026-10-12). This appendix addresses module carrier boards; designs with the chip mounted directly need a separate addition.

| Item | Requirement |
|---|---|
| D.1 Crystal | A 40 MHz crystal is integrated in the module and does not apply to the carrier board; avoid ultrasonic welding or cleaning of the module (datasheet 12.4) |
| D.2 Flash | Inside the module; does not apply to the carrier board (GPIO12–17 are not brought out) |
| D.3 RF/antenna (02U) | External 2.4 GHz, 50 Ω antenna with gain not above the certified value (FCC 1.57 dBi, other certifications 2.33 dBi), first-generation U.FL connector (datasheet 10.2); keep USB and UART traces, vias, test points and headers away from the antenna; an antenna must be connected whenever RF is enabled |
| D.4 Module placement | "Antenna extends beyond the board edge, clearance ≥ 15 mm" applies to the on-board PCB antenna (02) and **does not apply** to the 02U. Requirements for the 02U: keep the U.FL connector away from other RF interfaces on the board (such as an external antenna connector), and the feed line must not cross other RF areas |
| D.5 EPAD/ground | The EPAD may be left unsoldered; soldering gives better heat dissipation, but do not use too much solder paste (datasheet chapter 9); connect to a large ground area with wide copper; carrier-board ground vias use the thermal via array that comes with the KiCad standard footprint `RF_Module:ESP32-C3-WROOM-02U` (the primary documents give no count; unverified) |
| D.6 Power | 3.3 V supply, current ≥ 500 mA; ≥ 10 µF at the entry; main ≥ 25 mil, branches 10 mil, ≥ 2 vias per layer change |
| D.7 EN | RC 10 kΩ/1 µF, or a supply-supervisor chip (threshold about 3.0 V); keep the EN trace as short as possible |
| D.8 USB | 90 Ω differential, parallel and length-matched, continuous reference ground below, guarded with ground on both sides; series resistors and reserved capacitors close to module pins 13 and 14. Full-speed USB is grade U1 and follows 3.1, with impedance as a recommendation |
| D.9 UART | U0TXD series resistor 499 Ω, close to the module; guard the trace with ground on both sides and add ground vias |
| D.10 SPI | SPI_CLK gets a series resistor and a capacitor to ground, close to the module; the capacitor to ground may be left unpopulated by default and fitted according to EMI results |
| D.11 Strapping | GPIO2, GPIO8, GPIO9; GPIO9 pulled up and not loaded with a large capacitor; GPIO2 and GPIO8 must be high at power-up |
| D.12 Stackup | On a four-layer board, layer 2 is a complete ground with no routing |

## Appendix E  (Not included in this public edition)

(Not included in this public edition)

## Appendix G  (Not included in this public edition)

(Not included in this public edition)

## References

(Not included in this public edition)
