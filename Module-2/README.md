# Module 2 – Sequential Logic, Timing Libraries and Synthesis

## Overview

Module 2 extends the RTL-to-synthesis flow by introducing **sequential logic, flip-flops, reset techniques, PVT conditions, standard-cell timing libraries, and synthesis using Yosys**. The session also covers simulation with **Icarus Verilog** and waveform observation using **GTKWave**.

---

## Contents

1. Flip-Flops and Reset Methods
2. PVT Conditions and SKY130 Library
3. Liberty `.lib` File
4. RTL Simulation
5. Synthesis Using Yosys
6. Hierarchical and Flattened Synthesis
7. Learning Outcomes
8. Conclusion

---

# 1. Flip-Flops and Reset Methods

A **flip-flop** is a sequential circuit used to store a single bit of data. Unlike combinational logic, its output depends on the clock and may also be controlled by reset or set signals.

### Synchronous Reset

In a synchronous reset flip-flop, the reset condition is evaluated only when the active clock edge occurs.

```verilog
always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

The reset does not immediately affect the output. The change occurs with the clock edge.

### Asynchronous Reset

An asynchronous reset operates independently of the clock. When the reset becomes active, the output is immediately driven to logic `0`.

```verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

### Asynchronous Set

An asynchronous set is similar to an asynchronous reset, but it forces the flip-flop output to logic `1`.

```verilog
always @(posedge clk or posedge set)
begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end
```

### Comparison

| Feature          | Synchronous Reset | Asynchronous Reset | Asynchronous Set |
| :--------------- | :---------------- | :----------------- | :--------------- |
| Output forced to | `0`               | `0`                | `1`              |
| Depends on clock | Yes               | No                 | No               |
| Response         | At clock edge     | Immediate          | Immediate        |

---

# 2. PVT Conditions and SKY130 Library

**PVT** refers to **Process, Voltage, and Temperature**. These three parameters have a significant effect on the delay, power consumption, and overall performance of semiconductor circuits.

The SKY130 standard-cell library used in this exercise is:

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

The filename can be interpreted as follows:

* `tt` → Typical process condition
* `025C` → Characterized at 25°C
* `1v80` → Operating voltage of 1.8V

The **SKY130 PDK** contains standard-cell libraries that provide information required by synthesis and technology-mapping tools.

---

# 3. Liberty `.lib` File

A **Liberty file** is a technology library file that describes the electrical and timing characteristics of standard cells.

It provides information such as:

* Cell functionality
* Input and output pins
* Propagation delay
* Timing arcs
* Input capacitance
* Power characteristics
* Cell area

The library can be inspected using:

```bash
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

Examining the file helps in understanding how synthesis tools obtain the information required to map RTL logic into physical standard cells.

---

# 4. RTL Simulation

The flip-flop designs were functionally verified before synthesis using **Icarus Verilog**. The generated waveform was then inspected using **GTKWave**.

### Simulation Flow

```text
RTL Design
     +
Testbench
     ↓
Icarus Verilog
     ↓
Simulation
     ↓
VCD Waveform
     ↓
GTKWave
```

### Simulation Commands

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
```

```bash
./a.out
```

```bash
gtkwave tb_dff_asyncres.vcd
```

The waveform allows us to verify the relationship between the clock, reset, input data, and flip-flop output.

---

# 5. Synthesis Using Yosys

**Yosys** is an open-source synthesis tool used to convert RTL descriptions into hardware netlists.

For sequential designs, the inferred flip-flop must also be mapped to an appropriate standard cell from the technology library.

### Yosys Commands

```text
yosys
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The overall process can be represented as:

```text
RTL Code
   ↓
RTL Synthesis
   ↓
Logic Optimization
   ↓
Flip-Flop Mapping
   ↓
Technology Mapping
   ↓
Gate-Level Netlist
```

The `dfflibmap` command is particularly important for sequential circuits because it maps the flip-flops inferred from the RTL to suitable flip-flop cells available in the SKY130 library.

---

# 6. Hierarchical and Flattened Synthesis

## Hierarchical Synthesis

In hierarchical synthesis, the structure of the original RTL modules is retained.

For example:

```text
Top Module
├── Sub Module 1
├── Sub Module 2
└── Sub Module 3
```

### Advantages

* Preserves the original design structure
* Makes debugging easier
* Useful for modular and block-based designs
* Can reduce synthesis complexity for large designs

### Limitations

* Optimization across module boundaries may be restricted
* Some analysis and reporting may require additional handling

---

## Flattened Synthesis

Flattened synthesis removes the module hierarchy and combines the logic into a single design representation.

Yosys can perform this operation using:

```text
flatten
```

### Advantages

* Allows optimization across module boundaries
* Provides a unified netlist
* Can improve overall logic optimization

### Limitations

* Can increase synthesis time for large designs
* Makes debugging more difficult
* Original module boundaries are no longer clearly visible

### Comparison

| Parameter            | Hierarchical          | Flattened              |
| :------------------- | :-------------------- | :--------------------- |
| Module hierarchy     | Preserved             | Removed                |
| Optimization         | Mainly within modules | Across complete design |
| Debugging            | Easier                | More difficult         |
| Design structure     | Modular               | Unified                |
| Large design runtime | Generally lower       | Generally higher       |
| Main purpose         | Modularity and reuse  | Global optimization    |

---

# 7. Learning Outcomes

After completing Day 2, I gained an understanding of:

* Sequential logic and flip-flop operation.
* Synchronous and asynchronous reset behavior.
* Asynchronous set functionality.
* Process, Voltage, and Temperature conditions.
* SKY130 standard-cell library naming.
* Purpose and contents of Liberty `.lib` files.
* RTL simulation using Icarus Verilog.
* Waveform verification using GTKWave.
* Flip-flop mapping using Yosys.
* Technology mapping with the SKY130 library.
* Differences between hierarchical and flattened synthesis.
* The basic transition from RTL to a technology-mapped netlist.

---

# 8. Conclusion

Module 2 provided practical knowledge of **sequential RTL design and synthesis**. It demonstrated how different reset and set coding styles influence flip-flop behavior and how standard-cell libraries provide the timing and electrical information required during technology mapping.

The session also showed the difference between **hierarchical and flattened synthesis**, highlighting their respective advantages in optimization, debugging, and design organization. Overall, Day 2 strengthened the understanding of how a sequential RTL design progresses from **coding and simulation to synthesis and standard-cell mapping**.
