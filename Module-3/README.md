# Module 3 – Combinational and Sequential RTL Optimization

## 1. Objective

Module-3 was dedicated to studying how **Yosys optimizes RTL designs** by identifying unnecessary logic and reducing hardware while maintaining the required functionality.

The major concepts covered were:

* Combinational logic optimization
* Constant propagation
* Boolean expression simplification
* Removal of redundant logic
* Sequential circuit optimization
* Flip-flop and register optimization
* Removal of unused state elements
* State optimization
* Retiming
* Sequential logic cloning
* SKY130 technology mapping

---

## 2. RTL Optimization

RTL optimization is the process of improving the hardware implementation of an RTL design without changing its intended operation.

The synthesis tool examines the design and tries to create a simpler and more efficient circuit.

### Main Goals

* Eliminate unnecessary hardware
* Reduce logic complexity
* Simplify Boolean expressions
* Remove redundant registers and flip-flops
* Reduce area usage
* Improve power efficiency
* Support better timing and technology mapping

RTL optimization is mainly classified into:

* **Combinational optimization** – optimizes logic based on present input values.
* **Sequential optimization** – optimizes registers, flip-flops and state-related logic.

---

## 3. Yosys Setup for Optimization

The RTL design is first loaded into Yosys along with the SKY130 standard-cell library.

### Start Yosys

```bash
yosys
```

### Load the SKY130 Liberty File

