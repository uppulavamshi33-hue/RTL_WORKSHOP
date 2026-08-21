# Module 4 — Gate-Level Simulation, Blocking vs Non-Blocking Assignments, and Synthesis-Simulation Mismatch

## Overview

Module-4 focused on understanding the relationship between **RTL simulation, synthesis, and Gate-Level Simulation (GLS)**. The experiments demonstrated how coding practices such as sensitivity lists and assignment types can affect simulation results and synthesized hardware.

The main areas covered were:

* Gate-Level Simulation
* RTL and GLS comparison
* Synthesis-simulation mismatch
* Incomplete sensitivity lists
* Blocking and non-blocking assignments
* Combinational and sequential coding styles
* Yosys-based synthesis
* Icarus Verilog simulation
* GTKWave waveform analysis
* SDF-based timing simulation

---

## 1. Gate-Level Simulation

**Gate-Level Simulation (GLS)** is used to simulate the netlist generated after synthesis. Unlike RTL simulation, which works with behavioral Verilog code, GLS works with gates and standard-cell models.

A synthesized design may contain:

* AND, OR and NOT gates
* Multiplexers
* Flip-flops
* Buffers
* Technology-specific standard cells

GLS helps verify that the synthesized hardware still performs the intended function of the RTL design.

### Gate-Level Verification Flow

```text
RTL Design
     ↓
RTL Simulation
     ↓
Synthesis
     ↓
Gate-Level Netlist
     ↓
Gate-Level Simulation
     ↓
Waveform Comparison
```

---

## 2. RTL Simulation vs Gate-Level Simulation

### RTL Simulation

RTL simulation checks the functional behavior of the original HDL description. It does not normally represent the actual gates, standard-cell delays, or physical implementation.

### Gate-Level Simulation

GLS uses the synthesized netlist and standard-cell simulation models. It provides an additional verification step after synthesis.

### Key Difference

```text
RTL Simulation → Checks RTL functionality

GLS → Checks synthesized hardware behavior
```

Comparing both results helps identify **synthesis-simulation mismatches**.

---

## 3. Incomplete Sensitivity Lists

A sensitivity list determines which signal changes cause an `always` block to execute.

Consider:

```verilog
always @(sel) begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

Although the circuit is intended to behave as a multiplexer, only `sel` is included in the sensitivity list.

If `i0` changes while `sel` remains unchanged, the block may not execute. Therefore, the simulated output may not update immediately.

### Correct Approach

For combinational logic, use:

```verilog
always @(*)
```

or:

```systemverilog
always_comb
```

These constructs automatically include the signals used by the combinational block.

### Important Point

An incomplete sensitivity list can create a **simulation-only mismatch** because synthesis tools generally determine the intended combinational logic from the RTL equations, while the simulator follows the specified event controls.

---

## 4. Blocking Assignments

Blocking assignments use the `=` operator.

Example:

```verilog
always @(*) begin
    x = a | b;
    y = x & c;
end
```

With blocking assignments, each statement takes effect immediately before the next statement is executed.

Therefore, the order of statements matters.

For combinational logic, the statements should normally follow the intended data flow:

```text
a,b
 ↓
 OR
 ↓
 x
 ↓
 AND ← c
 ↓
 y
```

Blocking assignments are generally preferred when describing combinational logic.

---

## 5. Non-Blocking Assignments

Non-blocking assignments use the `<=` operator.

Example:

```verilog
always @(posedge clk) begin
    q <= d;
end
```

The update is scheduled rather than taking effect immediately within the procedural sequence. This behavior makes non-blocking assignments suitable for modeling clocked sequential hardware.

For example:

```verilog
always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;
end
```

At the clock edge, `q2` receives the previous value of `q1`, correctly representing two flip-flops connected in series.

### Recommended Rule

```text
Combinational Logic → always @(*) → Blocking (=)

Sequential Logic   → always @(posedge clk) → Non-Blocking (<=)
```

---

## 6. Synthesis Using Yosys

**Yosys** was used to synthesize the RTL design and convert it into a gate-level representation.

The SKY130 standard-cell library was used during technology mapping.

### Synthesis Flow

```text
Read RTL
   ↓
Read Standard Cell Library
   ↓
RTL Synthesis
   ↓
Logic Optimization
   ↓
Technology Mapping
   ↓
