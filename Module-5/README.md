# Module 5 – RTL Constructs, Conditional Logic and Synthesis

## 1. Introduction

Module 5 focuses on understanding how different Verilog RTL coding styles are interpreted during simulation and converted into hardware during synthesis.

The experiments cover `if` and `case` statements, incomplete assignments, latch inference, MUX and DEMUX designs, generate constructs, partial case logic, and Ripple Carry Adders.

**Icarus Verilog** and **GTKWave** were used for simulation and waveform analysis, while **Yosys** with the **SKY130 standard-cell library** was used for synthesis and hardware-structure analysis.

---

## 2. Learning Objectives

The main objectives of this module are:

* Understanding `if` and `case` based combinational logic
* Identifying incomplete RTL assignments
* Understanding latch inference
* Implementing MUX and DEMUX circuits
* Using generate constructs for repeated hardware
* Understanding partial case assignments
* Constructing a Ripple Carry Adder
* Comparing RTL simulation with synthesized hardware
* Developing synthesis-friendly RTL coding practices

---

## 3. Combinational RTL and Conditional Statements

Combinational logic should produce an output for every possible input condition.

Verilog `if` and `case` statements are commonly used to describe this type of logic.

For example:

```verilog
always @(*)
begin
    if(sel)
        y = i1;
    else
        y = i0;
end
```

This can represent a simple multiplexer because the select signal determines which input is connected to the output.

Complete conditional assignments help the synthesis tool generate the intended combinational hardware.

---

## 4. Incomplete `if` and Latch Inference

An incomplete `if` statement occurs when an output is assigned only for some conditions.

```verilog
always @(*)
begin
    if(i0)
        y = i1;
end
```

When `i0` is low, no new value is assigned to `y`. To retain its previous value, synthesis may infer a latch.

The process can be represented as:

```text
Missing assignment
      ↓
Previous value retained
      ↓
Storage behavior
      ↓
Latch inference
```

An `else` branch or default assignment can be used to avoid unintended latches.

---

## 5. `case` Based Combinational Logic

A `case` statement is useful when several input conditions need to be handled.

Example:

```verilog
always @(*)
begin
    case(sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
        2'b10 : y = i2;
        2'b11 : y = i3;
    endcase
end
```

This type of structure can represent a **4-to-1 multiplexer**.

The `comp_case` experiment demonstrates case-based combinational logic and compares its RTL simulation with the synthesized structure.

### Evidence

* `comp_case_simulation.png`
* `comp_case_synthesis.png`

---

## 6. Bad Case Experiment

The `bad_case` experiment examines case-based RTL and its interpretation during synthesis.

The design was simulated using GTKWave and synthesized using Yosys to observe the resulting logic structure.

### Evidence

* `bad_case_simulation.png`
* `bad_case_synthesis.png`

This experiment helps demonstrate how RTL coding decisions affect synthesized hardware.

---

## 7. DEMUX Using Case

The `demux_case` experiment implements a demultiplexer using a `case` statement.

A DEMUX routes one input signal to one of several outputs depending on the select signal.

The simulation waveform verifies the output selected for different select combinations.

### Evidence

* `demux_case_simulation.png`

---

## 8. DEMUX Using Generate

The `demux_generate` experiment implements DEMUX functionality using generate-based RTL.

Generate constructs are useful when similar hardware structures need to be repeated. The simulation verifies the generated output paths for different select values.

### Evidence

* `demux_generate_simulation.png`

---

## 9. Generate Constructs

Generate statements are used to create repeated hardware structures during elaboration.

They are useful for:

* Repeated module instances
* Multi-bit circuits
* Parameterized designs
* Arithmetic structures
* Bus-based hardware

A procedural `for` loop is generally used inside an `always` block for repeated operations, whereas a `for generate` construct is used to replicate hardware.

| Procedural `for`              | `for generate`               |
| ----------------------------- | ---------------------------- |
| Used inside procedural blocks | Used for structural hardware |
| Repeats operations            | Replicates hardware          |
| Usually uses `integer`        | Uses `genvar`                |
| Used for vector processing    | Used for repeated modules    |

---

## 10. MUX Using Generate

The `mux_generate` experiment demonstrates a multiplexer implemented using generate-based construction.

The design was simulated to verify functionality and synthesized using Yosys to examine the resulting hardware.

### Evidence

