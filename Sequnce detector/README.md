# Sequence Detector: RTL to Gate-Level Simulation

## Overview

This project demonstrates the complete RTL-to-Gate-Level Simulation (GLS) flow for a Verilog sequence detector designed to recognize the binary pattern `1101011`.

The design is simulated before synthesis and again after being mapped to the SKY130 standard-cell library. The same testbench and input sequence are used for both simulations to compare their functional behavior.

According to the recorded simulation results, both RTL and GLS produce **5 detections**, with the first detection occurring at **188 ns**.

## Contents

1. RTL Design
2. Testbench Configuration
3. RTL Simulation
4. Synthesis Using Yosys
5. Synthesis Statistics
6. Synthesized Netlist Inspection
7. Gate-Level Simulation
8. RTL and GLS Comparison
9. Functional Verification
10. Conclusion

---

## 1. RTL Design

The sequence detector is implemented as a finite-state machine (FSM) that recognizes the target sequence `1101011`.

### Design specifications

| Parameter                | Description              |
| ------------------------ | ------------------------ |
| Target sequence          | `1101011`                |
| Number of logical states | 7                        |
| Declared state width     | 3 bits                   |
| Input signal             | `din`                    |
| Output signal            | `detected`               |
| Reset type               | Synchronous, active high |
| Detection style          | Mealy-style              |
| Overlapping matches      | Supported                |

The detector examines the serial input and asserts `detected` when the complete target pattern is recognized.

### State machine operation

The FSM advances through states as the input bits arrive. Each state represents progress toward matching the target sequence.

```text
             Input Bits
                 |
                 v
          Sequence Detector
                 |
                 v
            State Machine
                 |
                 v
          Pattern Recognized?
                 |
                 v
             detected
```

### Overlapping sequence detection

After a successful match, the FSM returns to state 2 rather than state 0 when the final input bit is `1`. This retains the relevant trailing bits of the matched pattern and allows overlapping occurrences to be detected.

The RTL declares a 3-bit state representation. However, synthesis may select a different internal state encoding while preserving the design's observable behavior.

---

## 2. Testbench Configuration

The testbench supplies the clock and reset, applies the input sequence one bit at a time, monitors the detection output, and reports the final detection count.

The same testbench is used for the pre-synthesis and post-synthesis simulations so that their results can be compared under identical stimulus.

### Important testbench parameters

| Parameter                | Value           |
| ------------------------ | --------------- |
| Clock generation         | `#4 clk = ~clk` |
| Clock period             | 8 ns            |
| Clock frequency          | 125 MHz         |
| Target pattern           | `1101011`       |
| Reset                    | Active high     |
| Initial reset duration   | 4 clock cycles  |
| Testbench file           | `tb/tb.v`       |
| Assessment instance      | `24eg104c52`    |
| Reported detection count | 5               |

The complete testbench, containing approximately 180 `drive_bit` calls, is stored in `tb/tb.v`.

The testbench records the detection pulses and provides the information needed to compare the two simulation stages.

---

## 3. Pre-Synthesis RTL Simulation

The original RTL design is simulated first to establish its expected functional behavior. Icarus Verilog compiles the design, and GTKWave is used to inspect the resulting waveform.

### Simulation commands

Compile the RTL and testbench:

```bash
iverilog sequence_detector.v ../tb/tb.v
```

Run the generated executable:

```bash
./a.out
```

Open the waveform:

```bash
gtkwave dump.vcd
```

### Expected waveform observations

The recorded waveform contains the following signals:

- `clk` — continuously running clock with an 8 ns period.
- `reset` — reset signal applied at the beginning and again near the end of the test.
- `din` — serial input carrying the test sequence.
- `detected` — output indicating a recognized pattern.
- `next_detected` — next-value logic for the detection signal.
- `state[2:0]` — current state in the RTL representation.
- `next_state[2:0]` — next-state value.

The FSM progresses through its states as the input bits are applied, and the detection output pulses when the required sequence is recognized.

### RTL GTKWave evidence

