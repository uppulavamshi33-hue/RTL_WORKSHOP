# Module 9 — Pre-Layout Timing Analysis and the Importance of Clock Tree Design

This lab explores how to prepare a custom standard cell for the OpenLane RTL-to-GDSII flow. The main activities include converting layout grid information into routing tracks, defining cell pins, generating a LEF abstract, integrating custom cells into OpenLane, understanding Liberty timing libraries, and estimating clock-buffer delays before physical layout.

---

## 1. Converting Magic Grid Information into Routing Track Information

### Understanding the routing grid

Magic provides the layout grid details through its `tkcon` console. For the loaded cell, the following command reports the grid spacing and offsets:

```tcl
% grid 0.46um 0.34um 0.23um 0.17um
```

The values represent the horizontal and vertical grid pitches along with the corresponding offsets. These coordinates determine where routing features can be placed in the layout.

OpenLane requires this information in a different format. The routing-track definition must specify the metal layer, routing direction, offset, and pitch.

### Example `tracks.info` configuration

```text
li1  X 0.23 0.46
li1  Y 0.17 0.34
met1 X 0.17 0.34
met1 Y 0.17 0.34
met2 X 0.23 0.46
met2 Y 0.23 0.46
met3 X 0.34 0.68
met3 Y 0.34 0.68
met4 X 0.46 0.92
met4 Y 0.46 0.92
met5 X 1.70 3.40
met5 Y 1.70 3.40
```

**General format:**

```text
<layer> <direction> <offset> <pitch>
```

The track definitions must match the intended routing grid. Custom-cell pins should align with the appropriate routing tracks so that OpenLane can connect them during routing without introducing design-rule violations.

### Defining cell ports in Magic

The inverter layout contains four important pins:

- `VPWR` — power connection
- `VGND` — ground connection
- `A` — input signal
- `Y` — output signal

The labels are created using Magic's text or label tool. Each label must be attached to the correct physical layer and configured as a port. The port option and its index allow Magic to recognize the label as an actual cell connection rather than ordinary text.

---

## 2. Assigning Port Properties and Generating a Standard-Cell LEF

Before exporting the layout as a LEF file, each port must have a valid direction and usage. These properties can be configured in Magic's `tkcon` console.

### Configuring the ports

**Ground pin — `VGND`**

```tcl
% what
% port class inout
% port use ground
```

**Power pin — `VPWR`**

```tcl
% what
% port class inout
% port use power
```

**Input pin — `A`**

```tcl
% what
% port class input
% port use signal
```

**Output pin — `Y`**

```tcl
% what
% port class output
% port use signal
```

The `what` command helps identify the selected layer and label before assigning the corresponding port properties.

### Port configuration summary

| Pin    | Physical layer | Port class | Port use |
| ------ | -------------- | ---------- | -------- |
| `VGND` | metal1         | inout      | ground   |
| `VPWR` | metal1         | inout      | power    |
| `A`    | locali (li1)   | input      | signal   |
| `Y`    | locali (li1)   | output     | signal   |

### Example of a standard-cell LEF file

LEF stands for **Library Exchange Format**. It describes the physical information needed by placement and routing tools, including the cell boundary, placement site, pin locations, and routing layers.

The following is an example excerpt from `sky130_inv.lef`:

```lef
VERSION 5.7 ;
NOWIREEXTENSIONATPIN ON ;
DIVIDERCHAR "/" ;
BUSBITCHARS "[]" ;

MACRO sky130_inv
    CLASS CORE ;
    FOREIGN sky130_inv ;
    ORIGIN 0.000 0.000 ;
    SIZE 1.380 BY 2.720 ;
    SITE unithd ;

    PIN A
        DIRECTION INPUT ;
        USE SIGNAL ;
        ANTENNAGATEAREA 0.165600 ;
        PORT
            LAYER li1 ;
                RECT 0.060 1.180 0.510 1.690 ;
        END
    END A

    PIN Y
        DIRECTION OUTPUT ;
        USE SIGNAL ;
        ANTENNADIFFAREA 0.287800 ;
        PORT
            LAYER li1 ;
                RECT 0.760 1.960 1.100 2.330 ;
                RECT 0.880 1.690 1.050 1.960 ;
                RECT 0.880 1.180 1.330 1.690 ;
                RECT 0.880 0.760 1.050 1.180 ;
                RECT 0.780 0.410 1.130 0.760 ;
        END
    END Y

    PIN VPWR
        DIRECTION INOUT ;
        USE POWER ;
        PORT
            LAYER nwell ;
                RECT -0.200 1.140 1.570 3.040 ;
            LAYER li1 ;
                RECT -0.200 2.580 1.430 2.900 ;
                RECT 0.180 2.330 0.350 2.580 ;
                RECT 0.100 1.970 0.440 2.330 ;
            LAYER mcon ;
                RECT 0.230 2.640 0.400 2.810 ;
                RECT 1.000 2.650 1.170 2.820 ;
        ...
```

