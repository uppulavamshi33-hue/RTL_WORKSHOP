# VSDBabySoC: RTL to Gate-Level Verification

## Overview

VSDBabySoC is a small System-on-Chip that combines a RISC-V processor with clock-generation and digital-to-analog interface blocks. This module demonstrates how a design progresses from RTL coding through logic synthesis and technology mapping to gate-level functional verification.

The main activities include RTL simulation, synthesis using Yosys, mapping to Sky130 standard cells, generating and inspecting the synthesized netlist, and performing post-synthesis simulation.

## Contents

1. VSDBabySoC Architecture
2. Project Setup
3. RTL Design and Testbench
4. RTL Simulation
5. Synthesis Using Yosys
6. Sky130 Technology Libraries
7. Netlist Optimization and Generation
8. Netlist Inspection
9. Gate-Level Simulation
10. POST_SYNTH_SIM and FUNCTIONAL
11. RTL and GLS Comparison
12. Complete Design Flow
13. Complete Yosys Script
14. Learning Outcomes
15. Conclusion

---

## 1. VSDBabySoC Architecture

The VSDBabySoC design uses `vsdbabysoc` as its top-level module. It connects the processor, clock-generation block, and digital-to-analog converter to create a complete system.

The architecture consists of three major components:

```text
                    VSDBabySoC
                        |
          +-------------+-------------+
          |             |             |
        RVMYTH         PLL           DAC
          |             |             |
     RISC-V Core     Clocking      Analog Output
```

### RVMYTH Processor

RVMYTH is the RISC-V processor core. It performs the main digital processing operations and supplies the digital data used by the output path.

### Phase-Locked Loop (PLL)

The PLL block provides clock-generation functionality for the system.

### Digital-to-Analog Converter (DAC)

The DAC receives digital data from the processor path and generates the corresponding analog output.

### Top-Level Integration

The `vsdbabysoc` module connects the RVMYTH, PLL, and DAC blocks. It defines how the major components communicate with each other.

---

## 2. Project Setup

A separate directory is used to organize the BabySoC simulation project. The repository is downloaded from GitHub before starting the simulation and synthesis activities.

```bash
mkdir baby_soc
cd baby_soc/
git clone https://github.com/Subhasis-Sahu/BabySoC_Simulation
cd BabySoC_Simulation
```

The project directory contains RTL source files, testbenches, technology libraries, and supporting resources required for verification and synthesis.

**Original screenshot:** `Screenshot 2026-08-22 143313`

---

## 3. RTL Design and Testbench

The RTL implementation describes the modules that form the VSDBabySoC. The top-level module, `vsdbabysoc`, establishes the connections between the processor, PLL, and DAC.

A testbench is used to apply the required clock and reset signals and monitor the behavior of the design during simulation.

The testbench supports the verification process before synthesis and after the synthesized netlist has been generated. This allows the original RTL behavior to be compared with the behavior of the mapped implementation.

---

## 4. RTL Simulation

Before synthesis, the RTL design is simulated to check its functional behavior. This establishes a reference against which the post-synthesis results can be compared.

### RTL simulation flow

```text
       RTL Source Files
              +
           Testbench
              |
              v
          Compilation
              |
              v
        RTL Simulation
              |
              v
         VCD Waveform
              |
              v
           GTKWave
```

### Compile the RTL simulation

Icarus Verilog is used to compile the testbench and the RTL modules:

```bash
iverilog -o ./pre_synth_sim.out -DPRE_SYNTH_SIM src/module/testbench.v -I src/include/ -I src/module/
```

### Run the simulation

```bash
./pre_synth_sim.out
```

### Open the waveform

```bash
gtkwave pre_synth_sim.vcd
```

The waveform viewer helps examine the behavior of important signals, including:

- `CLK` — clock signal
- `reset` — reset signal
- `OUT` — output signal
- `RV_TO_DAC[9:0]` — digital data bus sent toward the DAC

**Original screenshot:** `Screenshot 2026-08-26 181002`

The objective of RTL simulation is to confirm the expected behavior of the design before it is converted into a technology-mapped implementation.

---

## 5. Synthesis Using Yosys

After the RTL behavior has been checked, Yosys is used to synthesize the design. Synthesis transforms the behavioral RTL description into a logic-level representation suitable for implementation using cells from the target technology library.

### Synthesis flow

```text
          RTL Design
              |
              v
             Yosys
              |
              v
       Logic Synthesis
              |
              v
          Optimization
              |
              v
      Technology Mapping
              |
              v
      Gate-Level Netlist
```

### Start Yosys

```bash
yosys
```

The following commands are entered at the Yosys prompt, not at the regular Linux shell.

### Read the top-level RTL

```tcl
read_verilog src/module/vsdbabysoc.v
```

### Read the RVMYTH processor

```tcl
read_verilog -I src/include/ src/module/rvmyth.v
```

### Read the clock-gating module

```tcl
read_verilog -I src/include/ src/module/clk_gate.v
```