**Original image placeholder:** `RTL_SIM`

The supplied waveform description identifies the first detection at **188 ns**. This agrees with the reported hand-derived prediction for the input stream, where the first match completes at bit index 19.

The RTL simulation reports a total of five detections.

---

## 4. Synthesis Using Yosys

After RTL simulation, Yosys is used to synthesize the sequence detector and map its logic to the SKY130 high-density standard-cell library.

### Synthesis sequence

Read the SKY130 Liberty library:

```tcl
read_liberty -lib ../../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the RTL source:

```tcl
read_verilog sequence_detector.v
```

Synthesize the design:

```tcl
synth -top sequence_detector
```

Map sequential elements:

```tcl
dfflibmap -liberty ../../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Map combinational logic:

```tcl
abc -liberty ../../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Display the synthesized logic:

```tcl
show
```

Display synthesis statistics:

```tcl
stat
```

Write the synthesized netlist:

```tcl
write_verilog det_netlist.v
```

These operations transform the RTL description into a gate-level representation containing sequential cells and combinational logic.

---

## 5. Synthesis Statistics

The supplied Yosys statistics report summarizes the synthesized `sequence_detector` design.

### Overall design statistics

| Item             | Count |
| ---------------- | ----: |
| Wires            |    44 |
| Wire bits        |    50 |
| Public wires     |     5 |
| Public wire bits |    11 |
| Ports            |     4 |
| Port bits        |     4 |
| Memories         |     0 |
| Memory bits      |     0 |
| Processes        |     0 |
| Cells            |    20 |

### Cell breakdown

| Cell                        |  Count |
| --------------------------- | -----: |
| `$_DFF_P_`                  |      7 |
| `$_SDFF_PP0_`               |      1 |
| `sky130_fd_sc_hd__a21boi_0` |      1 |
| `sky130_fd_sc_hd__a21oi_1`  |      1 |
| `sky130_fd_sc_hd__and2_0`   |      3 |
| `sky130_fd_sc_hd__nand2b_1` |      1 |
| `sky130_fd_sc_hd__nor2_1`   |      2 |
| `sky130_fd_sc_hd__nor2b_1`  |      1 |
| `sky130_fd_sc_hd__nor3b_1`  |      2 |
| `sky130_fd_sc_hd__nor4_1`   |      1 |
| **Total**                   | **20** |

**Original screenshot placeholder:** `stat`

The supplied synthesis description reports seven `$_DFF_P_` cells and one `$_SDFF_PP0_` cell, together with thirteen combinational cells.

The sequential-cell breakdown and the netlist should be interpreted according to the actual Yosys output, since synthesis can alter the internal representation of the original RTL.

---

## 6. Synthesized Netlist Inspection

The generated file, `det_netlist.v`, represents the synthesized sequence detector. It can be examined to understand how the FSM has been translated into standard-cell logic.

### Logic diagram

The Yosys `show` command produces a graphical representation of the synthesized design.

The reported diagram includes:

- Input signals `din`, `reset`, and `clk`.
- Combinational gates used to implement next-state and detection logic.
- Sequential cells storing the FSM state.
- Logic connected to the `detected` output.
- Clock and reset connections distributed to the relevant sequential elements.

### Full synthesized logic diagram

**Original image placeholder:** `netlist_sequence_detector`

### Zoomed register/output view

**Original image placeholder:** `zoom view`

The zoomed view is intended to make the reset connection of the special sequential cell associated with `detected` easier to identify.

### Written-out netlist

**Original image placeholder:** `detnetlidt v`

The supplied netlist excerpt is:

```verilog
always @(posedge clk)
  if (_07_) detected <= 1'h0;
  else detected <= din;