*Note: This is an illustrative excerpt, not a complete LEF file. The original example continues with additional geometry and pins.*

### Important LEF attributes

- **`CLASS CORE`** identifies the cell as a standard core cell suitable for placement.
- **`SIZE 1.380 BY 2.720`** defines the cell's width and height in micrometres.
- **`SITE unithd`** associates the cell with a standard placement site.
- **`PIN`, `PORT`, `LAYER`, and `RECT`** specify the pins and their physical connection areas.
- **`ANTENNAGATEAREA` and `ANTENNADIFFAREA`** provide antenna-related area information used by physical-design checks.

The LEF abstract allows physical-design tools to understand how the cell occupies space and where its pins can be accessed.

---

## 3. Integrating a Custom Cell into OpenLane

The next activity demonstrates how to prepare a design in OpenLane and add a custom standard-cell LEF.

### Launching OpenLane interactively

```tcl
./flow.tcl -interactive
package require openlane 0.9

prep -design picorv32a -tag 15-09_06-28 -overwrite
```

During preparation, OpenLane reads the design configuration and merges the standard-cell LEF files. The example log reports 437 standard-cell macros in `sky130_fd_sc_hd.lef`.

The preparation process also determines the placement-site dimensions. Some special-purpose cells, such as tap, decoupling, and filler cells, are excluded from ordinary LEF padding because their physical placement requirements differ from those of normal standard cells.

### Adding a custom LEF

Place the custom LEF file in the design's source directory and load it into the interactive OpenLane session.

```tcl
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
```

Example source file:

```text
/openLANE_flow/designs/picorv32a/src/sky130_vsdinv.lef
```

Merge the custom LEF using:

```tcl
add_lefs -src $lefs
```

OpenLane can then include the custom cell's physical abstract in the placement and routing environment.

**Important:** A LEF file describes physical geometry; it does not, by itself, provide the timing and power models needed for timing-aware synthesis and static timing analysis.

---

## 4. Introduction to Liberty Timing Libraries

Liberty files (`.lib`) describe the electrical, timing, and power characteristics of standard cells. Synthesis and STA tools use these models to estimate delays, check timing requirements, and evaluate power-related properties.

An example library used in the lab is:

`sky130_fd_sc_hd_tt_025C_1v80.lib`

### Example Liberty header

```liberty
library ("sky130_fd_sc_hd_tt_025C_1v80") {
    technology("cmos");
    delay_model : "table_lookup";
    bus_naming_style : "%s[%d]";
    time_unit : "1ns";
    voltage_unit : "1V";
    leakage_power_unit : "1nW";
    current_unit : "1mA";
    pulling_resistance_unit : "1kohm";
    capacitive_load_unit(1.0000000000, "pf");

    default_max_transition : 1.5000000000;
    default_arc_mode : "worst_edges";
    default_constraint_arc_mode : "worst";

    operating_conditions ("tt_025C_1v80") {
        voltage     : 1.8000000000;
        process     : 1.0000000000;
        temperature : 25.000000000;
        tree_type   : "balanced_tree";
    }

    power_lut_template ("power_inputs_1") {
        variable_1 : "input_transition_time";
        index_1("1, 2, 3, 4, 5, 6, 7");
    }

    power_lut_template ("power_outputs_1") {
        variable_1 : "input_transition_time";
        variable_2 : "total_output_net_capacitance";
        index_1("1, 2, 3, 4, 5, 6, 7");
        index_2("1, 2, 3, 4, 5, 6, 7");
    }
}
```

### Understanding the library fields

- **`delay_model : "table_lookup"`** indicates that timing is represented using lookup tables.
- **`operating_conditions`** defines the process, voltage, and temperature conditions for the library. Here, `tt_025C_1v80` represents the typical-process, 25 °C, 1.8 V condition.
- **`time_unit` and `voltage_unit`** establish the units used in the library.
- **`power_lut_template`** specifies the indexing variables used by power lookup tables.
- **`input_transition_time`** represents input slew.
- **`total_output_net_capacitance`** represents the capacitive load driven by the output.

For a custom cell to be fully supported by synthesis and STA, its Liberty model must provide the appropriate cell definition and characterized timing and power information, in addition to its physical LEF.

---

## 5. Delay Tables and Clock-Tree Synthesis

### What is a clock tree?

A clock tree distributes the clock signal from its source to the sequential elements in a digital design. Buffers are inserted at different levels to handle fanout and load, while the clock-tree synthesis process aims to control delay and clock skew.

The lab compares two simplified clock-distribution structures.

**Ordinary buffer tree**

A first-level buffer drives two second-level buffers, `Cbuf1` and `Cbuf2`. Each second-level buffer drives two loads.

**Clock-gated tree**

