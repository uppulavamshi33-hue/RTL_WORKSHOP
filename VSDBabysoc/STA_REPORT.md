VSDBabySoC — Multi-Corner Static Timing Analysis for 16 PVT Conditions
1. Project Overview
This report presents a Static Timing Analysis (STA) study of VSDBabySoC performed with OpenSTA/OpenROAD and the SKY130 standard-cell library.
The main goal is to assess the timing performance of the synthesized and physically implemented circuit under a range of process, voltage, and temperature (PVT) combinations.
The study covers setup and hold checks, Worst Negative Slack (WNS), Total Negative Slack (TNS), and timing verification at 16 distinct PVT corners.
Key project details:
- Design: VSDBabySoC (RISC-V-based System-on-Chip)
- Technology: SKY130
- Standard-cell library: SKY130 HD
- Timing tools: OpenSTA and OpenROAD
- Analysis: Static Timing Analysis
- Corners analyzed: 16 PVT corners
- Timing checks: Setup and hold
- Implementation stage: Post-route STA
2. VSDBabySoC Design Background
VSDBabySoC is a compact mixed-signal System-on-Chip combining a RISC-V processor with analog intellectual-property blocks.
The main design components include:
- RVMYTH: RISC-V-based processor core.
- PLL: Phase-Locked Loop used to generate the system clock.
- DAC: Digital-to-Analog Converter for analog output.
STA checks whether signals travel through the digital logic within the specified timing requirements for the selected operating conditions.
3. Goals of Static Timing Analysis
Static Timing Analysis evaluates circuit timing mathematically, without simulating every possible combination of input values.
The analysis is intended to:
1. Verify setup timing across supported PVT corners.
2. Verify hold timing across supported PVT corners.
3. Identify critical timing paths.
4. Calculate WNS and TNS.
5. Evaluate the effect of process, voltage, and temperature variations.
6. Identify timing violations requiring optimization.
7. Compare timing behavior between operating corners.
4. Design Tools and Technology Stack
Component	Description
Design	VSDBabySoC
PDK	SKY130
Standard-cell library	sky130_fd_sc_hd
Synthesis	Yosys
Technology mapping	ABC
Physical design	OpenLANE / OpenROAD
Static timing analysis	OpenSTA
Timing constraints	SDC
Physical parasitics	Routed interconnect estimates or extracted parasitics, depending on the analysis stage


Standard-cell Liberty models describe cell timing properties such as propagation delay, setup and hold requirements, input capacitance, and output transition behavior.
5. STA Workflow
The analysis follows this implementation sequence:
VSDBabySoC RTL
      |
      v
   Synthesis
      |
      v
Gate-Level Netlist
      |
      v
  Floorplanning
      |
      v
   Placement
      |
      v
Clock Tree Synthesis
      |
      v
     Routing
      |
      v
Post-Route Timing Analysis
      |
      v
Analyze 16 PVT Corners
      |
      v
Setup and Hold Verification
      |
      v
WNS / TNS Evaluation
      |
      v
Identify Critical Paths
      |
      v
Timing Optimization

Post-route STA uses the implemented physical design together with available interconnect parasitics. Result accuracy depends on whether those parasitics are extracted or estimated.
6. Understanding PVT Corners
PVT refers to Process, Voltage, and Temperature.
Variations in these three parameters can change standard-cell delay and overall circuit timing.
6.1 Process
- FF (Fast-Fast): Fast process corner.
- SS (Slow-Slow): Slow process corner.
- TT (Typical-Typical): Typical process corner.
6.2 Voltage
The voltage suffix identifies the operating voltage in the corner name.
For example:
- 1v28 represents 1.28 V.
- 1v65 represents 1.65 V.
- 1v80 represents 1.80 V.
- 1v95 represents 1.95 V.
6.3 Temperature
The temperature suffix identifies the operating temperature.
For example:
- n40C represents −40 °C.
- 025C represents 25 °C.
- 100C represents 100 °C.
Each corner label specifies the process model, temperature, and supply voltage applied in the timing run.
7. The 16 PVT Corners
The following table lists the 16 operating corners included in the study.
No.	PVT Corner	Process	Temperature	Voltage
1	FF_100C_1v65	FF	100 °C	1.65 V
2	FF_100C_1v95	FF	100 °C	1.95 V
3	FF_n40C_1v56	FF	−40 °C	1.56 V
4	FF_n40C_1v65	FF	−40 °C	1.65 V
5	FF_n40C_1v76	FF	−40 °C	1.76 V
6	FF_n40C_1v95	FF	−40 °C	1.95 V
7	SS_100C_1v40	SS	100 °C	1.40 V
8	SS_100C_1v60	SS	100 °C	1.60 V
9	SS_n40C_1v28	SS	−40 °C	1.28 V
10	SS_n40C_1v35	SS	−40 °C	1.35 V
11	SS_n40C_1v40	SS	−40 °C	1.40 V
12	SS_n40C_1v44	SS	−40 °C	1.44 V
13	SS_n40C_1v60	SS	−40 °C	1.60 V
14	SS_n40C_1v76	SS	−40 °C	1.76 V
15	TT_025C_1v80	TT	25 °C	1.80 V
16	TT_100C_1v80	TT	100 °C	1.80 V