```text
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This command loads the timing and cell information required for technology mapping.

### Load the Verilog Design

```text
read_verilog <design_name>.v
```

This imports the RTL source code into Yosys.

### Perform Synthesis

```text
synth -top <module_name>
```

This synthesizes the selected module and treats it as the top-level design.

---

## 4. Combinational RTL Optimization

Combinational optimization focuses on simplifying logic whose output depends only on the current inputs.

The synthesis process can identify fixed values, repeated logic and unnecessary conditional structures.

### Constant Propagation

Constant propagation replaces known constant signals with their actual values and simplifies the remaining logic.

For example:

```verilog
assign y = a ? b : 1'b0;
```

can be represented more simply as:

```verilog
assign y = a & b;
```

### Boolean Simplification

Equivalent Boolean expressions can be transformed into simpler logic.

For example:

```verilog
assign y = a ? 1'b1 : b;
```

can be reduced to:

```verilog
assign y = a | b;
```

### Redundant Logic Elimination

If a portion of the logic does not influence the required outputs, Yosys can identify and remove it.

---

## 5. Conditional Logic Simplification

Nested conditional statements may create unnecessary multiplexing or logic structures.

For example:

```verilog
assign y = a ? (c ? 1'b1 : b) : 1'b0;
```

can be simplified to:

```verilog
assign y = a & (c | b);
```

Another example:

```verilog
assign y = a ? (b ? (c ? 1'b1 : 1'b0) : 1'b0) : 1'b0;
```

is equivalent to:

```verilog
assign y = a & b & c;
```

### Observation

Although the RTL may contain nested conditions, the synthesis tool can analyze their Boolean behavior and generate a much simpler hardware structure.

---

## 6. Sequential RTL Optimization

Sequential optimization is applied to designs containing memory elements such as registers and flip-flops.

Yosys analyzes different aspects of sequential logic, including:

* Register assignments
* Clock behavior
* Reset conditions
* State transitions
* Output connections
* State-element usage

The purpose is to remove or simplify sequential hardware without affecting the required operation.

### Constant-Driven Flip-Flops

When a flip-flop is continuously assigned a fixed value and its storage behavior is not needed, it may be replaced with constant logic.

However, the synthesis tool must also examine reset and set behavior before removing a sequential element.

---

## 7. Sequential Constant Propagation and State Reduction

Sequential constant propagation examines register values across clock transitions to determine whether they become fixed.

For example, if a register is always driven to:

```text
q = 1
```

the synthesis tool can recognize that storing the value in a flip-flop may be unnecessary.

### Removal of Unused State

A register that cannot influence any required output may also be eliminated.

This is an important part of sequential optimization.

### Important Principle

> Hardware that has no effect on the observable outputs can potentially be removed during synthesis.

---

## 8. Counter Optimization

Counters provide a useful example of state optimization.

### Counter with Unused State Bits

Consider a 3-bit counter:

```text
count[2:0]
```

If the design uses only:

```text
count[0]
```

as an output, the other state bits may not be necessary for the required observable behavior.

The optimization concept can be represented as:

```text
3-bit Counter
      ↓
Only count[0] is observed
      ↓
Unused state identified
      ↓
Unnecessary state may be eliminated
```

### Counter with All Bits Used

Consider:

```verilog
assign q = (count[2:0] == 3'b100);
```

Here, the complete counter value is required to determine the output.

Therefore:

```text
count[2]
count[1]
count[0]
```

must all be retained.

### Key Observation

State elements are removed only when the synthesis tool determines that they are not required for the observable behavior of the circuit.

---

## 9. Advanced Sequential Optimization

### State Optimization

State optimization attempts to reduce unnecessary state representation while preserving the required sequential behavior.

This can reduce the number of flip-flops and simplify the associated logic.

### Retiming

Retiming changes the placement of flip-flops around combinational logic without changing the functional behavior.

It is mainly used to improve timing by balancing logic between sequential stages.

### Sequential Logic Cloning

Sequential cloning creates additional copies of sequential logic when this helps reduce fanout or improve timing and implementation.

The duplicated elements perform equivalent operations while allowing different sections of the design to use separate copies.

---

## 10. Observability Analysis

Observability determines whether an internal signal or state element can influence a required output.

The synthesis tool can remove internal logic when it proves that the logic cannot affect any observable result.

```text
Internal Signal
      ↓
Does it affect an output?
      ↓
   ┌──┴──┐
   │     │
  No    Yes
   │     │
   ▼     ▼
Remove  Retain
```

This explains why some registers or logic blocks may disappear after synthesis.

---

## 11. Yosys Optimization Commands

| Command            | Function                                                 |
| ------------------ | -------------------------------------------------------- |
| `opt`              | Executes a set of general optimization passes            |
| `opt_expr`         | Simplifies expressions and performs constant propagation |
| `opt_clean -purge` | Deletes unused and unreferenced cells and wires          |
| `dfflibmap`        | Converts generic flip-flops into library-specific cells  |
| `abc`              | Optimizes and maps combinational logic to standard cells |
| `show`             | Opens a schematic view of the synthesized design         |

### Commands Used

```text
opt
```

```text
opt_expr
```

```text
opt_clean -purge
```

```text
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

```text
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

```text
show
```

---

## 12. SKY130 Technology Mapping

Once the RTL has been optimized, it can be converted into actual cells available in the SKY130 standard-cell library.

### Flip-Flop Cell Mapping

```text
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps generic sequential elements to suitable SKY130 flip-flop cells.

### Combinational Cell Mapping

```text
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC performs logic optimization and maps the combinational portion of the design to available standard cells.

### Schematic Visualization

```text
show
```

This command displays the synthesized circuit structure.

---

## 13. Optimization Experiments

The following examples were studied during the optimization exercises.

### Combinational Designs

* `opt_check.v` – Demonstrates constant propagation
* `opt_check2.v` – Shows optimization using fixed logic values
* `opt_check3.v` – Demonstrates simplification of nested conditions
* `opt_check4.v` – Shows multi-level Boolean logic reduction

### Sequential Designs

* `dff_const1.v` – Examines a constant-related D flip-flop
* `dff_const2.v` – Demonstrates a flip-flop with a permanently fixed value
* `dff_const3.v` – Demonstrates propagation through multiple flip-flops

### Counter Designs

* `counter_opt.v` – Demonstrates optimization of unused counter bits
* `counter_opt2.v` – Demonstrates a counter where all state bits are needed

---

## 14. Learning Outcomes

After completing Day 3, I gained an understanding of:

* The importance of RTL optimization in digital design.
* The difference between combinational and sequential optimization.
* How constant propagation reduces unnecessary logic.
* How Boolean expressions can be simplified during synthesis.
* How redundant conditional structures can be minimized.
* How unused hardware can be detected and removed.
* How flip-flops can be optimized when their stored values are unnecessary.
* Why reset behavior must be considered during sequential optimization.
* How output observability affects state-element removal.
* Why some counter bits can be eliminated while others must remain.
* The basic concepts of state optimization and retiming.
* The purpose of sequential logic cloning.
* The functions of `opt`, `opt_expr` and `opt_clean -purge`.
* How `dfflibmap` performs flip-flop technology mapping.
* How ABC maps optimized logic to SKY130 standard cells.

---

## 15. Conclusion

Module-3 provided practical experience with the optimization of both **combinational and sequential RTL designs** using Yosys.

The experiments demonstrated how synthesis can use **constant propagation, Boolean simplification, redundant logic removal, state reduction and observability analysis** to create a more efficient hardware implementation.

The session also introduced advanced sequential techniques such as **state optimization, retiming and sequential logic cloning**, which are useful for improving implementation efficiency and timing.

The overall process can be summarized as:

```text
RTL Source
    ↓
RTL Synthesis
    ↓
Logic Optimization
    ↓
Redundant Hardware Removal
    ↓
Sequential Optimization
    ↓
Technology Mapping
    ↓
SKY130 Standard Cells
    ↓
Optimized Hardware
```

This exercise improved my understanding of how synthesis tools interpret RTL code, optimize the resulting logic and finally convert it into technology-specific hardware using the SKY130 standard-cell library.
