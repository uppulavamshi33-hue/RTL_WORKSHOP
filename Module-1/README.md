# Day 1 – RTL Design Using Verilog HDL

## Introduction

Day 1 focuses on the fundamentals of **RTL (Register Transfer Level) design** using **Verilog HDL**.

The main objective is to understand how an RTL design is written, simulated, tested, and verified using open-source tools.

The practical experiment is a **2:1 Multiplexer (MUX)**.

---

## Tools Used

* Verilog HDL
* Icarus Verilog
* GTKWave
* Linux/Ubuntu

---

## RTL Simulation Flow

```text
Verilog RTL Design
        ↓
    Testbench
        ↓
  Icarus Verilog
        ↓
    Simulation
        ↓
  VCD Waveform
        ↓
     GTKWave
        ↓
Waveform Verification
```

---

## Experiment: 2:1 Multiplexer

A 2:1 Multiplexer selects one of two inputs based on the select signal.

| Signal | Description       |
| ------ | ----------------- |
| `i0`   | First data input  |
| `i1`   | Second data input |
| `sel`  | Select signal     |
| `y`    | Output            |

### Logic

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

---

## Block Diagram

```text
             ┌─────────────┐
 i0 ────────►│             │
             │    2:1 MUX  ├──────► y
 i1 ────────►│             │
             │             │
 sel ───────►│             │
             └─────────────┘
```

---

## Verilog RTL Code

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

---

## Testbench

```verilog
module tb_good_mux;

reg i0;
reg i1;
reg sel;
wire y;

good_mux dut (
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin

    $dumpfile("tb_good_mux.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0;
    i1 = 1;
    sel = 0;
    #10;

    sel = 1;
    #10;

    i0 = 1;
    i1 = 0;
    sel = 0;
    #10;

    sel = 1;
    #10;

    $finish;

end

endmodule
```

---

## Simulation Commands

### Compile

```bash
iverilog good_mux.v tb_good_mux.v
```

### Run Simulation

```bash
./a.out
```

### Open Waveform

```bash
gtkwave tb_good_mux.vcd
```

---

## Expected Waveform

The waveform contains:

```text
i0
i1
sel
y
```

The output should follow the selected input:

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

---

## Truth Table

| `sel` | `i0` | `i1` | `y` |
| :---: | :--: | :--: | :-: |
|   0   |   0  |   1  |  0  |
|   0   |   1  |   0  |  1  |
|   1   |   0  |   1  |  1  |
|   1   |   1  |   0  |  0  |

---

## Key Concepts Learned

* RTL design using Verilog
* Verilog module structure
* Testbench development
* Icarus Verilog simulation
* VCD waveform generation
* GTKWave waveform analysis
* Functional verification
* 2:1 Multiplexer operation

---

## Day 1 Outcome

The complete RTL verification flow was practiced:

```text
Design
  ↓
Testbench
  ↓
Compile
  ↓
Simulate
  ↓
Generate VCD
  ↓
View Waveform
  ↓
Verify Output
```

---

## Conclusion

Day 1 provided a basic understanding of **RTL design and verification using Verilog HDL**. The 2:1 Multiplexer experiment demonstrated how to write RTL code, create a testbench, simulate the design, generate a waveform, and verify the output using GTKWave.