### Load the Liberty libraries

The PLL library is loaded using:

```tcl
read_liberty -lib src/lib/avsdpll.lib
```

The DAC library is loaded using:

```tcl
read_liberty -lib src/lib/avsddac.lib
```

The Sky130 standard-cell library is loaded using:

```tcl
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Select the top-level design

```tcl
synth -top vsdbabysoc
```

The `-top vsdbabysoc` option identifies the module from which Yosys should begin synthesizing the design hierarchy.

---

## 6. Sky130 Technology Libraries

The Sky130 high-density standard-cell library provides the cells used to implement the synthesized logic.

Examples of standard-cell types include:

- Logic gates
- Inverters
- Buffers
- Flip-flops

The PLL and DAC are represented by their respective Liberty files in the synthesis setup.

### Technology-mapping flow

```text
          RTL Design
              |
              v
             Yosys
              |
              v
       Sky130 Libraries
              |
              v
      Technology Mapping
              |
              v
     Mapped Gate-Level Design
```

### Mapping sequential elements

The `dfflibmap` command maps generic flip-flops to compatible sequential cells available in the Sky130 library.

```tcl
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The mapping process can be summarized as:

```text
      RTL Flip-Flop
            |
            v
         dfflibmap
            |
            v
      Sky130 Flip-Flop
```

### Mapping combinational logic

ABC performs combinational logic optimization and maps the logic to cells available in the target library.

```tcl
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

These mapping stages convert generic synthesis elements into technology-specific standard cells.

---

## 7. Netlist Optimization and Generation

After synthesis and technology mapping, additional cleanup commands are used to prepare the design for netlist generation.

### Replace undefined values

```tcl
setundef -zero
```

This command replaces undefined logic values with zero according to the selected synthesis flow.

### Remove unnecessary elements

```tcl
clean -purge
```

This removes unused objects and unnecessary connections where possible. It may be run again when needed:

```tcl
clean -purge
```

### Rename internal objects

```tcl
rename -enumerate
```

This organizes generated internal names by assigning enumerated identifiers.

### Display synthesis statistics

```tcl
stat
```

The statistics report provides information about the synthesized design and the cells used.

**Original screenshot:** `Screenshot 2026-08-23 134727`

### Write the synthesized netlist

```tcl
write_verilog -noattr baby_soc_net3.v
```

The command writes the synthesized design to `baby_soc_net3.v`. This file contains the gate-level implementation produced by the synthesis process.

---

## 8. Netlist Inspection

Inspecting the generated netlist helps explain how the original RTL hierarchy has been transformed into a synthesized implementation.

### Top-level connectivity

The major functional connections can be represented as follows:

```text
          PLL
           |
           | Clock
           v
         RVMYTH
           |
           | RV_TO_DAC
           v
           DAC
           |
           v
          OUT
```

This block-level view highlights the main signal path from the processor toward the DAC output.

**Original screenshot:** `Screenshot 2026-08-23 140945`

### Technology-mapped netlist

After ABC mapping, the internal logic is represented using Sky130 standard cells and their interconnections.

**Original image placeholder:** `image`

Compared with the block-level diagram, the mapped netlist contains many more individual elements because the internal logic has been expanded into technology-specific cells.

Netlist inspection is useful for identifying cell instances, checking connectivity, and understanding the implementation generated by synthesis.

---

## 9. Gate-Level Simulation

After generating the synthesized netlist, the design is simulated again. This is called Gate-Level Simulation (GLS), or post-synthesis simulation.

Unlike RTL simulation, GLS uses the synthesized Verilog netlist along with the functional Verilog models of the Sky130 standard cells.

### Gate-level simulation flow

```text
          Testbench
              +
      Gate-Level Netlist
              +
       Sky130 Cell Models
              |
              v
        Icarus Verilog
              |
              v
      Gate-Level Simulation
              |
              v
      post_synth_sim.vcd
```

### Compile the gate-level simulation

```bash
iverilog -DPOST_SYNTH_SIM -DFUNCTIONAL \
-I src/include/ \
-I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/ \
-I src/module/ \
src/module/testbench.v
```

### Run the compiled simulation

```bash
./a.out
```

**Original screenshot:** `Screenshot 2026-08-26 180113`

### View the resulting waveform

```bash
gtkwave post_synth_sim.vcd
```

**Original image placeholder:** `image`

The post-synthesis waveform can be examined to check whether the synthesized implementation behaves consistently with the original RTL design.

---

## 10. Understanding `POST_SYNTH_SIM` and `FUNCTIONAL`

Two compilation definitions are used in the gate-level simulation command: `POST_SYNTH_SIM` and `FUNCTIONAL`. They serve different purposes.

### `POST_SYNTH_SIM`

The following compiler option defines the `POST_SYNTH_SIM` macro:

```bash
-DPOST_SYNTH_SIM
```

The testbench can use this macro to select the post-synthesis simulation configuration.

For example:

```verilog
`ifdef POST_SYNTH_SIM

    // Post-synthesis simulation configuration

`else

    // RTL simulation configuration