* `mux_generate_simulation.png`
* `mux_generate_synthesis.png`

---

## 11. Partial Case Assignment

The `partial_case_assign` experiment studies the effect of partially defined case conditions.

When some possible case values are not assigned, the synthesis tool may infer additional hardware depending on the intended behavior.

The synthesized structure was examined using Yosys.

### Evidence

* `partial_case_assign_synthesis.png`

This experiment emphasizes the importance of providing complete assignments when designing combinational logic.

---

## 12. Overlapping Case Conditions

Wildcard case patterns must be used carefully.

For example:

```text
1?
```

can match:

```text
10
11
```

If another condition also matches one of these values, the conditions may overlap.

Unintended overlapping conditions can introduce priority behavior and make the synthesized design different from what was expected. Therefore, case conditions should be written clearly and carefully.

---

## 13. Ripple Carry Adder

The Ripple Carry Adder experiment demonstrates a multi-bit arithmetic circuit constructed using multiple full-adder stages.

Each full adder generates a sum and carry output. The carry from one stage is passed to the next stage.

```text
FA0 → FA1 → FA2 → FA3 → FA4 → FA5 → FA6 → FA7
```

Generate constructs can be used to create repeated full-adder instances efficiently.

### Evidence

* `rca_generate_structure.png`
* `rca_simulation_detailed.png`
* `rca_synthesis.png`

The experiment provides a clear connection between generated RTL, simulation, and synthesized hardware.

---

## 14. Hierarchical Design and Compilation

Large Verilog designs are often divided into multiple modules.

For example:

```text
fa.v
  ↓
rca.v
  ↓
tb_rca.v
```

Here, the RCA depends on the full-adder module, while the testbench is used for simulation.

All dependent files must be included during compilation.

```bash
iverilog fa.v rca.v tb_rca.v
```

The compiled simulation can be executed using:

```bash
./a.out
```

The generated waveform can be viewed with:

```bash
gtkwave tb_rca.vcd
```

If a required module is missing, the simulator cannot correctly resolve the design hierarchy.

---

## 15. Simulation and Synthesis Analysis

Simulation and synthesis provide different but complementary views of an RTL design.

### Simulation

Simulation is used to verify:

* Functional behavior
* Input-output relationships
* Signal transitions
* Correct operation of the design

GTKWave provides a visual representation of the simulation waveforms.

### Synthesis

Synthesis converts the RTL into a hardware representation and helps identify:

* Logic gates
* Multiplexers
* Latches
* Module connections
* Standard-cell implementations

Therefore, both simulation and synthesis should be checked during RTL development.

---

## 16. Tools Used

* **Verilog HDL** – RTL design
* **Icarus Verilog** – Simulation
* **GTKWave** – Waveform analysis
* **Yosys** – RTL synthesis
* **SKY130** – Standard-cell library for ASIC analysis

---

## 17. Key Learning Outcomes

The major lessons from Module 5 are:

1. RTL coding style directly affects synthesized hardware.
2. Combinational outputs should be assigned for all possible conditions.
3. Incomplete `if` statements can cause latch inference.
4. Incomplete `case` statements can also result in storage elements.
5. Default assignments help create complete combinational logic.
6. `if` and `case` constructs can describe selection and multiplexer logic.
7. Generate constructs are useful for repeated hardware structures.
8. Procedural `for` and `for generate` have different purposes.
9. Partial and overlapping case conditions should be handled carefully.
10. Ripple Carry Adders can be constructed using repeated full-adder stages.
11. Hierarchical designs require all dependent modules during compilation.
12. Simulation checks functionality, while synthesis shows the inferred hardware.

---

## 18. Module 5 Summary

Module 5 provided practical knowledge of **RTL coding constructs, conditional logic, latch inference, MUX and DEMUX implementations, generate structures, partial case assignments, and Ripple Carry Adders**.

The overall flow studied in this module is:

```text
Verilog RTL
    ↓
Simulation
    ↓
GTKWave Analysis
    ↓
Yosys Synthesis
    ↓
Hardware Structure
    ↓
SKY130 Standard Cells
```

Overall, the experiments demonstrated that proper RTL coding is essential for obtaining predictable and synthesis-friendly hardware. The module strengthened the understanding of the relationship between **Verilog code, simulation results, synthesis behavior, and actual digital hardware**.
