# Automatic schematic layout for CircuitJS1: research report

- **Purpose:** Keep the full findings behind the circuit layout engine in section 10.3 of [the t3-learn plan](../../PLAN.md), with every source.
- **For:** Whoever builds the layout engine.
- **Date:** 2026-10-18.
- **How it was made:** A research subagent read tool repositories, papers and the CircuitJS1 source code, at commit `5a70716` (2026-09-23). Its report is below, unchanged except for formatting. Three claims were then checked again.

## Spot-check results

| Claim | How it was checked | Result |
| :--- | :--- | :--- |
| This CircuitJS1 version exports XML but still imports the old text format | Read `CirSim.java` (`dumpCircuit` uses `XMLSerializer`) and `CircuitLoader.java` (`readCircuit` takes XML when text starts with `<`, otherwise text) | Confirmed |
| Weave converts SPICE netlists to LTspice schematics with a layered layout and a round-trip check: 100% on 117 circuits, 88.4% on 3,460 ADI demos | [Weave paper abstract on arXiv](https://arxiv.org/abs/2607.03835) | Confirmed |
| Weave uses elkjs, classifies nets as ground, rail or signal, and falls back to simpler layouts | [Weave README on GitHub](https://github.com/senolgulgonul/weave) | Confirmed. The README doesn't say ground and rails become flags; that detail comes from the researcher's reading of Weave's source. |

## Report

### Summary

Adapt rather than reuse or build from scratch. Use elkjs (ELK's layered algorithm) to place the parts. Around it, write a thin TypeScript layer that turns ground and supply connections into CircuitJS1 ground, rail and labeled-node symbols, draws a few common sub-structures from fixed templates, and writes 16-px-grid CircuitJS1 text. The closest prior design is Weave (July 2026), which does exactly this for LTspice: SPICE netlist → elkjs → `.asc`, with a check that reads its own output back and compares connectivity, plus a fallback ladder of simpler layouts. The AI should give only coarse hints (net roles, stage order, orientation, "feedback", "above U1"), never coordinates. Pure-LLM layout tops out at about 76% of outputs that even load. Expect about 3–4 weeks for a solid first version and about 1 week for a prototype; these are the researcher's own estimates.

### Existing tools

| Tool | What it does | Fit for analog | Language | License | Maintained (last activity) |
|---|---|---|---|---|---|
| netlistsvg (+ analog skin) | Draws SVG from a Yosys JSON netlist using elkjs. Every cell gets ELK `FIXED_POS` ports taken from the skin. The analog skin flows top to bottom (`direction="DOWN"`), and the user picks the orientation by choosing `r_h` or `r_v`. | Partial. Open issues #31 (horizontal flow makes VCC-top/GND-bottom hard) and #40 (auto-orientation experiments) are from 2018. | TypeScript/JS | MIT | Mostly inactive: last commit 2024-01-25, last release 1.0.2 (2020-12). A v2 pull request was closed unmerged in 2026-03. |
| ELK layered / elkjs | Sugiyama-style layered layout with orthogonal routing, port constraints, layer constraints, partitions, model order and junction-point output. | Good as an engine, not an analog tool. You must remove ground/rail nets and choose edge directions yourself. | Java; JS built with GWT (1.6 MB, about 467 KB gzipped) | EPL-2.0 (elkjs master: "EPL-2.0 OR GPL-3.0-or-later") | Yes. ELK commit 2026-10-12, v0.12.0 released 2026-07; elkjs 0.12.0 released 2026-07-17. |
| Weave | SPICE → LTspice `.asc`. Classifies nets as ground/rail/signal; ground and rails become flags, not wires. elkjs settings: RIGHT, ORTHOGONAL, FIXED_POS. Placement patterns for feedback, divider legs, hanging shunts and supply corners. Round-trip check plus a 10-step fallback ladder. | Good; the closest match. It reports 100% verified connectivity on 117 circuits and 88.4% on 3,460 ADI demos. Long value text can overlap wires; long multi-stage chains are "correct but not compact". | JS, single HTML file | MIT per the README; GitHub detects no LICENSE file | Yes, last push 2026-08-19 |
| Lcapy | Draws Circuitikz schematics from a netlist with a direction hint on every part (`R1 1 2; right=2`). x and y are solved separately by a longest-path graph placer. Has `autoground` and implicit ground/rail symbols. | Partial. Output looks good, but every part needs a hint, inconsistent hints break the layout, some loops fail (noted in its own source), and it needs LaTeX. | Python | LGPL-2.1 | Yes, last commit 2026-08-16 |
| SKiDL → schematizer | SKiDL 3.0 hands schematic drawing to a new `schematizer` package (KiCad 6–10, `.kicad_sch` or SVG). Uses force-directed placement plus global and switchbox routing. The author's 2023 verdict: "the block placement doesn't look great." | Poor for this use. The README says each run gives a different drawing and "There is no way to pin a layout down." | Python | MIT | Yes. SKiDL 3.0.0 on 2026-10-16; schematizer 0.2.0 on 2026-10-13. |
| schemdraw | Python drawing library: you place elements one after another with right/down/at/to/tox calls. No netlist input. | Poor for automatic layout. A good model for relative, hint-style drawing. | Python | MIT | Yes, last commit 2026-10-08 |
| KiCad efforts | KiCad itself only auto-places field text ("Autoplace Fields"). kicad-sch-api draws L-shaped routes with no collision detection and no placement. kicad-schematic-builder: placement is manual, then checked with kicad-cli. ksch: deterministic YAML-to-KiCad compiler with its own layout pipeline. | Poor or partial | Python | kicad-sch-api MIT; kicad-schematic-builder MIT; ksch no license detected | kicad-sch-api last push 2025-12; ksch 2026-07; kicad-schematic-builder 2026-10 |
| libavoid (adaptagrams), libavoid-js | Orthogonal connector routing around obstacles. Routing only, no placement. | Partial: useful if you place parts yourself | C++; WASM port | libavoid-js LGPL-2.1; adaptagrams license not shown | adaptagrams 2025-10; libavoid-js 2026-09 |
| Academic and industrial work | SPAR (Frezza & Levitan, IEEE TCAD 1993): partition → place → global route → local route, with readability judged by "functional identification and traceability". ASG (Jehng et al., 1991). Garg et al. 2008: analog netlists. Hsu & Lin, MLCAD 2022: building-block classification plus reinforcement learning. Patents: SpringSoft/Synopsys rule-based generator (2005), TI template-driven generator (2017), AnaGlobe current-flow "device leveling" (2022). LLM approaches: Schemato (76% of outputs compile), SchGen (2026, relative placement). | Rules, templates and current-flow ordering are the analog-specific ideas. LLM-only output is unreliable. | n/a | papers/patents | n/a |

The researcher found no existing netlist→CircuitJS1 converter. A GitHub search found only a prompt guide that warns against having LLMs write CircuitJS coordinates. The search was not exhaustive.

### Algorithms

- **Layered (Sugiyama) with port constraints.** Phases: cycle removal, layering, crossing reduction, coordinate assignment. This gives left-to-right signal flow, few crossings, orthogonal routes and fixed pin positions (`FIXED_POS`). **Best fit** for the signal graph. Weave uses it, and netlistsvg sets `FIXED_POS` on every cell. Weaknesses: analog nets have no direction, so you must assign one (Weave uses breadth-first depth from the sources). Cycle breaking can route feedback badly. It has no notion of "up = higher voltage", so remove ground and rail nets first.
- **Orthogonal topology-shape-metrics (Tamassia 1987).** Planarize, minimize bends by min-cost flow, then compact. Good for bends, but the classic method assumes degree ≤ 4 and knows nothing about signal flow. Poor fit as the main engine.
- **Force-directed.** Springs and repulsion. Results depend on the random start, so it is not deterministic, and it is not orthogonal. SKiDL uses it and its author is unhappy with the output. Poor fit.
- **Rule/template placement.** Match known sub-structures and place them by fixed rules: SpringSoft "soft groups" (current mirrors, differential pairs, vertical order from power to ground, left-to-right signal order), TI templates, Weave's patterns. **Essential for the analog look.** Use it together with layered placement.
- **Constraint-graph / longest path (Lcapy).** Hints fix relative directions; x and y are solved separately. Simple and deterministic, but it needs consistent hints. A good fallback or alternative for very small circuits.
- **Simulated annealing / reinforcement learning (Hsu & Lin, AnaGlobe).** The reward counts crossings, bends, wire length and building-block rules. Good quality is reported, but it is heavy, needs training, and is not needed for tutoring-sized circuits.
- **Label placement.** CircuitJS1 fixes value-text positions itself (see below). Reserve a keep-out box per part; no free label placement is needed. ELK can size nodes to include labels (`nodeSize.constraints = NODE_LABELS`).
- **Routing.** Grid A* with a turn penalty (CircuitJS1's own router does this) or libavoid. Use it only to repair wires after snapping to the grid.

### How the best approaches handle analog conventions

- **Ground and rails.** Weave turns ground and supply nets into flags and lays out only signal nets. Lcapy has `autoground` and implicit `vcc`/`gnd` symbols. AnaGlobe orders devices along current flow from supply to ground. netlistsvg's analog skin flows downward so VCC lands on top.
- **Feedback loops.** Weave has "local and far feedback" patterns. CircuitJS1's own example `amp-invert.txt` puts the feedback resistor above the op-amp (`r 192 144 336 144` over `a 192 192 336 192`). That matches CircuitJS1's default of the "−" input on top.
- **Multi-pin parts.** Use fixed port offsets per symbol (netlistsvg skins, Weave's pin table for 5,093 symbols) with ELK `FIXED_POS`.
- **AI hints help, if they stay coarse and soft.** Lcapy shows that per-part direction hints plus a solver work. SchGen has the LLM give "relative placement". Schemato (LLM-only) compiles 76% of the time, while deterministic Weave reaches 100% on the same set. Map hints to ELK partitions (stage → column), model order (top-to-bottom order) and `layerConstraint` FIRST/LAST (inputs/outputs). Map pattern hints to templates. Drop any hint that conflicts with the others.

### CircuitJS1 specifics

All paths are under `src/com/lushprojects/circuitjs1/client/` in the CircuitJS1 repository.

- **Grid.** 16 px by default, 8 px with "Small Grid" (`UIManager.java:1013-1016`; flag bit 2 of the `$` line, `CircuitLoader.java:273`). `snapGrid` rounds to the nearest grid point (`CirSim.java:536-538`), but only while editing (`CircuitElm.java:558-570`). When a file loads, coordinates are used exactly as written (`CircuitLoader.java:194-206`). In the bundled examples, about 90% of coordinates are multiples of 16 and the rest multiples of 8.
- **Text line format.** `type x1 y1 x2 y2 flags params…` (`CircuitLoader.java:142-206`). This version *exports XML* (`<cir>`, attribute `x="x1 y1 x2 y2"`; `CirSim.java:452-463`, `CircuitElm.java:1417-1421`) but still *imports* text (`CircuitLoader.java:73-81`).
- **Base geometry.** (x,y) is `point1` and (x2,y2) is `point2` (`CircuitElm.java:66-70, 331-339`). Two-terminal parts have their posts at the two endpoints (`getPost`, `:818-820`). The body sits centered between the leads (`calcLeads`, `:343-351`); a resistor's leads are 32 px apart (`ResistorElm.java:54`). Perpendicular offsets use `interpPoint(a,b,f,g)` (`:417-423`); for a left-to-right part, a positive offset means up.
- **Connections.** Posts connect only where coordinates match exactly (`SimulationManager.java:126`, `pointKey`). Wires are split at T-junctions only during interactive drawing (`WireElm.java:258-265`), not on load. **So the generator must split every wire at every junction.** A post that sits in the middle of a wire looks connected but is not.
- **Voltage source `v`.** Post 0 at (x,y) is −, post 1 at (x2,y2) is + (`VoltageElm.java:141-147`). Parameters: waveform, frequency, max voltage, bias, phase, duty (`:62-91`). Example `voltdivide.txt` uses `v 112 368 112 48` with + at the top.
- **Op-amp `a` (3 posts).** Post 0 is the "−" input at (x, y−16), post 1 the "+" input at (x, y+16), post 2 the output at (x2,y2) (`OpAmpElm.java:119-144`). The sign logic keeps "−" on top for both left-to-right and right-to-left. `FLAG_SWAP=1` swaps the inputs. `FLAG_SMALL=2` shrinks the offsets to ±8, which is off the 16 grid, so avoid it (`:28-31, 113-118`). Size: triangle 52 px long, ±32 px tall. Horizontal or vertical only (`noDiagonal`).
- **BJT `t` (3 posts).** Base at (x,y). For a left-to-right NPN, collector at (x2, y2−16) and emitter at (x2, y2+16). PNP puts the emitter on top. `FLAG_FLIP=1` swaps (`TransistorElm.java:44, 211-255`). Parameters: `pnp vbe vbc beta model` (`:58-75`). Example `ceamp.txt`: `t 240 208 336 208`, collector at (336,192).
- **MOSFET `f` (3–4 posts).** Gate at (x,y). For NMOS, source at (x2, y2+16) and drain at (x2, y2−16). For PMOS the roles of posts 1 and 2 swap, so the source ends up on top. `FLAG_FLIP=8`; optional body post (`MosfetElm.java:35-37, 220, 361-372, 391-403`).
- **Potentiometer.** 3 posts (`PotElm.java:69-75`).
- **Chips.** Pins sit on N/S/E/W sides at 32-px pitch (16 px in small mode). Flip bits are 10–12 (`ChipElm.java:31-33, 77-83, 181-213`).
- **Parts that remove wires:**
  - `g` ground: one post at (x,y); the symbol is drawn past (x2,y2). All grounds are the same node (`GroundElm.java:62-123`). Write it as `g x y x y+16` (or +32).
  - `R` rail: one post at (x,y), label at (x2,y2). It is an ideal source to ground (`RailElm.java:39-108`). Several `R` elements are several sources, which changes what the current display shows.
  - `207` labeled node: one post; every labeled node with the same text is the same net (`LabeledNodeElm.java:98-126`). Format: `207 x y x2 y2 4 NAME`.
- **Value text.** Placed automatically (`CircuitElm.java:916-944`): centered above horizontal parts, to the right of vertical parts, and to the left for voltage sources. Font is 12 px (`:53`). Reserve that box in the layout.
- **Already in the code.** An XML-only routed wire `rw` that routes itself with an A* grid router (turn penalty 4, L/Z shapes tried first, no overlapping same-direction wires) when the circuit loads (`RoutedWireElm.java:37-38, 109-122`; `WireRouter.java:23, 183+, 421+, 659-666`). It routes one two-point wire at a time, and it only treats elements loaded before it as obstacles (`XMLDeserializer.java:177-190`).
- **JS API.** `importCircuit`, `exportCircuit` and `getElements` (`JSInterface.java:46, 65-68`) let the app check the result inside the simulator.

### Recommended approach

1. **AI output.** JSON netlist: parts (id, type, value, pin→net), nets with a role (input, output, gnd, supply with its voltage, signal), and optional hints (`stage` column, `order` within a column, `orient` h/v, `place` "above U1", `pattern` feedback/divider/bias). No coordinates. Validate against a schema.
2. **Classify nets.** Ground nets become one `g` per ground pin, pointing down. Supply nets become one `R` rail plus `207` labeled nodes for the other taps, which keeps a single source. Only signal nets go into the layout graph (as in Weave).
3. **Apply patterns** (rigid blocks with fixed internal geometry):
   - a part from a signal node to ground or rail is vertical;
   - a divider is a vertical column with the tap on the right;
   - an op-amp's feedback parts go above it, from the input x to the output x; the "+" input's network goes below;
   - a transistor stage gets its load up to the rail and its degeneration down to ground.
4. **Place with elkjs.** Settings: `direction RIGHT`, `edgeRouting ORTHOGONAL`, `portConstraints FIXED_POS`. Make every size and spacing a multiple of 16. Set a fixed `randomSeed` and keep the input order stable. Give input parts `layerConstraint FIRST` and output parts `LAST`. Map AI `stage` to partitions and AI `order` to model order. Point edges along breadth-first depth from the sources.
5. **Convert to CircuitJS1 coordinates.** Snap each node to 16. Compute each part's (x1,y1,x2,y2) from its post positions using a geometry table that copies the `setPoints` formulas above. Turn ELK edge sections into `w` segments. Merge collinear segments of the same net. Split wires at ELK `junctionPoints` and wherever an endpoint touches another wire of the same net. If snapping breaks a route, reroute that net with your own grid A* (turn penalty, bodies and label boxes as obstacles).
6. **Emit text.** A `$` header, then part lines, then wires. Using text, not XML, keeps you on the stable import path.
7. **Check the result automatically.**
   - Connectivity: recompute posts from the emitted lines with the same table. Merge coincident points, wire endpoints, same-name labeled nodes and all grounds. The resulting nets must equal the input nets exactly.
   - Geometry: part and label boxes don't overlap; no wire crosses a body; no post of one net sits on another net's wire; different nets never overlap; no 4-way junctions; every coordinate on the grid.
   - Conventions: rails above ground; inputs leftmost; outputs rightmost.
   - Score crossings, bends and area.
   - If a check fails, step down a fallback ladder (drop patterns → drop AI hints → widen spacing → labels for long nets), as Weave does. Never show a circuit that fails the connectivity check.
8. **Check in the simulator.** `CircuitJS1.importCircuit(text)`, then compare the `getElements()` count. Build a regression set by turning the 373 bundled example circuits into netlists with the step 7 extractor, laying them out again, and taking CI screenshots for visual review.

**Effort (the researcher's estimate):**
- geometry table, writer and parser: 2–3 days
- connectivity and geometry checks: 2 days
- ground/rail/label symbols: 1–2 days
- elkjs integration, snapping and wire splitting: 3–5 days
- grid re-router: 2 days
- 4–5 patterns: 3–5 days
- hint schema: 1–2 days
- regression set: 2–3 days

Total about 3–4 weeks. A prototype with only ground/rail symbols and vertical shunts takes about 1 week.

### Risks and open questions

- ELK returns floating-point coordinates. Snapping to 16 px can break routes, so plan for the re-router.
- Junctions must be exact. CircuitJS1 never connects a post to the middle of a wire, so a missed split silently cuts the circuit.
- Feedback and cycles: without patterns, layered layout routes loops awkwardly.
- Rail semantics: several `R` elements are several ideal sources. Decide whether supply current must stay visible.
- Value text can collide with wires when values are long; Weave reports the same issue.
- Format drift: this version exports XML, and `rw` exists only in XML. Pin the embedded build and test text import against it.
- Licensing: CircuitJS1 is GPL-2.0. Don't copy `WireRouter` into a non-GPL app; write your own. elkjs is EPL-2.0, Lcapy and libavoid-js are LGPL, and Weave's MIT license is stated only in its README.
- Open question: are AI hints accurate enough to be worth it? Measure layout quality with and without hints on the regression set.
- Open question: for very small circuits, would an Lcapy-style longest-path solver be simpler than ELK?

### Sources

GitHub repo metadata (licenses, last commits, releases, issues) came from `https://api.github.com/repos/<owner>/<repo>`.

- https://github.com/nturley/netlistsvg
- https://raw.githubusercontent.com/nturley/netlistsvg/master/lib/analog.svg
- https://raw.githubusercontent.com/nturley/netlistsvg/master/lib/Cell.ts
- https://raw.githubusercontent.com/nturley/netlistsvg/master/lib/elkGraph.ts
- https://github.com/nturley/netlistsvg/issues/31
- https://github.com/nturley/netlistsvg/issues/40
- https://github.com/nturley/netlistsvg/pull/153
- https://eclipse.dev/elk/reference/algorithms/org-eclipse-elk-layered.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-portConstraints.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-layered-layering-layerConstraint.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-layered-layering-layerChoiceConstraint.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-layered-crossingMinimization-positionChoiceConstraint.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-layered-crossingMinimization-inLayerPredOf.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-junctionPoints.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-layered-feedbackEdges.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-partitioning-partition.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-layered-considerModelOrder-strategy.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-nodeSize-constraints.html
- https://eclipse.dev/elk/reference/options/org-eclipse-elk-randomSeed.html
- https://github.com/kieler/elkjs
- https://raw.githubusercontent.com/kieler/elkjs/master/LICENSE.md
- https://raw.githubusercontent.com/kieler/elkjs/master/package.json
- https://cdn.jsdelivr.net/npm/elkjs@0.12.0/lib/elk.bundled.js
- https://github.com/eclipse-elk/elk
- https://github.com/senolgulgonul/weave
- https://raw.githubusercontent.com/senolgulgonul/weave/main/weave_index_v41_20260819_1951.html
- https://arxiv.org/abs/2607.03835
- https://arxiv.org/html/2607.03835v1
- https://lcapy.readthedocs.io/en/latest/schematics.html
- https://raw.githubusercontent.com/mph-/lcapy/master/lcapy/schemplacer.py
- https://raw.githubusercontent.com/mph-/lcapy/master/lcapy/schemgraphplacer.py
- https://raw.githubusercontent.com/mph-/lcapy/master/lcapy/schemplacerbase.py
- https://raw.githubusercontent.com/mph-/lcapy/master/lcapy/schemlineqplacer.py
- https://raw.githubusercontent.com/mph-/lcapy/master/lcapy/schemgraph.py
- https://raw.githubusercontent.com/mph-/lcapy/master/doc/schematics.rst
- https://raw.githubusercontent.com/mph-/lcapy/master/doc/examples/schematics/opamp1.sch
- https://raw.githubusercontent.com/devbisme/skidl/master/docs/generating-editable-schematics-2023-07-14.html
- https://raw.githubusercontent.com/devbisme/skidl/master/src/skidl/tools/kicad9/gen_sch.py
- https://raw.githubusercontent.com/devbisme/skidl/master/src/skidl/tools/kicad9/gen_svg.py
- https://raw.githubusercontent.com/devbisme/skidl/master/src/skidl/tools/kicad5/gen_sch.py
- https://github.com/devbisme/schematizer
- https://pypi.org/pypi/schematizer/json
- https://schemdraw.readthedocs.io/en/latest/usage/placement.html
- https://github.com/cdelker/schemdraw
- https://docs.kicad.org/9.0/en/eeschema/eeschema.html
- https://kicad-sch-api.readthedocs.io/en/latest/ORTHOGONAL_ROUTING.html
- https://github.com/bleugreen/ksch
- https://github.com/movray/kicad-schematic-builder
- https://github.com/circuit-synth/kicad-sch-api
- https://www.adaptagrams.org/documentation/libavoid.html
- https://github.com/Aksem/libavoid-js
- https://github.com/mjwybrow/adaptagrams
- https://github.com/ogdf/ogdf
- https://arxiv.org/abs/2411.13899
- https://arxiv.org/abs/2605.30345
- https://arxiv.org/abs/2008.10583
- https://www.fitee.zjujournals.com/en/article/doi/10.1631/FITEE.2400612/
- https://api.openalex.org/works/doi:10.1109/43.238032
- https://api.openalex.org/works/doi:10.1016/0167-9260(91)90004-5
- https://api.openalex.org/works/doi:10.1109/ewdts.2008.5580148
- https://api.openalex.org/works/doi:10.1145/3551901.3556486
- https://api.openalex.org/works/doi:10.1016/j.jvlc.2013.11.005
- https://patents.google.com/patent/US7386823
- https://patents.google.com/patent/US11645438
- https://patents.google.com/patent/US12475291
- https://users.fit.cvut.cz/~fiserp/dip/2005/Kucera
- https://en.wikipedia.org/wiki/Layered_graph_drawing
- https://en.wikipedia.org/wiki/Force-directed_graph_drawing
- https://chomsky.uni-trier.de/Lehre/Sommer-2023/GV/05-orthogonal-drawings.pdf
- https://hwdocs.hackclub.dev/user-contrib-guides/schematic-practices/
- https://github.com/dul1-git/falstad-ai-prompts
- https://github.com/pfalstad/circuitjs1

### Claims the researcher could not fully verify

- Swinkels & Hafer's expert-system work: the SFU thesis PDF was blocked, so it is not cited above.
- SPAR, ASG, Garg et al. and Hsu & Lin: abstracts and metadata only, not full texts. The ACM page returned 403.
- The FITEE review is paywalled; only its abstract and reference list were seen.
- Chimani et al. (port constraints, 2011): only the title and authors, from an OpenAlex search.
- ELK's default `randomSeed` is not stated in its docs, so set it explicitly.
- Weave's MIT license is stated in its README only.
- OGDF's and adaptagrams' licenses were not checked.
- The effort numbers are the researcher's own estimates.