`endif
```

This conditional compilation approach allows the testbench to support different simulation setups.

### `FUNCTIONAL`

The second definition is:

```bash
-DFUNCTIONAL
```

The Sky130 standard cells have Verilog simulation models. The `FUNCTIONAL` definition selects the functional model configuration where the models support this option.

The simplified simulation flow is:

```text
     Synthesized Netlist
              |
              v
      Sky130 Cell Models
              |
              v
      Functional Simulation
```

This setup allows the logical behavior of the synthesized design to be examined without requiring transistor-level simulation.

**Original screenshot:** `Screenshot 2026-08-23 140243`

---

## 11. RTL and Gate-Level Simulation Comparison

The final verification stage compares the RTL waveform with the gate-level waveform.

The two waveform files are:

```text
pre_synth_sim.vcd
post_synth_sim.vcd
```

### Signals to compare

- `CLK`
- `reset`
- `RV_TO_DAC[9:0]`
- `OUT`

### Comparison flow

```text
       RTL Simulation
              |
              v
      pre_synth_sim.vcd
              |
              | Compare
              v
      post_synth_sim.vcd
              ^
              |
      Gate-Level Simulation
```

The supplied project description reports consistent functional behavior between the RTL and GLS waveforms over the observed simulation interval.

The `RV_TO_DAC[9:0]` bus follows the same sequence in both simulations, while the clock, reset, and output signals remain consistent.

**Original screenshot:** `Screenshot 2026-08-26 181002`

Waveform comparison is an important check that synthesis has preserved the intended logic behavior. The conclusion applies to the signals and interval actually examined; it does not independently prove correctness for every possible input or operating condition.

---

## 12. Complete VSDBabySoC Design Flow

The entire flow can be summarized from RTL verification through synthesis and post-synthesis simulation.

```text
                    VSDBabySoC
                         |
                         v
                     RTL Design
                         |
                         v
              Pre-Synthesis Simulation
                         |
                         v
                  RTL Verification
                         |
                         v
                        Yosys
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
          Read RTL   Read Libraries  Select Top
             +-----------+-----------+
                         |
                         v
                     Synthesis
                         |
                         v
                     dfflibmap
                         |
                         v
                        ABC
                         |
                         v
               Technology Mapping
                         |
                         v
                Gate-Level Netlist
                         |
                         v
            Post-Synthesis Simulation
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
       POST_SYNTH_SIM FUNCTIONAL  Sky130 Models
             +-----------+-----------+
                         |
                         v
              Functional Verification
```

This flow illustrates the relationship between RTL simulation, logic synthesis, standard-cell mapping, netlist generation, and gate-level verification.

---

## 13. Complete Yosys Script

The following sequence combines the main synthesis commands used in the experiment.

```tcl
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include/ src/module/rvmyth.v
read_verilog -I src/include/ src/module/clk_gate.v

read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc

dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

opt

abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

setundef -zero
clean -purge
rename -enumerate

stat

write_verilog -noattr baby_soc_net3.v
```

**Note:** This script preserves the command sequence from the supplied material. Successful execution depends on the source-file paths, module dependencies, library compatibility, and the Yosys version. The script should be checked against the actual project environment before being used as an automated flow.

---

## 14. Learning Outcomes

The experiment develops practical understanding of the RTL-to-gate-level verification process.

The key learning outcomes are:

- Understanding the architecture of a multi-module SoC.
- Identifying the top-level module and its component connections.
- Studying the roles of the RVMYTH processor, PLL, and DAC.
- Performing RTL simulation using Icarus Verilog.
- Inspecting simulation waveforms with GTKWave.
- Managing Verilog include directories and source files.
- Loading RTL modules and Liberty libraries into Yosys.
- Performing logic synthesis and optimization.
- Mapping sequential cells using `dfflibmap`.
- Mapping combinational logic using ABC.
- Working with the Sky130 standard-cell library.
- Generating and inspecting a synthesized Verilog netlist.
- Running post-synthesis functional simulation.
- Understanding the `POST_SYNTH_SIM` and `FUNCTIONAL` definitions.
- Comparing RTL and gate-level simulation results.
- Checking whether synthesis preserves the observed functional behavior.

---

## 15. Conclusion

This module demonstrates the transformation of VSDBabySoC from its RTL description into a technology-mapped gate-level implementation. The process includes pre-synthesis simulation, Yosys synthesis, Sky130 standard-cell mapping, netlist generation, and post-synthesis functional verification.

The comparison of the RTL and GLS waveforms shows consistent behavior for the examined signals, including `RV_TO_DAC[9:0]`, `CLK`, `reset`, and `OUT`, over the observed simulation interval.

Overall, the experiment provides practical experience with the transition from RTL design to a synthesized gate-level netlist and demonstrates how post-synthesis simulation can be used to check that the implementation retains the intended functional behavior.
