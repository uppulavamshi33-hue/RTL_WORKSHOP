# Day 8 — CMOS Inverter Characterization and CMOS Manufacturing

## Overview

This module connects circuit-level behavior with the physical steps used to manufacture CMOS devices. It combines SPICE simulation concepts with a walkthrough of the 16-mask fabrication sequence, then applies both topics through practical exercises using OpenROAD-related floorplanning settings, Magic, and ngspice.

The main activities are:
- Building and understanding a SPICE deck for a CMOS inverter.
- Studying its DC switching point and transient timing behavior.
- Following the main stages of the 16-mask CMOS process.
- Inspecting a Sky130 inverter layout, extracting its circuit, correcting model references, and measuring timing.
- Investigating design-rule violations in Metal3 and polysilicon layers.

---

## 1. Understanding a SPICE Description

A SPICE deck records the circuit connections, the electrical values of its components, and the models that describe the devices. For an inverter, this usually means a PMOS transistor and an NMOS transistor, power and input sources, an output load, and commands for operating-point, DC, or transient analysis.

When a netlist is extracted from a Magic layout, geometry-derived information such as transistor dimensions and diffusion parasitics is added automatically. However, the extracted device names may not match the names expected by the model files, so the deck may need editing before simulation.

---

## 2. DC Transfer Curve and Switching Point

A CMOS inverter's DC transfer curve shows how the output voltage changes while the input is swept across the supply range. During this sweep, the PMOS and NMOS devices move through different combinations of cutoff, linear, and saturation operation.

The switching threshold, commonly written as \(V_M\), is the point where input and output voltages are equal. It indicates the balance between the pull-up and pull-down devices. Changing the relative transistor widths changes that balance: a stronger PMOS tends to move the switching point upward. Since hole mobility is lower than electron mobility in typical CMOS processes, PMOS devices are often made wider to help balance the inverter.

<img width="1920" height="1080" alt="vm" src="https://github.com/user-attachments/assets/e9041d27-91c5-47b2-8ff0-a5f3c3c2f63e" />

---

## 3. Transient Timing Measurements

DC analysis describes the voltage transfer relationship, while transient analysis measures how the circuit responds over time to a changing input. A pulse source and a `.tran` command can be used to observe the output waveform.

Two important measurements are:
- **Rise/fall transition time:** the time taken for the output to move between defined voltage levels, here measured from 20% to 80% of the swing.
- **Cell rise/fall delay:** the interval between the 50% input crossing and the corresponding 50% output crossing.

Transition time describes edge sharpness, while propagation delay is directly relevant to timing analysis of digital paths.

---

## 4. CMOS Fabrication Using 16 Masks

The fabrication sequence gradually builds transistor regions and metal connections on a silicon wafer. The stages below summarize the process covered in the module; individual stages can involve more than one mask.

### 4.1 Starting Wafer

The example begins with a p-type silicon wafer with a `<100>` crystal orientation. The substrate's doping and crystal properties influence the silicon–oxide interface and the eventual device behavior.

<img width="1920" height="1080" alt="1" src="https://github.com/user-attachments/assets/953a361a-1874-4d16-8515-3df920358082" />

### 4.2 Defining Active Areas — Mask 1

Oxide, silicon nitride, and photoresist layers are prepared and patterned. The patterned nitride protects selected areas while field oxide grows in exposed regions through LOCOS. This can create a tapered oxide edge known as the bird's beak. The nitride is then removed.

<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/ddfe6b9e-19f3-4517-811d-de998b7ee8f5" />

### 4.3 Forming the Wells — Mask 2

Boron and phosphorus implantation steps create the p-well and n-well regions needed for NMOS and PMOS transistors on the same wafer. A high-temperature drive-in process moves the dopants into the substrate and activates them.

<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/4af1f527-d194-406e-8b67-2665d2199856" />
<img width="1920" height="1080" alt="5" src="https://github.com/user-attachments/assets/4fa34611-6536-4b76-aad8-7a67c158b665" />

