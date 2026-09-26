MODULE 5
POWER DISTRIBUTION NETWORK (PDN) & DETAILED ROUTING

TritonRoute, GDSII Generation and Physical DRC Verification

1. Aim

To implement the Power Distribution Network (PDN), perform global and detailed routing using the OpenLANE/OpenROAD flow with TritonRoute, generate the final GDSII layout, and perform physical Design Rule Checking (DRC) for the picorv32a RISC-V core using the Sky130 PDK.

2. Objectives

Generate the power and ground distribution network consisting of rings, straps, and standard-cell rails using gen_pdn.

Perform global and detailed routing of the picorv32a design using the OpenLANE routing flow and TritonRoute.

Check routing completion and verify that there are no remaining unrouted connections after detailed routing.

Generate the final layout and stream it out to GDSII format using Magic.

Perform physical DRC verification and obtain zero DRC violations.

Generate the final runtime and summary reports for the completed OpenLANE run.

Handle practical flow issues such as limited disk space and intermediate execution requirements.

3. Environment Setup and Prerequisites

The OpenLANE interactive Tcl shell was used to initialize the picorv32a design. The required physical-design stages were completed before routing: synthesis, floorplan, placement, and clock tree synthesis (CTS).

package require openlane 0.9
prep -design picorv32a -tag RUN_MOD5_CLEAN -overwrite

4. Power Distribution Network (PDN) Generation

After CTS, the power and ground nets were configured as VDD and VSS. The PDN was then generated using gen_pdn. This step creates the power/ground distribution network required to supply the standard cells across the layout.

set ::env(VDD_NET) "VDD"
set ::env(GND_NET) "VSS"
gen_pdn

The generated PDN provides the required power and ground connectivity through the physical layout.

5. Detailed Routing Using TritonRoute

The routing stage was executed using the OpenLANE routing command. The routing flow performs global routing followed by detailed routing to connect the design nets using the available Sky130 metal layers.

run_routing

The routing log shows that detailed routing completed successfully. The reported routing statistics include approximately 1,007,681 µm total wire length and 139,894 vias. The routing stage also reported: “No DRC violations after detailed routing.”


<img width="1920" height="923" alt="voilatedd" src="https://github.com/user-attachments/assets/18030ff5-7c45-45d3-bdcf-765b72809a50" />




6. Routing and Timing Output

The routing run completed successfully, but the captured timing output contains negative slack values. For example, the displayed path has a slack of approximately -0.19 ns, while the summary shown in the same terminal output reports negative WNS/TNS values. Therefore, the screenshots should be treated as evidence of routing completion, not as evidence that timing was fully closed.

The terminal also reports that only about 1.1 GB of disk space remained. This was a practical limitation during the run and required cleanup of unnecessary files when additional storage was needed.



<img width="1920" height="923" alt="runrouting2" src="https://github.com/user-attachments/assets/ae1624ad-7745-4a7f-b23b-b452983453b6" />



7. Detailed Routing Statistics

The detailed-routing report displayed the following important routing statistics

<img width="1920" height="923" alt="runrouting" src="https://github.com/user-attachments/assets/5ff216c7-2349-4d0b-af35-881bd84a98b3" />
GDSII Generation and Magic DRC Verification

After detailed routing, Magic was used to process the routed layout and perform physical verification. The commands used were:

run_magic
run_magic_drc

The Magic DRC output explicitly reports that no errors were found and that there were no DRC violations after GDS streaming out.



<img width="1920" height="923" alt="magicc" src="https://github.com/user-attachments/assets/96539788-8924-42ad-92e0-c2ff2a8b50d5" />



9. Final Summary Report

The final runtime and summary report were generated using the following OpenLANE commands:

calc_total_runtime
generate_final_summary_report

The final summary identifies the design as picorv32a and records the run directory as:

/openLANE_flow/designs/picorv32a/runs/RUN_MOD5_CLEAN

The Magic DRC summary in the final report shows a total of 0 Magic DRC violations. The report also indicates that an LVS source was not found and that no antenna report was found; these are separate from the confirmed Magic DRC result.

/openLANE_flow/designs/picorv32a/runs/RUN_MOD5_CLEAN/reports/final_summary_report.csv





10. Results

<img width="1920" height="923" alt="finalsummaryreport" src="https://github.com/user-attachments/assets/4286e6ba-54e0-448c-91b9-d523a69ccd29" />


11. Observations

The PDN generation and detailed-routing stages completed successfully.

The routing report confirms completion and reports no DRC violations after detailed routing.

Magic verification reported zero DRC violations after GDS streaming.

The run encountered a low-disk-space condition, with approximately 1.1 GB remaining at the time shown in the screenshots.

The captured timing output contains negative WNS/TNS values, so timing closure should not be claimed from these screenshots alone.

The final summary also indicates that LVS and antenna report sources were not found in this particular run.

12. Conclusion

Module 5 of the RTL-to-GDS flow for the picorv32a RISC-V core was successfully executed through PDN generation, detailed routing, GDSII streaming, and Magic DRC verification. The final Magic DRC result was 0 violations, confirming that the checked physical design rules were satisfied. The detailed-routing stage also completed successfully with the reported routing statistics. However, the captured timing output shows negative slack, so timing closure should be verified separately before describing the design as fully timing-clean or tape-out ready.
