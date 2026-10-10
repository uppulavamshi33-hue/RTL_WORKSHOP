# Day 7 — Floorplan Design, Cell Placement & Library Characterization

> **ASIC Physical Design | OpenLane | SKY130 | `picorv32a` | Floorplanning and Placement**

## Overview

This module explores how a synthesized circuit is given a physical footprint and how its standard cells are arranged inside that footprint. It follows the synthesis work from Day 6 and focuses on floorplan dimensions, utilization, fixed blocks, decoupling cells, power distribution, I/O pins, placement restrictions, and placement optimization. It also introduces the library characterization process that supplies timing, power, and noise information to EDA tools.

The practical exercises use OpenLane with the `picorv32a` design and inspect the generated floorplan and placement database in Magic. The screenshots from the original notes are retained below as visual references.

## Contents

1. Die and Core Geometry
2. Fixed and Pre-Placed Blocks
3. Decoupling Cells and Signal Noise
4. Power Distribution Planning
5. I/O Pin Arrangement
6. Placement Blockages
7. Cell Mapping, Placement, and Optimization
8. Standard-Cell Characterization
9. OpenLane Lab Walkthrough
10. Summary

---

## 1. Die and Core Geometry

Floorplanning establishes the physical boundaries for a chip. The **die** is the overall silicon outline, while the **core** is the region reserved mainly for the design's logic cells. Space around the core may be needed for I/O structures, power routing, and other implementation requirements.

Two common floorplan settings are:

- **Aspect ratio:** core height divided by core width. A value of 1 corresponds to a square core; other values produce a rectangle.
- **Core utilization:** the proportion of core area occupied by placed logic cells.

\[
\text{Utilization (\%)} =
\frac{\text{Total standard-cell area}}{\text{Core area}}\times100
\]

A moderate target utilization leaves room for signal routing, clock buffers, and optimization. If cells are packed too tightly, congestion can increase and later physical-design steps may become harder. The suitable value depends on the design, library, and flow configuration; the 50–60% range is a common starting point, not a universal rule.

## 2. Fixed and Pre-Placed Blocks

Some components need a known physical location before the main standard-cell placement process begins. These may include memory macros, reusable IP, or other blocks whose dimensions and interface pins are already defined.

Such blocks are often called **pre-placed blocks**. Their location and orientation are selected during floorplanning, and the placement engine arranges ordinary standard cells around the reserved area. The physical boundary and pin locations of each block must be respected during routing.

The original notes discuss larger reusable blocks and fixed-location objects; in practice, whether an item is treated as a hard macro, a soft block, or a standard cell depends on the implementation flow.

## 3. Decoupling Cells and Signal Noise

Digital inputs are interpreted using voltage ranges for logic low and logic high. Between those valid ranges is a region where a receiver may not reliably distinguish a 0 from a 1. Supply disturbances can reduce noise margin and make the circuit less robust.

One source of supply disturbance is **IR drop**: current flowing through resistance in the power network causes a voltage reduction. When many gates switch at nearly the same time, the local supply can temporarily sag.

**Decoupling capacitors (decaps)** provide a nearby reservoir of charge. They can supply short bursts of current locally and then recharge from the power network. This helps reduce rapid supply variations. Decap cells may be inserted into available areas during floorplanning or other implementation stages, according to the flow.

## 4. Power Distribution Planning

The power network must deliver VDD and ground to many cells while keeping voltage variation within acceptable limits. A design that relies on a narrow or poorly distributed path may experience supply drop or ground bounce.

A power-distribution network commonly uses interconnected rings and straps to carry supply current across the die and core.

```text
Power source
    |
    v
Power / ground connections
    |
    v
Core power rings
    |
    v
Power straps and rails
    |
    v
Standard-cell power pins
```

The power network's geometry and layer selection affect resistance, current capacity, and the space available for signal routing. The exact structure is technology- and configuration-dependent.

## 5. I/O Pin Arrangement

RTL describes logical connections but does not normally define where the corresponding I/O pins should sit on the die. **Pin placement** assigns physical locations to the interface pins, often along the chip boundary.

