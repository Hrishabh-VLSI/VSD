# Week 4 (SoC): RTL-to-GDSII Hardening of Caravel User Project Wrapper

## 📌 Overview

This phase covers the complete RTL-to-GDSII physical design implementation of the Caravel **User Project Wrapper** (`user_project_wrapper`). Utilizing **OpenROAD Flow Scripts (ORFS)** mapped onto the **Sky130HD PDK**, this hardening cycle integrates structural SoC blocks with the system interconnect harness, enforces a baseline 100 MHz constraint boundary, and resolves real-world floorplan initialization and tool path conflicts to achieve clean physical signoff.

---

## 🏗️ Phase 1: Top-Level Wrapper Architecture & RTL Hierarchy

The `user_project_wrapper` acts as the primary structural interface linking custom user logic to the core **Efabless Caravel SoC** subsystem. Rather than performing active data processing, it manages address space decoding and distributes shared system lines—including the Wishbone interconnect bus, Logic Analyzer (LA) probes, and peripheral GPIO routing channels.

### Structural Module Dependency Tree

```text
user_project_wrapper (Top Integration Layer)
│
├── debug_regs (Wishbone-accessible Debug Configuration Registers)
│
├── user_project_gpio_example (Conditional compilation via `GPIO_TESTING`)
│
└── user_project_la_example   (Conditional compilation via `LA_TESTING`)

```

### RTL Deliverables Matrix

| System Source File | Design Description & Functional Scope | Role |
| --- | --- | --- |
| `user_project_wrapper.v` | Structural integration module anchoring the user design to the Caravel infrastructure. | **Top Module** |
| `debug_regs.v` | Implements the interior Wishbone register block tracking hardware debug states. | Sub-Module |
| `defines.v` | Global macro repository defining compile-time configurations and bus metrics. | Package Lib |
| `user_project_gpio_example.v` | Optional testing peripheral instantiated for dynamic I/O buffer validation. | Optional Ref |
| `user_project_la_example.v` | Optional verification block mapping interior signals to Logic Analyzer probes. | Optional Ref |

> 🔍 **Address Decoding Mechanics:** The top-level wrapper acts as an address router. When a Wishbone transaction enters the core, the wrapper decodes the incoming address lines. If the target falls inside the reserved system debug sector, the transaction is routed directly to `debug_regs`. All other transaction ranges are forwarded cleanly to the user project space.

---

## 🛠️ Phase 2 & 3: ORFS Design Setup & 100 MHz Constraint Engineering

To provision the automated layout environment, an isolated workspace configuration was developed:

```text
user_project_wrapper_workspace/
├── config.mk         # Global parameters, RTL lists, and PDK sizing choices
├── constraints.sdc   # Synopsys Design Constraints targeting clock period boundaries
└── rtl/              # Source code repository hosting wrapper modules

```

### Synopsys Design Constraints (`constraints.sdc`)

To ensure timing closure across the Wishbone bus logic, a strict clock constraint was defined to govern Static Timing Analysis:

```tcl
create_clock -name clk -period 10.0 [get_ports wb_clk_i]

```

The system clock input (`wb_clk_i`) drives all synchronous register transfers across the design fabric. Enforcing a **10.0 ns period** sets a target operating speed of **100 MHz**, giving the timing optimization engine a clear target for setup and hold path analysis.

---

## 🚀 Phase 4: RTL-to-GDSII Physical Hardening Execution

The automated layout pipeline was launched via the top-level Make coordinator:

```bash
make DESIGN_CONFIG=./designs/sky130hd/user_project_wrapper/config.mk

```

### 📋 Resolution Log & Flow Milestones

#### 1. Binary Discovery Error

* **Roadblock:** The flow aborted immediately because ORFS could not find the active paths for Yosys and OpenROAD.
* **Resolution:** Reconfigured the shell variables by explicitly exporting the absolute system binary paths inside the Make command wrapper.

#### 2. Floorplan Initialization Collision

* **Roadblock:** The floorplan step threw a fatal initialization error: `Floorplan initialization methods are mutually exclusive`. This occurred because the local `config.mk` specified custom `DIEAREA` coordinates while simultaneously activating a global `CORE_UTILIZATION` percentage.
* **Resolution:** Removed the `CORE_UTILIZATION` line entirely, locking down the manual die/core coordinate boxes to guarantee a deterministic silicon perimeter.

```markdown
![Floorplan Initialization Collision](Images/Week_4_Floorplan_Error.png)
*Figure 1: Automated terminal log displaying the mutual exclusion floorplan initialization abort.*

```

```markdown
![Successful Floorplan Locking](Images/Week_4_Successful_Floorplan.png)
*Figure 2: Resolution log demonstrating successful die perimeter bounding after parameter pruning.*

```

#### 3. Standard Cell Placement & CTS Optimization

