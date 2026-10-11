# Physical Design and Multi-Corner Timing Analysis of VSDBabySoC using OpenROAD Flow Scripts (sky130hd)

## Abstract
This project implements the RTL-to-GDSII physical design of VSDBabySoC, a small RISC-V based SoC with an on-chip PLL (`avsdpll`) and DAC (`avsddac`), using the open-source OpenROAD-flow-scripts (ORFS) on the SkyWater sky130hd platform. The work was done in a cloud environment (GitHub Codespace with a noVNC desktop). The design was taken through synthesis, floorplan, placement, clock tree synthesis, routing and finishing, with timing tracked after every stage. The final routed design was analysed across all 16 PVT corners of the sky130_fd_sc_hd library. It meets setup and hold at the typical corner, at all fast corners and at slow corners of 1.60 V and above. Setup fails only at five slow corners between 1.28 V and 1.44 V. Hold is met at all 16 corners.

## 1. Introduction
Physical design converts a synthesised netlist into a manufacturable layout. ORFS chains Yosys (synthesis) and OpenROAD (floorplan, placement, clock tree synthesis, routing) into one automated flow. VSDBabySoC contains analog macros (PLL and DAC) that are treated as hard macros with LEF and Liberty views, so the flow places and routes around them.

## 2. Objectives
- Run the complete ORFS physical design flow on VSDBabySoC in a cloud-based environment.
- Integrate the `avsdpll` and `avsddac` hard macros into the floorplan.
- Monitor setup timing at each stage of the flow.
- Perform post-route static timing analysis across all 16 PVT corners and interpret the results.

## 3. Tools and Environment

| Item | Details |
|---|---|
| Flow | OpenROAD-flow-scripts (ORFS) |
| Platform | sky130hd (sky130_fd_sc_hd) |
| Design | `vsdbabysoc` |
| Environment | GitHub Codespace (VSD-PD) with XFCE desktop through noVNC (port 6080) |
| Macros | `avsddac` and `avsdpll` (LEF and Liberty) |
| Clock | `clk`, 12 ns period |
| Analysis | OpenROAD / OpenSTA with sky130_fd_sc_hd corner Liberty files |

## 4. Design Description
VSDBabySoC integrates a RISC-V CPU core, a PLL that provides the clock, and a DAC that converts the digital output to analog. In the physical design, the PLL and DAC are hard macros with fixed internal layouts (met1 to met5) supplied through their LEF files. The synthesis constraints are in `vsdbabysoc_synthesis.sdc`, and macro placement is controlled by `macro.tcl`.

## 5. Methodology: Flow Execution
The flow was run with:

`make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk`

| Stage | Result file | Description |
|---|---|---|
| 1. Synthesis | `1_synth` | Yosys converts RTL to a gate-level netlist and SDC |
| 2. Floorplan | `2_floorplan` | Die and core definition, macro placement, tap cells, power distribution network |
| 3. Placement | `3_place` | Global placement, I/O placement, resizing, detailed placement |
| 4. Clock tree synthesis | `4_cts` | Clock tree built and timing updated with the propagated clock |
| 5. Routing | `5_route` | Global routing and detailed routing |
| 6. Finishing | `6_final` | Filler cells (209,798 instances placed), final ODB, DEF, netlist and SPEF |

The final layout was inspected in the OpenROAD GUI with `make gui_final` and `make gui_route`.

## 6. Results: Stage-wise Layout and Timing

**6.1 Floorplan.** The floorplan defines the core area and places the `dac` and `pll` hard macros inside it, with the power grid across the core. The worst setup slack is 0.872 ns (required 10.570 ns, arrival 9.698 ns). This is the least optimised estimate, because cells are not yet placed or resized.
<img width="1920" height="1080" alt="Screenshot 2026-10-03 102909" src="https://github.com/user-attachments/assets/e9fcf9e5-2db2-41fe-8b1e-fd6665eb35a9" />



 Floorplan zoomed on the PLL macro (`2_floorplan`).** This close-up shows the `pll` macro (`avsdpll`) and its internal metal layers. The macro is fixed, so the flow places the standard cells around it and leaves its layout unchanged.
<img width="1920" height="1080" alt="Screenshot 2026-10-03 103145" src="https://github.com/user-attachments/assets/8610ccf1-e2ba-4f04-9478-d7f6799173ab" />

**6.2 Placement.** The standard cells are legally placed in rows between the power straps. With placement-based parasitic estimation, the worst setup slack improves to 6.443 ns (required 10.841 ns, arrival 4.398 ns), because placement and resizing shorten the paths.
    
<img width="1920" height="1080" alt="Screenshot 2026-10-03 112834" src="https://github.com/user-attachments/assets/d3261f57-9bf4-46b3-b2bc-794e951ec266" />

