# SKY130 – Pre-Layout Timing Analysis using OpenSTA and OpenLane
# Aim
To understand and perform pre-layout timing analysis of a digital design using SKY130 technology, timing libraries, delay tables, synthesis, OpenSTA and clock-tree concepts.
# Project Overview
This module explains how timing information is prepared and used before physical layout. The flow includes:
Timing modelling
Timing libraries
Delay tables
Synthesis
Setup and hold timing analysis

# Project Overview
This project focuses on pre-layout timing analysis using the SKY130 PDK. It explains how timing information is modelled using standard-cell libraries and delay tables, followed by synthesis and static timing analysis using OpenSTA. The project also covers ideal-clock timing analysis, Clock Tree Synthesis (CTS), clock buffering, crosstalk, and real-clock setup and hold analysis. Through these steps, the timing performance and possible timing violations of a digital design can be studied and improved.

# procedure
Matter:
This module explains pre-layout timing analysis using SKY130 technology, including timing libraries, delay tables, synthesis, OpenSTA, clock-tree synthesis and real-clock timing analysis.


Convert Grid Information to Tracks

Grid and track information is required for physical design and routing. The technology information is used to define the routing tracks.

<img width="1280" height="768" alt="sky130A_tech" src="https://github.com/user-attachments/assets/555de6e5-c7ee-46ff-8cb8-5d95053ae5ac" />


Then:
Track information can be inspected using the technology and LEF-related files.

<img width="1280" height="768" alt="tracksinfo" src="https://github.com/user-attachments/assets/1c367663-b5ec-42ad-a0ef-a60a8b5940ea" />




# Timing Libraries
Timing Analysis with Real Clocks
Setup Timing Analysis Using Real Clock
Unlike ideal-clock analysis, real-clock analysis considers the actual clock network.
The timing path includes:
Clock Source
     ↓
Clock Tree
     ↓
Launch Flip-Flop
     ↓
Combinational Logic
     ↓
Capture Flip-Flop

First: fast lib
<img width="1280" height="768" alt="fastlib" src="https://github.com/user-attachments/assets/a7c436da-908c-4824-988d-243257384763" />


Second: typical lib
<img width="1280" height="768" alt="typicallib" src="https://github.com/user-attachments/assets/e8243854-03e9-4a4e-b6bd-c73dbb590657" />


Third: slow lib
<img width="1280" height="768" alt="slowlib" src="https://github.com/user-attachments/assets/1e8de6d6-94f7-4855-b5c8-20368255248c" />


 # Library Information
After this matter:
LEF files contain physical information about standard cells and are used during physical design.
<img width="1280" height="768" alt="leffff" src="https://github.com/user-attachments/assets/e94b9bf4-5385-4f52-adac-ed11592c8dc4" />



# Configure Synthesis
Synthesis converts the RTL design into a gate-level netlist using the selected standard-cell library.
#  Commands:
yosys
read_verilog <design>.v
read_liberty -lib <library>.lib
synth -top <top_module>
stat
<img width="1280" height="768" alt="analy" src="https://github.com/user-attachments/assets/5a09832a-5407-48de-9e20-c7ed61b3ba6e" />


Then:
The synthesized design can be inspected to verify the generated cells and design statistics.
<img width="1280" height="768" alt="analy2" src="https://github.com/user-attachments/assets/fa180794-3afd-4118-815e-843f44c15a31" />


# Timing Analysis with Ideal Clocks 
Setup Timing Analysis
After explaining setup timing:
Setup timing verifies whether data reaches the destination flip-flop within the required time before the clock edge.
Commands:
read_liberty <library>.lib
read_verilog <design>.v
link_design <top_module>
read_sdc <constraints>.sdc
report_checks -path_delay max

Clock Jitter and Uncertainty
After:
Clock jitter represents variation in clock arrival time. Clock uncertainty provides a timing margin for such variations.
Command:


# OpenSTA Configuration
After:
read_liberty <library>.lib
read_verilog <design>.v
link_design <top_module>
read_sdc <constraints>.sdc
report_checks

 Clock Tree Synthesis
# Placement
After:
Before CTS, the design undergoes placement, where standard cells are positioned in the layout.
<img width="1280" height="768" alt="vsdinvlayout1" src="https://github.com/user-attachments/assets/46f604af-3b1a-4f02-b5e7-7fef8421299f" />


Then:
The placement can be inspected in greater detail.
<img width="1280" height="768" alt="vsdinvlayout2" src="https://github.com/user-attachments/assets/224aaece-c890-4de4-91c7-bf5bea00f3b4" />


For an enlarged view:
<img width="1280" height="768" alt="vsdinvlayout3" src="https://github.com/user-attachments/assets/1356f4df-794e-4cd8-9b5f-3dcf64538b64" />
#CTS
<img width="1280" height="768" alt="runcts" src="https://github.com/user-attachments/assets/da85fbaa-2032-41ff-96dd-06f3c201c679" />
#PRE-STA
<img width="1280" height="768" alt="presta2" src="https://github.com/user-attachments/assets/99c9ac3f-6066-4af0-ba6b-a56db93c2f24" />
<img width="1280" height="768" alt="presta_sta" src="https://github.com/user-attachments/assets/45986cfa-f1e2-49a5-a1a4-8c46883c224e" />
#slacks
<img width="1280" height="768" alt="oldslack" src="https://github.com/user-attachments/assets/82e86202-118e-4daf-8c31-5670b565b1b6" />
<img width="1280" height="768" alt="newslack" src="https://github.com/user-attachments/assets/8dafd94a-d91a-4e56-8fba-afa0692a035a" />
<img width="1280" height="768" alt="oldslack1" src="https://github.com/user-attachments/assets/aedb6295-0013-433d-af56-100af706b888" />
<img width="1280" height="768" alt="newslack1" src="https://github.com/user-attachments/assets/bcae774a-9d4f-4a34-806b-e4aad7d1c41d" />




# Result
The SKY130 pre-layout timing flow was studied successfully. Timing libraries and delay tables were understood, synthesis was configured, and setup/hold timing analysis was performed conceptually using OpenSTA. Clock-tree synthesis and the difference between ideal-clock and real-clock timing analysis were also studied.
# Conclusion
This module provides an understanding of how timing information is generated and used before and after clock-tree implementation. The use of Liberty timing libraries, delay tables, synthesis, OpenSTA and CTS helps identify timing problems and improve the overall performance of a digital design.