### 4.4 Gate Stack — Mask 6

The gate region is prepared after sacrificial oxide removal and threshold-adjust implants. Doped polysilicon is deposited, and Mask 6 patterns it into the transistor gate. The substrate doping and gate-oxide capacitance contribute to the threshold voltage.

<img width="1920" height="1080" alt="6" src="https://github.com/user-attachments/assets/2cc59149-f2c0-48a9-ba99-90a2b0f350ea" />

### 4.5 Lightly Doped Drain Regions — Masks 7 and 8

Lower-dose implants form the lightly doped regions near the gate. These help reduce hot-carrier effects and short-channel behavior. Sidewall spacers are then formed beside the gate so that the heavier source/drain implants are offset from the channel edge.

<img width="1920" height="1080" alt="8" src="https://github.com/user-attachments/assets/3694ecbb-c917-4087-b735-f1331d5cce10" />

### 4.6 Source and Drain — Masks 9 and 10

A screen oxide helps limit implant channeling. Arsenic and boron implants create the heavily doped NMOS and PMOS source/drain regions. Annealing activates the dopants and helps repair implantation damage.

<img width="1920" height="1080" alt="9" src="https://github.com/user-attachments/assets/8a77f115-ab59-48da-83fa-d56d39aa7e07" />

### 4.7 Local Connections and Contacts — Mask 11

The oxide above selected gate and diffusion areas is opened. Titanium processing forms a conductive local-interconnect material, and Mask 11 defines the contact regions. These short connections link nearby transistor terminals before the higher-level routing is added.

<img width="1920" height="1080" alt="10" src="https://github.com/user-attachments/assets/eb32177d-1459-4e11-bdca-fab4be80289d" />
<img width="1920" height="1080" alt="11" src="https://github.com/user-attachments/assets/c03a4b2c-6d84-45da-b60b-48a004302286" />
<img width="1920" height="980" alt="12" src="https://github.com/user-attachments/assets/ed28e2dd-0149-4b5b-9ee9-b162ca614d81" />
<img width="1920" height="998" alt="13" src="https://github.com/user-attachments/assets/61563921-29b2-4cbc-a3b1-6702a64b66d3" />

### 4.8 Upper Metal Layers and Passivation — Masks 12–16

Dielectric deposition, planarization, contact filling, and metal patterning are repeated to build the routing stack. Tungsten-filled contacts connect layers, aluminum forms metal patterns, and a final silicon-nitride passivation layer protects the chip. Mask 16 opens the bond-pad areas so the completed device can connect to external circuitry.

<img width="1920" height="997" alt="Screenshot 2026-09-15 230644" src="https://github.com/user-attachments/assets/29717eb9-87d2-4ca3-aae4-ba9c671a99c5" />
<img width="1920" height="986" alt="14" src="https://github.com/user-attachments/assets/be20a4ec-fd24-43a5-9bd5-d7cbc88d5d5a" />
<img width="1920" height="997" alt="Screenshot 2026-09-15 230655" src="https://github.com/user-attachments/assets/e56836f3-a5d6-4f89-970c-66313407cc80" />

---

## Practical Labs

### Lab 1 — Comparing IO Pin Placement

The floorplan was generated using the default equidistant IO placement and then with another placement mode. Comparing the views makes it possible to see how the selected mode changes pin distribution around the die.

<img width="1920" height="983" alt="io placer equi distance" src="https://github.com/user-attachments/assets/4a7ae227-5ad5-4bae-8821-903fcf729087" />
<img width="1920" height="983" alt="zoomed io placer equi distance" src="https://github.com/user-attachments/assets/2830dee4-8a77-4da5-bdb0-2d0dc4596937" />
<img width="1920" height="983" alt="io placer after mode set to 2" src="https://github.com/user-attachments/assets/9695ad32-b6e5-4119-9c88-1dd5ca9215f8" />
<img width="1920" height="983" alt="zoomed io placer after mode set to 2" src="https://github.com/user-attachments/assets/405ed811-699e-4f9f-bb3a-9daa36eea458" />