** Placement view (`3_place`).** The standard cells are legally placed in rows between the power straps. Setup slack improves from 0.872 ns to 6.443 ns.

**6.3 Clock Tree Synthesis.** After the clock tree is built, the worst setup slack is 6.431 ns (required 11.691 ns, arrival 5.260 ns). It is nearly unchanged from placement, so the clock tree did not degrade setup timing.

  <img width="1920" height="1080" alt="Screenshot 2026-10-03 115551" src="https://github.com/user-attachments/assets/17d0e878-ac21-46f8-b8ec-80be8aac9107" />

** Cell distribution after clock tree synthesis (`4_cts`).** The heat map shows how densely the cells are packed in the core once the clock tree is built. Setup slack is 6.431 ns.
  
  <img width="1920" height="1080" alt="Screenshot 2026-10-03 120502" src="https://github.com/user-attachments/assets/6f88fc27-85d4-4c9f-a270-d41368a12b60" />

**Figure : Instance and pin view after clock tree synthesis (`4_cts`).** The cells form one cluster inside the core and the pins sit around the boundary.

**6.4 Final Routed Layout.** After routing and filler insertion, all routing layers (li1, met1 to met5) are in use, with VDD/VSS straps on met5. The worst setup slack is 6.965 ns (required 12.721 ns, arrival 5.756 ns), and the worst path is reported as slack 6.97 ns (MET).

  <img width="1920" height="1080" alt="Screenshot 2026-10-10 155102" src="https://github.com/user-attachments/assets/a0fb02b0-a85c-42af-98e7-a17e34e440c9" />
  
**Figure : Final routed layout with timing report (`6_final`).** All routing layers are in use, with VDD/VSS straps on met5. The worst setup slack is 6.965 ns.

**6.5 Stage-wise Setup Slack Summary**

| Stage | Required (ns) | Arrival (ns) | Setup slack (ns) |
|---|---|---|---|
| Floorplan | 10.570 | 9.698 | 0.872 |
| Placement | 10.841 | 4.398 | 6.443 |
| CTS | 11.691 | 5.260 | 6.431 |
| Final routing | 12.721 | 5.756 | 6.965 |

Setup slack rises sharply from floorplan to placement and then stays above 6.4 ns. The required time grows from 10.570 ns to 12.721 ns because the clock network delay is included once the clock tree exists.

## 7. Post-Route Multi-Corner Static Timing Analysis

**7.1 Method.** The final database (`6_final.odb`), parasitics (`6_final.spef`) and constraints (`6_1_fill.sdc`) were loaded in OpenROAD once per corner, together with the corner's sky130_fd_sc_hd Liberty file and the `avsddac` and `avsdpll` macro libraries. A shell script (`sta_all_corners.sh`) looped over all 16 corners and wrote one report per corner. The same SPEF and RC settings are used for every corner, so only the cell timing libraries change.

**7.2 Results**

| Corner | WNS (ns) | TNS (ns) | Setup slack (ns) | Hold slack (ns) |
|---|---|---|---|---|
| ff_100C_1v65 | 0.00 | 0.00 | 8.15 | 0.25 |
| ff_100C_1v95 | 0.00 | 0.00 | 9.03 | 0.20 |
| ff_n40C_1v56 | 0.00 | 0.00 | 6.91 | 0.29 |
| ff_n40C_1v65 | 0.00 | 0.00 | 7.62 | 0.25 |
| ff_n40C_1v76 | 0.00 | 0.00 | 8.24 | 0.23 |
| ff_n40C_1v95 | 0.00 | 0.00 | 8.94 | 0.19 |
| ss_100C_1v40 | -2.72 | -208.52 | -2.72 | 0.89 |
| ss_100C_1v60 | 0.00 | 0.00 | 2.10 | 0.63 |
| ss_n40C_1v28 | -29.05 | -13088.24 | -29.05 | 1.68 |
| ss_n40C_1v35 | -15.57 | -5581.01 | -15.57 | 1.25 |
| ss_n40C_1v40 | -9.98 | -2742.13 | -9.98 | 1.05 |
| ss_n40C_1v44 | -6.71 | -1529.25 | -6.71 | 0.94 |
| ss_n40C_1v60 | 0.00 | 0.00 | 0.43 | 0.63 |
| ss_n40C_1v76 | 0.00 | 0.00 | 3.73 | 0.48 |
| tt_025C_1v80 | 0.00 | 0.00 | 7.13 | 0.31 |
| tt_100C_1v80 | 0.00 | 0.00 | 7.26 | 0.31 |


**Figure 7: Multi-corner STA terminal output.** Worst setup and hold slack for each of the 16 PVT corners. Setup fails at five slow, low-voltage corners (1.28 V to 1.44 V), and hold is met at all 16 corners.
  <img width="1920" height="1080" alt="Screenshot 2026-10-10 172932" src="https://github.com/user-attachments/assets/2033d76a-da5f-446d-84a1-394f4502c934" />