Pin locations should be selected with signal connectivity, external interfaces, and any fixed macros in mind. A thoughtful pin arrangement can reduce detours in routing and make connections to internal logic more manageable.

## 6. Placement Blockages

A placement blockage marks a region where the automatic placer should not put ordinary standard cells, or where placement is restricted according to the blockage type. Blockages can preserve space for macros, routing access, power structures, or other physical requirements.

They give the floorplan designer more control over the available placement area. A blockage does not automatically create a route or solve congestion; it simply constrains where the placer may locate cells.

## 7. Cell Mapping, Placement, and Optimization

After the floorplan is established, the physical-design flow assigns library cells to locations within the core.

### 7.1 Connecting the netlist to library cells

The synthesized netlist refers to logic functions. Technology mapping associates those functions with cells from the selected standard-cell library. The library provides physical information as well as electrical and timing characteristics.

The same logic function may be available in several drive strengths. A stronger cell can often drive a larger load or reduce delay, but it may also consume more area and power. The tool selects cells based on optimization goals and constraints.

### 7.2 Initial placement

Placement determines where each mapped standard-cell instance sits within the core. The result converts the netlist from a connectivity description into a physical arrangement. Placement quality affects wire length, congestion, and timing.

### 7.3 Placement optimization

The placer estimates connections and timing, then may adjust cell positions or cell sizes and insert buffers. Long or heavily loaded nets can require buffering to improve transition times and reduce delay.

```text
Driver cell ───── Buffer ───── Receiver cell
```

Global placement first estimates an arrangement; legalization and detailed placement then enforce placement constraints and refine the cell locations. The exact sequence depends on the tool and flow version.

## 8. Standard-Cell Characterization

### 8.1 Why characterization is needed

Synthesis, placement, and static timing analysis rely on models describing how library cells behave. Characterization measures or simulates each cell under defined conditions to produce timing, power, and sometimes noise information.

Two timing-model approaches referenced in the notes are:

- **NLDM (Non-Linear Delay Model)**
- **CCS (Composite Current Source)**

These models capture cell behavior across different input transition times and output loads. The precise models available depend on the library.

### 8.2 Standard-cell development

A typical cell-development task draws on the PDK, process rules, transistor models, cell specifications, and reference libraries.

A simplified sequence is:

1. Design the transistor-level circuit.
2. Create the physical layout.
3. Check the layout against design rules and connectivity requirements.
4. Extract parasitic resistance and capacitance.
5. Generate the required circuit and physical views.
6. Characterize the cell and produce timing/power models.

Typical outputs can include a CDL netlist, GDSII layout, LEF physical abstract, extracted SPICE netlist, and Liberty (`.lib`) models. The exact deliverables vary with the library methodology.

### 8.3 Characterization procedure

The original notes describe a characterization setup using a tool such as GUNA. In broad terms, a characterization deck needs the following information:

1. Process and SPICE model files.
2. The extracted SPICE netlist for the cell.
3. The cell's logical function.
4. The subcircuit definition.
5. Power-supply connections.
6. Input stimulus patterns.
7. Output load capacitance.
8. A simulation setup, often including transient analysis.

The tool runs the required simulations and measures the responses used to construct cell models.

### 8.4 Delay and transition-time measurements

Timing measurements use specified voltage thresholds for input and output edges. The names in the source notes include `in_rise_thr`, `in_fall_thr`, `out_rise_thr`, and `out_fall_thr` for delay, and `slew_low_rise_thr`, `slew_high_rise_thr`, `slew_low_fall_thr`, and `slew_high_fall_thr` for transition measurements.

Propagation delay is measured as the difference between an output threshold-crossing time and the corresponding input threshold-crossing time:

```text
Propagation delay = output crossing time − input crossing time
```

Transition time (slew) measures the time taken for a signal to move between two defined voltage thresholds. For example:

```text
Rise transition = high-rise threshold time − low-rise threshold time
Fall transition = low-fall threshold time − high-fall threshold time
```

Threshold definitions and measurement conventions must be applied consistently. Poorly chosen or mismatched thresholds can produce misleading measurements.