8. Key Timing Metrics and Equations
8.1 Setup Slack
A setup check confirms that data reaches the capture register sufficiently before its active clock edge.
Setup Slack = Required Arrival Time - Data Arrival Time

For a simplified single-cycle path:
Setup Slack =
Clock Period
- Setup Time
- Clock Uncertainty
- Data Path Delay

The actual STA calculation also accounts for launch and capture clock arrival times, clock skew, and other applicable timing constraints.
Interpretation:
- Positive setup slack: Setup requirement is satisfied.
- Zero setup slack: The path meets the timing limit exactly.
- Negative setup slack: Setup timing violation.
8.2 Hold Slack
A hold check verifies that data remains stable long enough after the capture clock edge.
Hold Slack = Data Arrival Time - Required Arrival Time

The exact calculation depends on minimum data-path delay, clock arrival times, hold requirements, and timing uncertainty.
Interpretation:
- Positive hold slack: Hold requirement is satisfied.
- Zero hold slack: The path meets the hold limit exactly.
- Negative hold slack: Hold timing violation.
8.3 Worst Negative Slack (WNS)
WNS reports the minimum slack found among the paths included in the analysis.
WNS = Minimum Slack

A negative setup WNS indicates that at least one analyzed setup path violates its timing requirement.
8.4 Total Negative Slack (TNS)
TNS adds together the negative slack values from all paths that violate the timing requirement.
TNS = Sum of Negative Slack Values

A negative TNS indicates accumulated timing violations. A larger negative magnitude generally indicates a greater aggregate timing problem.
9. Results from Post-Route STA
The post-route reports show that timing performance varies depending on the selected PVT corner.
The reported results indicate:
- All FF corners satisfy setup timing.
- Both TT corners satisfy setup timing.
- SS_100C_1v60 and SS_n40C_1v76 satisfy setup timing.
- Six SS corners show setup timing violations.
- Hold timing is satisfied at all 16 analyzed corners.
9.1 Setup Timing Results for SS Corners
The table below reproduces values from the available post-route STA summary.
PVT Corner	Setup WNS (ns)	Setup TNS (ns)	Status
SS_n40C_1v28	-29.32	-14575.39	FAIL
SS_n40C_1v35	-16.45	-6570.44	FAIL
SS_n40C_1v40	-10.99	-3336.45	FAIL
SS_n40C_1v44	-7.72	-1909.84	FAIL
SS_100C_1v40	-3.51	-326.35	FAIL
SS_n40C_1v60	-0.30	-1.19	FAIL
SS_100C_1v60	Not listed here	Not listed here	PASS
SS_n40C_1v76	Not listed here	Not listed here	PASS


For the passing corners, the precise WNS and TNS figures are not included here; they should be taken directly from the matching rows in the original reports.
9.2 Worst-Case Setup Corner
The worst setup timing result occurs at:
Corner: SS_n40C_1v28

Setup WNS = -29.32 ns
Setup TNS = -14575.39 ns

Among the reported corners, this condition has the lowest setup WNS, making it the most critical setup case in the available data.
The strongly negative TNS suggests that setup violations affect numerous paths rather than only one isolated path.
9.3 Setup Timing Observations
The data indicates that several slow-process (SS) conditions present greater setup-timing challenges.
The worst reported violation occurs at SS_n40C_1v28, while SS_n40C_1v60 has a smaller negative setup WNS of −0.30 ns.
This spread in results highlights why a design should be checked across multiple operating corners rather than judged from a typical corner alone.
10. Hold-Check Findings
According to the available post-route STA summary, all 16 corners meet the hold-timing checks.
Hold timing violations = 0
Hold WNS               = 0.00 ns
Hold TNS               = 0.00 ns