Gate-Level Netlist
```

The generated netlist contains instances of standard cells instead of the original high-level RTL constructs.

---

## 7. Gate-Level Simulation Using Icarus Verilog

The synthesized netlist can be simulated using **Icarus Verilog** together with the required SKY130 Verilog models.

The simulation process includes:

1. Compile the standard-cell models.
2. Compile the synthesized netlist.
3. Add the testbench.
4. Run the simulation.
5. Generate a VCD waveform.
6. Open the waveform using GTKWave.

Example RTL simulation:

```bash
iverilog <design.v> <testbench.v> -o rtl.out
./rtl.out
gtkwave <dumpfile.vcd>
```

Example GLS compilation:

```bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
<netlist.v> \
<testbench.v> \
-o gls.out
```

The generated RTL and GLS waveforms can then be compared.

---

## 8. Waveform Analysis Using GTKWave

**GTKWave** was used to visualize signal transitions during simulation.

Waveform comparison helps check:

* Input-output behavior
* Signal transitions
* Logic correctness
* RTL and GLS consistency
* Unexpected simulation differences

If the RTL and GLS waveforms do not match, the design should be investigated for coding, synthesis, or timing-related issues.

---

## 9. Synthesis-Simulation Mismatch

A synthesis-simulation mismatch occurs when the behavior observed in RTL simulation differs from the synthesized gate-level design.

Possible causes include:

* Incomplete sensitivity lists
* Incorrect assignment usage
* Improper blocking assignment order
* Incomplete combinational assignments
* Unintended latch inference
* Timing effects

Good RTL coding practices and RTL-to-GLS comparison help detect these problems early.

---

## 10. SDF-Based Timing Simulation

**Standard Delay Format (SDF)** is used to provide timing information for gate-level simulation.

SDF may contain information about:

* Cell propagation delays
* Interconnect delays
* Input and output delays
* Setup timing
* Hold timing

### Timing Verification Flow

```text
RTL Design
     ↓
Synthesis
     ↓
Gate-Level Netlist
     ↓
SDF Generation
     ↓
SDF Back-Annotation
     ↓
Timing Simulation
```

Unlike basic RTL simulation, SDF-based GLS provides a more realistic view of the circuit by including timing delays.

---

## 11. Common RTL Coding Issues

| Issue                        | Effect                                  | Recommended Practice             |
| ---------------------------- | --------------------------------------- | -------------------------------- |
| Incomplete sensitivity list  | Output may not update during simulation | Use `always @(*)`                |
| Wrong blocking order         | Incorrect procedural behavior           | Follow data-flow order           |
| Blocking in sequential logic | Possible simulation races               | Use non-blocking `<=`            |
| Missing assignments          | May infer unwanted latches              | Assign outputs in all conditions |
| RTL/GLS mismatch             | Different behavior after synthesis      | Compare RTL and GLS              |
| Ignoring delays              | Timing behavior not visible             | Use SDF-based GLS                |

---

## 12. Practical Verification Flow

The experiments followed a complete RTL-to-gate verification process:

```text
RTL Coding
    ↓
Testbench Creation
    ↓
RTL Simulation
    ↓
Waveform Analysis
    ↓
Yosys Synthesis
    ↓
Gate-Level Netlist
    ↓
Gate-Level Simulation
    ↓
RTL vs GLS Comparison
    ↓
SDF Timing Analysis
```

This flow helps ensure that the synthesized design continues to meet the intended functional behavior.

---

## 13. Tools Used

### Icarus Verilog

Used for compiling and simulating RTL and gate-level designs.

### GTKWave

Used to view and analyze simulation waveforms.

### Yosys

Used for RTL synthesis, logic optimization, and generation of the gate-level netlist.

### SKY130 Standard-Cell Library

Used for technology mapping and gate-level simulation.

### SDF

Used for adding timing information to gate-level simulations.

---

## 14. Learning Outcomes

After completing Day 4, I understood:

* The purpose and importance of Gate-Level Simulation
* The difference between RTL and GLS
* How synthesis converts RTL into a gate-level netlist
* How incomplete sensitivity lists affect simulation
* Proper coding practices for combinational logic
* Proper use of blocking assignments
* Proper use of non-blocking assignments in sequential logic
* How statement ordering can affect simulation behavior
* How Yosys performs synthesis and technology mapping
* How Icarus Verilog performs RTL and GLS
* How GTKWave is used for waveform comparison
* The purpose of SDF back-annotation
* How timing effects can be analyzed after synthesis
* How to identify possible RTL/GLS mismatches

---

## Conclusion

Day 4 provided practical experience in **RTL simulation, synthesis, Gate-Level Simulation, coding styles, waveform verification, and timing analysis**.

The experiments highlighted the importance of using complete sensitivity lists, selecting the correct assignment type, and verifying the synthesized design against the original RTL.

### Complete Flow

```text
RTL Design
    ↓
RTL Simulation
    ↓
Synthesis using Yosys
    ↓
Gate-Level Netlist
    ↓
GLS using Icarus Verilog
    ↓
Waveform Analysis using GTKWave
    ↓
RTL/GLS Comparison
    ↓
SDF Timing Simulation
```