always @(posedge clk) state[0] <= _06_;
always @(posedge clk) state[1] <= _00_;
always @(posedge clk) state[2] <= _01_;
always @(posedge clk) state[3] <= _02_;
always @(posedge clk) state[4] <= _03_;
always @(posedge clk) state[5] <= _04_;
always @(posedge clk) state[6] <= _05_;
```

This excerpt illustrates the sequential assignments reported in the supplied material. It should not be treated as a complete, standalone netlist.

### State encoding after synthesis

The supplied description reports a seven-bit internal state representation in the synthesized netlist, even though the RTL declares a 3-bit state register.

This is consistent with a possible one-hot-style implementation, in which individual state bits represent FSM states. Synthesis can choose a different encoding from the source RTL if the resulting implementation preserves the intended functionality.

---

## 7. Post-Synthesis Gate-Level Simulation

Once the synthesized netlist has been generated, the design is simulated using the same testbench and input sequence used for RTL simulation.

The gate-level simulation includes the synthesized netlist and the SKY130 Verilog models needed to simulate the mapped standard cells.

### GLS commands

Change to the RTL directory:

```bash
cd rtl
```

Compile the cell models, netlist, and testbench:

```bash
iverilog ../../my_lib/verilog_model/primitives.v ../../my_lib/verilog_model/sky130_fd_sc_hd.v det_netlist.v ../tb/tb.v
```

Run the simulation:

```bash
./a.out
```

Open the waveform:

```bash
gtkwave dump.vcd
```

### GLS waveform evidence

**Original image placeholder:** `GLS_`

The supplied results report that the GLS console output follows the same detection behavior as the RTL simulation. The example trace begins with:

```text
TIME=45000 NS DIN=1 DETECTED=0
```

The waveform is then inspected to check the detection pulses and the behavior of the relevant signals.

---

## 8. RTL and GLS Comparison

The same input sequence and testbench are used in both simulation stages. This provides a consistent basis for comparing the functional results.

### Comparison summary

| Parameter             | RTL Simulation | GLS       |
| --------------------- | -------------- | --------- |
| Target sequence       | `1101011`      | `1101011` |
| First detection       | 188 ns         | 188 ns    |
| Total detections      | 5              | 5         |
| Final detection count | 5              | 5         |

### Detection timing table

The supplied timing results identify five successful matches.

| Detection number | Bit index | Time (ns) |
| ---------------: | --------: | --------: |
|                1 |        19 |       188 |
|                2 |        30 |       276 |
|                3 |        49 |       428 |
|                4 |       111 |       924 |
|                5 |       145 |      1196 |

The reported first detection occurs at 188 ns in both simulations, and the total detection count is five in each case.

These results support the conclusion that the synthesized implementation preserves the observed sequence-detection behavior for the testbench stimulus.

---

## 9. Functional Verification

The overall verification process can be summarized as follows:

```text
         Input Sequence
                |
                v
      Sequence Detector FSM
                |
                v
      Target Pattern: 1101011
                |
                v
        Detection Pulses
                |
                v
       Five Reported Matches
                |
                v
         RTL Result = GLS Result
```

The GTKWave results and console output are used to compare the RTL and gate-level simulations.

The reported matching detection count and first-detection timestamp provide evidence that the synthesized design behaves consistently with the RTL for the supplied test sequence. This conclusion is limited to the testbench scenarios and waveforms that were examined.

---

## 10. Final Conclusion

The sequence detector was processed through RTL simulation, Yosys synthesis, SKY130 technology mapping, netlist generation, and post-synthesis gate-level simulation.

Both simulation stages are reported to detect the target sequence `1101011` five times, with the first detection occurring at 188 ns. The matching results demonstrate that the synthesized implementation preserves the observed functional behavior for the given testbench.

Although gate-level simulation may exhibit propagation delays depending on the simulation models and timing information used, the supplied results show consistent detection behavior between RTL and GLS.

## Result

| Verification item                           | Result    |
| ------------------------------------------- | --------- |
| Target sequence                             | `1101011` |
| RTL detections                              | 5         |
| GLS detections                              | 5         |
| RTL first detection                         | 188 ns    |
| GLS first detection                         | 188 ns    |
| Functional match for the supplied testbench | YES       |

**Final outcome:** The RTL and gate-level simulation results agree for the reported sequence detector test, with five detections and a first-detection timestamp of 188 ns.
