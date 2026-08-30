## RTL SIMULATION,SYNTHESIS AND GATE LEVEL SIMULATION FOR SEQUENCE_DETECTOR
## OVERVIEW
# Sequence Detector — RTL Design, Simulation, Synthesis and Gate-Level Verification

## Overview

A Sequence Detector is a sequential digital circuit used to identify a predefined pattern in a serial stream of input bits. The circuit processes one input bit at a time and produces an output whenever the required sequence is detected.

In this experiment, a sequence detector was designed using **Verilog HDL** and taken through the complete RTL-to-gate verification flow. The design was first simulated at the RTL level, followed by synthesis using **Yosys** and mapping to the **SKY130 standard-cell library**. The synthesized netlist was then simulated using Gate-Level Simulation (GLS), and the RTL and GLS waveforms were compared to verify that the synthesized circuit retained the intended functionality.

### Design Flow

**RTL Design → RTL Simulation → Synthesis → Netlist Generation → Gate-Level Simulation → RTL vs GLS Comparison**

---

# 1. Project Setup

A separate working directory was created for the sequence detector project. The required Verilog source code and testbench were placed inside the project directory.

```bash
mkdir sequence_detector
cd sequence_detector
```

The project folder was used to maintain the RTL source, testbench, simulation outputs, and synthesized netlist in an organized manner.

---

# 2. RTL Design and Testbench

The sequence detector was implemented using Verilog HDL.

The design consists of:

* Serial input `din`
* Clock signal `clk`
* Reset signal `reset`
* Output signal `detected`
* Internal state registers used to track the received sequence

The detector operates synchronously with the clock. On every active clock edge, the incoming input bit is examined and the state of the detector is updated.

A testbench was created to provide different input bit patterns to the design. The testbench generates the clock and reset signals and applies the required sequence to the input.

The output of the detector is monitored to determine whether the specified pattern has been recognized.
<img width="1920" height="922" alt="sequencegvim" src="https://github.com/user-attachments/assets/02099970-b096-41e9-b11f-39bce5bb732c" />
<img width="1920" height="922" alt="sequencetbgvim" src="https://github.com/user-attachments/assets/69c48147-9f7a-4d95-8430-5ae8c8b6accb" />



---

# 3. RTL Simulation

The RTL design was simulated before synthesis to verify its logical behavior.

The Verilog design and testbench were compiled using Icarus Verilog:

```bash
iverilog -o sequence_sim.out sequence_detector.v sequence_detector_tb.v
```

The generated simulation executable was then run:

```bash
./sequence_sim.out
```

The simulation produced a VCD waveform file, which was opened using GTKWave:

```bash
gtkwave sequence_detector.vcd
```

### Signals Observed

The following signals were examined in the waveform:

* `clk`
* `reset`
* `din`
* `detected`
* Internal state signals, if required

The input sequence was checked against the detector output to ensure that the output becomes active whenever the required sequence is received.

The RTL simulation serves as the reference for the later gate-level verification.
<img width="1920" height="922" alt="examgtkwave" src="https://github.com/user-attachments/assets/0a3546ea-7358-4b07-be5a-e185fe8493b5" />


After confirming the RTL functionality, the sequence detector was synthesized using Yosys.
## Logic synthesis using yosys

The Verilog design was first loaded into Yosys:

```bash
read_verilog sequence_detector.v
```

The top-level module was then synthesized:

```bash
synth -top sequence_detector
```

The synthesized logic was mapped to the SKY130 standard-cell library using:

```bash
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

The design was optimized and unused logic was removed:

```bash
opt
clean -purge
```

The synthesized design statistics were displayed using:

```bash
stat
```
<img width="1920" height="922" alt="examstat" src="https://github.com/user-attachments/assets/3e7db5e4-3e75-41f1-be79-18b563571ddc" />


Finally, the synthesized gate-level Verilog netlist was generated:

```bash
write_verilog -noattr sequence_detector_net.v
```

The generated netlist represents the sequence detector using standard cells instead of the original RTL constructs.


---

# 5. Netlist Inspection

The synthesized netlist was examined to understand the transformation of the RTL design into gate-level hardware.

## 5.1 RTL-Level Structure

At the RTL level, the sequence detector is represented using behavioral Verilog constructs such as sequential logic, state transitions, and conditional statements.

This representation makes it easier to understand the intended operation of the sequence detector.
<img width="1920" height="922" alt="examshow" src="https://github.com/user-attachments/assets/9072d5ae-ff07-4647-936f-4b0041c269ef" />


## 5.2 Gate-Level Netlist

After synthesis and technology mapping, the RTL description is converted into interconnected standard cells.

The gate-level implementation may contain:

* Flip-flops
* AND gates
* OR gates
* Inverters
* Multiplexers
* Other SKY130 standard cells

The synthesized netlist provides a structural representation of how the sequence detector can be implemented using physical standard-cell logic.
<img width="1920" height="922" alt="sequencenetlistgvim" src="https://github.com/user-attachments/assets/5aed7fb4-c85e-43d7-a8f7-86aa230fae3d" />


---

# 6. Gate-Level Simulation

After synthesis, the generated netlist was used for Gate-Level Simulation.

The standard-cell Verilog models were included during compilation so that the synthesized cells could be simulated correctly.

A typical GLS compilation command is:

```bash
iverilog -DFUNCTIONAL \
-I ./verilog_model/ \
sequence_detector_net.v sequence_detector_tb.v
```

The generated simulation executable was then executed:

```bash
./a.out
```

The post-synthesis waveform was opened using GTKWave:

```bash
gtkwave sequence_detector.vcd
```

During GLS, the same input sequence used during RTL simulation was applied to the synthesized netlist.
<img width="1920" height="922" alt="glsout" src="https://github.com/user-attachments/assets/ffe32275-a099-4b07-869e-c8047cf6ec19" />


---

# 7. RTL vs GLS Waveform Comparison

The RTL and Gate-Level Simulation waveforms were compared to verify the correctness of the synthesis process.

The important signals compared were:

* `clk`
* `reset`
* `din`
* `detected`
* State-related signals, where applicable

The input sequence and detector output were checked at corresponding clock cycles.

The GLS output followed the same functional behavior as the RTL simulation. The sequence was detected at the expected point, showing that the synthesized gate-level circuit correctly implements the original RTL design.

Therefore, no functional mismatch was observed between the RTL and gate-level simulations.
<img width="1920" height="922" alt="examgtkwave" src="https://github.com/user-attachments/assets/6738a746-fd83-4399-ba0d-724aaa8c013b" />
<img width="1920" height="922" alt="glsout" src="https://github.com/user-attachments/assets/367d62a1-9333-4d69-ae94-5a9223df0ed9" />


---

# 8. Conclusion

The Sequence Detector was successfully taken through the complete RTL design and verification flow.

The following stages were completed:

1. Sequence detector RTL design
2. Testbench development
3. RTL simulation using Icarus Verilog
4. Waveform analysis using GTKWave
5. Logic synthesis using Yosys
6. Technology mapping to the SKY130 standard-cell library
7. Gate-level netlist generation
8. Gate-Level Simulation
9. RTL and GLS waveform comparison

The RTL and GLS simulations produced consistent functional results for the applied input sequence. This confirms that the synthesis process preserved the intended operation of the sequence detector.

The experiment demonstrates the practical flow from a Verilog RTL description to a synthesized gate-level implementation and its subsequent functional verification.