### Lab 2 — Preparing Magic and the Sky130 Inverter

The `vsdstdcelldesign` repository was cloned to obtain the `sky130_inv.mag` reference layout. The Sky130A Magic technology file was copied into the working directory, and Magic was launched with:

```bash
magic -T sky130A.tech sky130_inv.mag &
```

The layout shows the inverter's supply rails, input `A`, output `Y`, and PMOS/NMOS device regions.

<img width="1920" height="983" alt="git clonned vsdstdcelldesign" src="https://github.com/user-attachments/assets/db652168-1e2e-464b-a044-94ae4585dcf8" />
<img width="1920" height="983" alt="copied sky130A tech to the directory clonned" src="https://github.com/user-attachments/assets/ca24c556-4172-4685-b192-f6d37c6c6c66" />
<img width="1920" height="983" alt="magic cmd" src="https://github.com/user-attachments/assets/673c71f0-e71d-4e68-a58f-b3e796b77f13" />
<img width="1920" height="983" alt="inverter layout" src="https://github.com/user-attachments/assets/ce662f30-6b04-4f39-a24c-7b8f46578e84" />

### Lab 3 — Checking the Transistor Layers

The Magic `what` command was used in the console to identify the layer beneath each selected device region. This confirmed the NMOS and PMOS areas in the inverter layout.

<img width="1920" height="983" alt="nmos" src="https://github.com/user-attachments/assets/b2d1f6ce-cf62-4498-9578-d6fdd7928c6d" />
<img width="1920" height="983" alt="pmos" src="https://github.com/user-attachments/assets/1da18003-1631-48df-83e6-467e6f751bcf" />

### Lab 4 — Exporting the Layout as SPICE

The following Magic commands extracted the layout information and generated a SPICE file:

```tcl
extract all
ext2spice cthresh 0 zthresh 0
ext2spice
```

The resulting `sky130_inv.ext` and `sky130_inv.spice` files contained geometry-based device information. The first extraction still used Magic's internal transistor model names and lacked a complete simulation setup.

<img width="1920" height="983" alt="extraxct all cmd in tcon" src="https://github.com/user-attachments/assets/a13e8bb1-95c8-4c88-b90f-752aae26b395" />
<img width="1920" height="983" alt="exttospicecmd" src="https://github.com/user-attachments/assets/9b0e9537-3224-4d31-afc3-7088e776dfc9" />
<img width="1920" height="983" alt="created successfully" src="https://github.com/user-attachments/assets/56897573-d67f-413a-b404-fc2f65a53d76" />
<img width="1920" height="983" alt="sky130_inv spice file" src="https://github.com/user-attachments/assets/e51a20f6-d6d2-4c9b-9573-727daf11e123" />

### Lab 5 — Finding the Device Model Files

The repository's `libs` directory was examined to locate the short-channel NMOS and PMOS models, along with the standard-cell corner libraries. The model names identified for use in the corrected deck were `nshort_model.0` and `pshort_model.0`.

<img width="1920" height="983" alt="lib files" src="https://github.com/user-attachments/assets/a8557a22-1863-4bec-8dee-74c971469eff" />
<img width="1920" height="983" alt="nshort_model file" src="https://github.com/user-attachments/assets/c51becb8-baca-4dc8-b443-e322123f5cda" />
<img width="1920" height="983" alt="pshort_model file" src="https://github.com/user-attachments/assets/005f68d0-989e-4661-a4f9-6c5d46bbd45f" />

### Lab 6 — Correcting the Deck and Starting ngspice

The initial simulation reported an unknown-subcircuit/model error because the extracted deck did not reference the model libraries correctly. The deck was then updated to include `./libs/pshort.lib` and `./libs/nshort.lib`, define the inverter subcircuit, instantiate the correct transistor models, provide a pulse input, and include transient-analysis commands. The revised simulation ran without the earlier error.