---

## 9. OpenLane Lab Walkthrough

The lab continues the `picorv32a` flow used in the earlier synthesis session. The steps below cover examining the configuration, generating a floorplan, reviewing the resulting files, opening the layout in Magic, and then running placement.

### 9.1 Review floorplan configuration options

OpenLane's configuration documentation describes required design and clock settings as well as optional floorplanning variables. The installation's directory structure and filenames can vary by release.

![OpenLane configuration directory](https://github.com/user-attachments/assets/d7b9fda4-179f-44d3-9df7-efb9d77118ba)

The configuration documentation includes variables such as `DESIGN_NAME`, `VERILOG_FILES`, `CLOCK_PERIOD`, `CLOCK_PORT`, and `CLOCK_NET`.

![OpenLane configuration README](https://github.com/user-attachments/assets/0c5a338f-81b6-4b8e-b5da-beeff6edd847)

Floorplan-related settings can include `FP_CORE_UTIL`, `FP_ASPECT_RATIO`, `FP_SIZING`, `DIE_AREA`, I/O metal-layer choices, and power-distribution pitch or offset parameters.

![Floorplan variables in documentation](https://github.com/user-attachments/assets/006d727e-15eb-4a2d-ab56-9854436afca9)

The floorplan Tcl configuration defines default values that can be changed by design-specific settings.

![Floorplan Tcl defaults](https://github.com/user-attachments/assets/5fd4d082-6bd9-4f0b-975d-e2dcfa1a2a76)

### 9.2 Generate the floorplan

From the appropriate OpenLane interactive session, run:

```tcl
run_floorplan
```

This stage prepares the floorplan representation and may perform operations such as converting the netlist to DEF, inserting tap cells, assigning I/O pin locations, and building the power-distribution network.

![Floorplan stage completed](https://github.com/user-attachments/assets/059c143c-3e41-4976-9cec-9801f2ab2b26)

Check the console output for errors and confirm that the stage completed before opening its output files.

### 9.3 Inspect floorplan logs

The run directory contains stage-specific logs for floorplanning sub-steps. The I/O placer log can indicate which LEF/DEF inputs were loaded, how many pins were handled, and which pin-distribution strategy was used.

![Floorplan command output](https://github.com/user-attachments/assets/f8884b71-7a39-41a6-83d3-1a68d5e351e1)

![I/O placer log](https://github.com/user-attachments/assets/a266a7ed-4e90-49ce-b46f-67d26f00975d)

Logs are useful when the generated layout differs from expectations: they show which operations ran and can reveal missing inputs or configuration problems.

### 9.4 Check the selected utilization

In the run described by the original notes, the SKY130 standard-cell configuration overrides the generic 50% core-utilization default with a target of **35%**. The resolved run configuration should be inspected to verify which value was actually applied.

![SKY130 configuration override](https://github.com/user-attachments/assets/b2b384c4-b3a4-40fc-ad25-5193eafae478)

The expanded configuration for that run shows `FP_CORE_UTIL` set to `"35"`.

![Resolved floorplan utilization setting](https://github.com/user-attachments/assets/6d455ce8-d7db-426f-9846-7529fbfa5007)

These are values from the reference run, not guaranteed defaults for every installation. Always check your own run's configuration.

### 9.5 Read the generated floorplan DEF

The floorplan DEF records the generated physical design information, including die boundaries and placement rows. Coordinates are expressed in database units. In the reference file, the units statement specifies 1000 database units per micron.

The example die coordinates `(0 0) (660685 671405)` therefore correspond approximately to **660.7 µm × 671.4 µm**.

![Floorplan DEF and die dimensions](https://github.com/user-attachments/assets/6885cbdb-a401-4b2d-a0b3-ce349e60eda1)

Your own dimensions can differ if the RTL, configuration, library, or tool version changes.

### 9.6 Open the floorplan in Magic

The DEF can be viewed with the merged LEF and the technology file appropriate to the installed SKY130 PDK. The following command illustrates the form used in the original lab; update paths to match your machine and run directory.

```bash
magic -T /path/to/sky130A.tech \
  lef read /path/to/merged.lef \
  def read picorv32a.floorplan.def &
```

![Floorplan displayed in Magic](https://github.com/user-attachments/assets/f48d0eee-9a8e-4ddb-af57-670cfd377db8)

### 9.7 Identify pins, tap cells, and decaps

At floorplan stage, the core may still look mostly open because ordinary logic cells have not yet been placed. Visible features can include row structures, I/O pin locations, and tap or decap cells inserted by the flow.

![Overall floorplan view](https://github.com/user-attachments/assets/df57b424-31e7-4544-914c-d75799979b48)

A zoomed view in the reference notes identifies a decap instance such as `sky130_fd_sc_hd__decap_3` and an I/O signal such as `mem_la_wstrb[0]` on a metal layer.

![Floorplan close-up with decap and pin](https://github.com/user-attachments/assets/7819f5ce-bf42-4f1b-bfd2-5fe5cb9621c1)

Other labels in the example include `mem_rdata[3]` and `trace_data[26]`, shown near metal2/metal3 pin geometry.

![Additional floorplan pin detail](https://github.com/user-attachments/assets/2fdc2ae3-e38f-4639-80bf-2dd61e79273f)

The original lab also inspects a fixed buffer instance (`sky130_fd_sc_hd__buf_1`) in the Magic console. This is still the floorplan-stage view rather than the completed standard-cell placement.

![Selected cell in floorplan](https://github.com/user-attachments/assets/2120b95d-9017-450e-98c4-fd1d53981e95)

### 9.8 Run placement

Once floorplanning is complete, start the placement stage:

```tcl
run_placement
```

Placement performs global placement and detailed/legalized placement, followed by optimization steps supported by the selected flow. The resulting report can include instance and net counts, design area, utilization, row count, and wire-length estimates.

The reference run reports approximately 21,699 instances, 15,449 nets, a design area of about 420,473 µm², and roughly 36% utilization (around 55% when padding is counted). These numbers describe the source run and may not match a new run exactly.

![Placement stage completion](https://github.com/user-attachments/assets/9faa4529-33ad-4b0c-9a33-6ebabef5f148)

### 9.9 Open the placement result in Magic

The placement DEF represents the updated cell arrangement. Open it using the same technology and merged LEF, replacing the paths with those used in your environment.

```bash
magic -T /path/to/sky130A.tech \
  lef read /path/to/merged.lef \
  def read picorv32a.placement.def &
```

![Placement result opened in Magic](https://github.com/user-attachments/assets/4cc30dba-9f40-4788-bc75-82c1dfc1499e)

### 9.10 Compare the complete and zoomed placement views

After placement, the core appears much more filled than it did in the floorplan view because standard-cell instances now occupy their assigned rows and locations.

![Full placement layout](https://github.com/user-attachments/assets/598b0149-b998-403a-8260-efc683fd3216)

A detailed view can reveal individual instances such as flip-flops (`dfxtp_4`), multiplexers (`mux2_8`), buffers, and AND-OR-invert cells.

![Detailed placement layout](https://github.com/user-attachments/assets/63fd7b48-0ca0-46b4-8a7d-f6c5085ca16b)

Inspecting both views helps connect the abstract netlist to the actual arrangement of physical cells. It is also useful for spotting dense regions and understanding how the placer uses the core area.

## 10. Summary

This session follows synthesis into the physical implementation stages of floorplanning and placement. Floorplanning sets die/core geometry, reserves space for fixed blocks, plans the power network, positions I/O pins, and establishes placement constraints. Placement then assigns standard cells to physical locations and may optimize cell sizes, positions, and buffering to improve timing and routability.

The library-characterization section explains where the timing, power, and noise models used by EDA tools originate. In the practical exercise, `picorv32a` is prepared in OpenLane, the floorplan is generated and examined in Magic, and the placement stage is run and reviewed.

The main learning outcome is to recognize the difference between a logical netlist and a physical implementation, and to understand how utilization, power delivery, pin arrangement, cell libraries, and optimization influence the resulting layout. Record actual values and tool messages from your own run when reproducing this exercise.