The report also shows positive hold worst slack at every corner.
The reported hold WNS and TNS are both 0.00 ns, indicating that the analysis contains no negative hold-slack violations.
Note: A zero WNS in this summary does not necessarily mean that a path has exactly zero slack. STA reports commonly use zero as the WNS floor when there are no negative-slack paths.
11. Consolidated Timing Summary
Parameter	Result
Total PVT corners	16
FF setup status	PASS
TT setup status	PASS
SS setup status	Six corners fail; two pass
Hold timing	PASS at all analyzed corners
Hold violations	0
Worst setup corner	SS_n40C_1v28
Worst setup WNS	-29.32 ns
Worst setup TNS	-14575.39 ns
Overall setup timing	Not closed across all corners
Overall hold timing	Satisfied in the reported analysis


12. Interpretation and Next Steps
The main concern highlighted by this STA run is setup timing.
Although the FF and TT corners pass setup timing, six SS corners show negative setup slack.
The most critical result is at SS_n40C_1v28, where the setup WNS is −29.32 ns and the setup TNS is −14575.39 ns.
Setup optimization and further verification are needed before timing closure can be claimed for every corner.
Possible optimization steps include:
1. Inspect the worst setup paths using detailed timing reports.
2. Identify the logic levels and cells contributing most to the critical path delay.
3. Review the clock period and all timing constraints for correctness.
4. Check whether the intended clock source and clock period are correctly defined.
5. Review high-fanout nets and interconnect delays.
6. Evaluate cell sizing and buffering where appropriate.
7. Re-run synthesis or physical implementation after making changes.
8. Re-run setup and hold analysis across all 16 corners.
9. Verify that any setup improvement does not introduce hold violations.
Given the size of the worst violation, first validate the clock definition and timing constraints before pursuing aggressive cell-level changes.
13. Running Timing Checks with OpenSTA/OpenROAD
A typical post-route analysis requires the correct physical database, timing libraries, design netlist, timing constraints, and parasitic information.
A simplified OpenROAD Tcl sequence is shown below. File paths and library selection must match the actual run.
# Load the physical database
read_db <design_database.odb>

# Load the timing libraries for the selected corner
read_liberty <corner_liberty_file.lib>

# Load the design netlist if required by the database workflow
read_verilog <post_route_netlist.v>

# Link the design
link_design vsdbabysoc

# Load timing constraints
read_sdc <design_constraints.sdc>

# Load extracted parasitics when available
read_spef <extracted_parasitics.spef>

# Enable propagated clock analysis
set_propagated_clock [all_clocks]

# Report setup and hold paths
report_checks -path_delay min_max \
    -format full_clock_expanded -digits 4

# Report clock skew
report_clock_skew -digits 4
Treat this snippet as a starting template, not a ready-to-run script. Ensure the database, netlist, parasitics, and timing library all correspond to the same design and corner. Depending on the flow, some of these inputs may already be included or loaded by other commands.
14. Reading an STA Report
The following report fields are useful when reviewing STA output:
Startpoint: The launch point of the timing path.
Endpoint: The capture point or endpoint of the timing path.
Path Group: The clock or timing group associated with the path.
Path Type: Indicates whether the report is evaluating maximum-delay or minimum-delay timing.
Data Arrival Time: The time at which data reaches the endpoint.
Data Required Time: The latest or earliest permitted arrival time, depending on the check.
Slack: The difference between required time and arrival time, with the sign convention depending on the timing check.
For setup checks, begin with the paths having the most negative slack. For hold checks, investigate any paths that report negative hold slack.
15. Conclusion and Final Assessment
This post-route STA study evaluates the VSDBabySoC implementation at 16 PVT corners with the SKY130 standard-cell library.
The available results show successful hold checks at every analyzed corner, but setup violations remain at six SS corners.
The worst reported setup result is at SS_n40C_1v28, with a WNS of −29.32 ns and a TNS of −14575.39 ns.
These findings underline the value of multi-corner STA in physical-design verification. Setup success at FF and TT conditions alone does not prove that timing closes across the full operating range.
Recommended follow-up work is to inspect the most critical paths, confirm the clock and constraint setup, improve the relevant data paths, and repeat STA across all corners.
Final status: Hold timing is satisfied in the reported analysis, but setup timing requires further investigation and optimization before the design can be considered setup-clean across all 16 PVT corners.
16. Reference Links
- VSDBabySoC project directory
- OpenROAD Project
- OpenSTA
- SKY130 PDK