**Figure : Corner summary table.** WNS, TNS, setup slack and hold slack for every corner. WNS and TNS are 0.00 at all passing corners.

   <img width="1388" height="827" alt="Screenshot 2026-10-10 184407" src="https://github.com/user-attachments/assets/cd8cc9e6-f3cd-41c5-abc4-ac706cd2a8b8" />


**Figure : WNS across PVT corners.** WNS is 0.00 ns at 11 of 16 corners. It dips to -29.05 ns at `ss_n40C_1v28` and recovers as voltage rises.


  <img width="1393" height="818" alt="Screenshot 2026-10-10 184437" src="https://github.com/user-attachments/assets/90a24c2e-9285-4f2d-809f-8ed133464453" />

**Figure : Setup slack across PVT corners.** Setup slack is positive at all fast and typical corners (6.91 ns to 9.03 ns) and crosses zero between 1.44 V and 1.60 V on the slow corners.

  <img width="1368" height="807" alt="Screenshot 2026-10-10 184507" src="https://github.com/user-attachments/assets/e70e41c2-9ee7-462e-a246-999fe60cb57f" />

**Figure : Hold slack across PVT corners.** Hold slack is positive everywhere. It is smallest at the fast corners (0.19 ns) and largest at the slowest corner (1.68 ns), the opposite trend to setup.
  <img width="1340" height="767" alt="Screenshot 2026-10-10 184524" src="https://github.com/user-attachments/assets/7f01a3d0-aeaf-4af9-83d1-2b6e0b3b1228" />

**7.3 Observations**
1. **Fast and typical corners are clean.** WNS and TNS are 0.00 at all eight ff and tt corners, with setup slack from 6.91 ns to 9.03 ns.
2. **Setup fails at five slow corners:** `ss_100C_1v40`, `ss_n40C_1v28`, `ss_n40C_1v35`, `ss_n40C_1v40` and `ss_n40C_1v44`. The worst is `ss_n40C_1v28` with WNS of -29.05 ns and TNS of -13088.24 ns.
3. **The failures are widespread at the lowest voltages.** Dividing TNS by WNS shows at least about 450 failing endpoints at 1.28 V, so the problem is not limited to one critical path.
4. **Supply voltage is the dominant factor.** At the slow process and -40°C, setup slack goes from -29.05 ns at 1.28 V to -6.71 ns at 1.44 V and becomes positive at 1.60 V (+0.43 ns) and 1.76 V (+3.73 ns). Lower voltage reduces drive strength and increases cell delay.
5. **Temperature effect at low voltage.** At 1.40 V, `ss_100C` (-2.72 ns) performs better than `ss_n40C` (-9.98 ns), consistent with temperature inversion at low supply voltage.
6. **Hold is met at all 16 corners.** The smallest hold slack is 0.19 ns at `ff_n40C_1v95` and the largest is 1.68 ns at `ss_n40C_1v28`. Setup and hold move in opposite directions across corners.
7. **Consistency check.** The typical corner gives 6.97 ns in the ORFS final report and 7.13 ns in the multi-corner run, which agree closely.

## Conclusion

The physical design of VSDBabySoC was completed using OpenROAD-flow-scripts on the sky130hd platform in a cloud-based Codespace, with the PLL and DAC integrated as hard macros. The design went through synthesis, floorplan, placement, clock tree synthesis, routing and finishing, and the final routed layout was verified with static timing analysis.

**Key results**
- **Setup timing improved through the flow.** Worst setup slack rose from 0.872 ns at floorplan to 6.443 ns after placement, 6.431 ns after CTS, and 6.965 ns after final routing, with a 12 ns clock.
- **The typical corner closes comfortably.** At TT, 25°C, 1.80 V, setup slack is 7.13 ns and hold slack is 0.31 ns in the multi-corner run.
- **11 of 16 PVT corners meet setup.** These are all fast corners, both typical corners, and the slow corners at 1.60 V and above, with WNS and TNS equal to 0.00 ns.
- **5 slow corners fail setup.** They are at 1.28 V to 1.44 V, with the worst case at `ss_n40C_1v28` (WNS -29.05 ns, TNS -13088.24 ns).
- **Hold is met at all 16 corners.** The smallest hold slack is 0.19 ns at `ff_n40C_1v95`.

**Overall assessment**
The routed design meets timing at nominal and fast conditions and at slow conditions down to 1.60 V. Supply voltage is the main driver of setup slack, since lower voltage weakens cell drive and increases delay. The failing corners are far below the nominal 1.8 V supply, so they reflect extreme low-voltage operation and not the normal operating point.