<img width="1920" height="983" alt="ngspice run cmd" src="https://github.com/user-attachments/assets/6c881ec6-429d-4d53-b621-a372972a3194" />
<img width="1920" height="983" alt="modified spice deck file" src="https://github.com/user-attachments/assets/71186fc0-2da7-4fc6-8b96-f5d5acefde21" />

### Lab 7 — Measuring Inverter Timing

The transient waveform was viewed in ngspice and the cursor was used to measure the relevant voltage crossings. The recorded results were:

| Measurement | Definition | Recorded value |
|---|---|---:|
| Rise transition time | 20%–80% on a rising output edge | 0.06194 ns |
| Fall transition time | 20%–80% on a falling output edge | 0.04267 ns |
| Cell rise delay | 50% input to 50% output | 0.05953 ns |
| Cell fall delay | 50% input to 50% output | 0.05941 ns |

<img width="1920" height="983" alt="transcient analysis plot of cmps inverter (modified code)" src="https://github.com/user-attachments/assets/587558a9-e4dc-4152-a208-e33f39c9a55b" />
<img width="1917" height="987" alt="rise time transistion" src="https://github.com/user-attachments/assets/37a15742-305a-4bd8-89e2-1a381692c4b1" />
<img width="1917" height="981" alt="fall time transistion" src="https://github.com/user-attachments/assets/97e1ec5d-88f7-47c5-82e9-6ed0a2652d99" />
<img width="1912" height="982" alt="cell rise delay" src="https://github.com/user-attachments/assets/12064c53-e62a-4c83-96f4-507ee86bde61" />
<img width="1917" height="977" alt="cell fall delay" src="https://github.com/user-attachments/assets/6c18e544-6a78-42d6-820b-047769db1b27" />

### Lab 8 — Reviewing Metal3 and Poly DRC Rules

The example DRC deck included multiple Metal3 structures, both compliant and non-compliant. Magic's `drc why` command identified issues including:

- `Metal3 spacing < 0.3um (met3.2)`
- `Metal3 minimum area < 0.24um^2 (met3.6)`

The VIA2 region was inspected using `paint m3contact` and `cif see VIA2`. A selected feature measured 0.190 × 0.200 µm, giving an area of 0.038 µm². A selected polysilicon shape measured 0.335 × 0.210 µm, or approximately 0.070 µm², and was associated with rule `poly.9`.

<img width="1920" height="983" alt="met3" src="https://github.com/user-attachments/assets/f83f8c06-707d-4c40-b04d-28f64313001d" />
<img width="1920" height="983" alt="drc why met2" src="https://github.com/user-attachments/assets/22ed60bf-4c47-4821-bb7d-90b772be695d" />
<img width="1920" height="983" alt="drc why met 3 2" src="https://github.com/user-attachments/assets/19cee2bb-670f-4174-9ddd-6691324c6712" />
<img width="1920" height="983" alt="cif see VIA2 cmd full view" src="https://github.com/user-attachments/assets/3094af63-0b52-43de-b6fc-360d3d36878d" />
<img width="1920" height="983" alt="cif see VIA2 cmd" src="https://github.com/user-attachments/assets/2a803c93-5686-4559-b2bb-1710e3dc4f14" />
<img width="1920" height="983" alt="measurement" src="https://github.com/user-attachments/assets/2afd3f72-2eeb-40d3-9e84-3d3d3682c10a" />
<img width="1920" height="983" alt="rule violation" src="https://github.com/user-attachments/assets/d80dfd77-e13a-40bb-be33-398927d40db2" />

---

## Conclusion

This session tied together the electrical behavior of a CMOS inverter and the physical process used to manufacture it. The fabrication review showed how wells, gates, source/drain regions, contacts, metal routing, and passivation are built in sequence. The practical work then followed a layout through extraction and model correction to transient simulation, producing measurable rise/fall delays and transition times. Finally, the DRC exercises connected layout geometry to concrete spacing and minimum-area rules.

Together, these exercises show how device structure, SPICE characterization, timing behavior, and layout-rule compliance fit into the wider standard-cell design flow.