One branch uses an ordinary buffer, while another branch contains a clock-gating AND cell, `Cand1`, controlled by an enable signal `EN`. When the relevant logic is inactive, the clock-gating cell can prevent clock transitions from propagating through that branch, reducing dynamic power.

Clock gating must be implemented with appropriate clock-gating structures and timing checks. The example illustrates the principle rather than a complete CTS implementation.

### How a delay lookup table works

A standard-cell delay table typically uses two dimensions:

1. **Input slew:** represented by the rows.
2. **Output load capacitance:** represented by the columns.

Each table entry contains a characterized delay for the corresponding input transition and output load. When the requested operating point falls between table entries, the timing tool can interpolate between neighbouring values.

### Example delay table for `CBUF '1'`

| Input slew / Load | 10 fF | 30 fF |  50 fF |   70 fF | 90 fF | 110 fF |
| ----------------- | ----: | ----: | -----: | ------: | ----: | -----: |
| 20 ps             |    x1 |    x2 |     x3 |      x4 |    x5 |     x6 |
| 40 ps             |    x7 |    x8 | **x9** | **x10** |   x11 |    x12 |
| 60 ps             |   x13 |   x14 |    x15 |     x16 |   x17 |    x18 |
| 80 ps             |   x19 |   x20 |    x21 |     x22 |   x23 |    x24 |

### Example delay table for `CBUF '2'`

| Input slew / Load | 10 fF | 30 fF | 50 fF | 70 fF | 90 fF | 110 fF |
| ----------------- | ----: | ----: | ----: | ----: | ----: | -----: |
| 20 ps             |    y1 |    y2 |    y3 |    y4 |    y5 |     y6 |
| 40 ps             |    y7 |    y8 |    y9 |   y10 |   y11 |    y12 |
| 60 ps             |   y13 |   y14 |   y15 |   y16 |   y17 |    y18 |
| 80 ps             |   y19 |   y20 |   y21 |   y22 |   y23 |    y24 |

The symbols `x1`–`x24` and `y1`–`y24` are placeholders for delay values obtained through cell characterization. They are not actual numerical delay values.

---

## 6. Worked Example: Estimating Buffer-Tree Delay

Consider a buffer tree with one first-level buffer and two second-level buffers.

- The input signal reaches the first buffer with a slew of **40 ps**.
- The first buffer's output, node **A**, drives `Cbuf1` and `Cbuf2`.
- Each second-level buffer contributes a load of **30 fF** at node A.
- Therefore, the total load driven by the first buffer is:

[
C_A = 30\text{ fF} + 30\text{ fF} = 60\text{ fF}
]

Each second-level buffer drives two loads of 25 fF:

[
C_B = 25\text{ fF} + 25\text{ fF} = 50\text{ fF}
]

[
C_C = 25\text{ fF} + 25\text{ fF} = 50\text{ fF}
]

### Selecting the delay-table entries

**First-level buffer**

The input slew is 40 ps and the output load is 60 fF. The table has entries at 50 fF and 70 fF, so 60 fF falls between the two columns.

The timing tool therefore interpolates between **`x9`** and **`x10`** to estimate the delay.

**Second-level buffers**

Both `Cbuf1` and `Cbuf2` drive a 50 fF load. Since this value is directly represented in the table, the corresponding delay can be read from the 50 fF column for the applicable input slew.

### Why this calculation is simplified

Two useful observations reduce the number of calculations:

- **Identical buffers:** If both second-level buffers are the same cell type and size, the same delay model can be used for each.
- **Equal loads:** Since both branches drive equal capacitances, their corresponding delay lookups use the same operating point when their input slews are also equal.

This approach provides a preliminary estimate of buffer delay before the final physical layout and clock tree are available. Accurate timing analysis still depends on the actual characterized library values, parasitics, and implementation.

---

## 7. Key Learnings

- Magic grid spacing and offsets must be converted into the correct OpenLane routing-track format.
- Custom-cell pins need correctly assigned labels, physical layers, port directions, and port uses.
- A LEF file describes the physical geometry, placement dimensions, and connection locations of a standard cell.
- OpenLane can merge custom LEF files so the physical-design tools can recognize the cell.
- Liberty files provide the timing and power models required for timing-aware synthesis and STA.
- Delay lookup tables relate input slew and output capacitance to characterized cell delay.
- Pre-layout clock-tree analysis estimates the load at each buffer output and uses the appropriate delay-table entries.
- Clock gating can reduce dynamic power by suppressing unnecessary clock activity in selected branches.
- Actual sign-off results require valid library models and implementation-aware timing analysis.

**Conclusion:** This lab connects custom standard-cell layout with physical-design integration and pre-layout timing estimation. Understanding LEF, Liberty, routing tracks, delay tables, and clock-tree structure is essential for building a reliable RTL-to-GDSII flow.