Following floorplan definition, standard cells were aligned to row sites. The design was then handed off to the placement optimizer and `TritonCTS` to build the physical clock distribution tree, replacing ideal clock lines with real clock buffers to satisfy setup and hold constraints.

```markdown
![Cell Placement & CTS Optimization](Images/Week_4_Placement_CTS.png)
*Figure 3: Layout pipeline log executing global cell legalization and physical clock buffer tree insertion.*

```

#### 4. Route Routing & Signoff Verification

Signal connections were completed via a two-pass global and detailed routing engine. After routing, filler cells were added to fill any empty spaces between standard cell instances, ensuring continuous power tracks across the core.

```markdown
![Interconnect Routing & Filler Insertion](Images/Week_4_Routing_Fill.png)
*Figure 4: Detailed routing validation logs showing zero antenna violations followed by layout cell filling.*

```

---

## ⏱️ Final Signoff Timing & Layout Profile Summary

Once the physical design pipeline completed, the post-routing database was parsed to extract the signed-off performance metrics:

### Physical Implementation Metrics

```ini
Worst Negative Slack (WNS)  = 0.00 ns (MET)
Total Negative Slack (TNS)  = 0.00 ns (MET)
Worst Slack                 = 0.00 ns (Zero Timing Violations)

```

Achieving exactly **0.00 ns of negative slack** confirms that every synchronous data path across the wrapper fabric closes timing successfully within the targeted 100 MHz (10 ns) operating boundary.

```markdown
![Final PPA Reporting](Images/Week_4_Final_Report.png)
*Figure 5: Physical signoff dashboard displaying zero timing violations and completed runtime execution summary.*

```

---

## 📂 Phase 5: ORFS File Hierarchy & Layout Forensic Analysis

The layout run outputs three structured top-level directories, providing a transparent trace of the entire ASIC pipeline:

```text
user_project_wrapper_run/
├── logs/              # Stage-by-stage tool logs (.log) and runtime metadata (.json)
├── reports/           # Static PPA text files (.rpt) and graphical layout views (.webp)
└── results/           # Physical design databases (.odb, .def, .spef, and final .gds)

```

```markdown
![ORFS Flow Deliverables Hierarchy](Images/Week_4_Logs_Results_Reports.png)
*Figure 6: Structural view of the generated logs, results, and reports workspace directories.*

```

### Critical Flow Deliverables Index

| Generated File Name | Extension | Architectural Contents & Purpose |
| --- | --- | --- |
| `1_synth` | `.odb` / `.v` | Technology-mapped gate-level netlist and initial database layout. |
| `2_floorplan` | `.odb` / `.def` | Initial die perimeter definitions, pin tracks, and locked power grids. |
| `3_place` | `.odb` | Legalized cell positions mapping standard logic gates onto row sites. |
| `4_cts` | `.odb` / `.sdc` | Propagated database containing the physical clock buffer tree structure. |
| `5_route` | `.odb` | Fully routed interconnect net layout tracking multi-layer metal wires. |
| `6_final` | `.gds` | **Final GDSII Layout Blueprint** ready for manufacturing. |
| `6_final` | `.spef` | Extracted parasitic RC wire network models used for signoff STA. |

```markdown
![Final GDSII Layout Verification](Images/Week_4_Final_GDS.png)
*Figure 7: Full graphical visualization of the completed 6_final.gds layout verified inside KLayout.*

```

---

## 💡 Key Technical Takeaways

* **Configuration Priorities Control Layout Initializations:** Physical design automation requires precise configuration parameters. Mixing automated scaling (utilization targets) with rigid physical definitions (die coordinates) causes tool initialization conflicts, requiring deterministic config overrides.
* **Top-Level Wrappers Focus on Routing, Not Logic:** The main responsibilities of structural wrappers are address management, signal buffering, and bus isolation. Keeping control logic isolated from the user space protects global system operations.
* **The GDSII File Closes the RTL Loop:** Opening the final `.gds` file inside KLayout bridges the gap between software hardware descriptions and physical silicon execution, proving that the digital Verilog code has been successfully converted into a manufacturable chip blueprint.

---

## Tooling Matrix

* **ASIC Flow Automation Platform:** OpenROAD Flow Scripts Framework (ORFS)
* **Logic Synthesis Engine:** Yosys Open SYnthesis Suite
* **Physical Layout Optimizer:** OpenROAD (incorporating TritonCTS and TritonRoute)
* **Signoff STA Engine:** OpenSTA (Propagated Parasitic Mode)
* **Graphical GDSII Viewer:** KLayout Layout Verification Suite
* **Process Design Node Kit:** Google/SkyWater SKY130HD (High Density)
* **Orchestration Tooling:** GNU Make / Shell Environment Variables
* **Host Operating System:** Linux Cloud Infrastructure Sandbox (Ubuntu 22.04 LTS)
