
# Week 4: Local Pipeline Deployment & Cross-Platform Metrics Auditing

## 📌 Overview

Week 4 focuses on evaluating our custom, natively compiled **OpenROAD** binary by executing the end-to-end RTL-to-GDSII hardening pipeline locally. Using the automated **OpenROAD Flow Scripts (ORFS)** orchestration engine and the **Sky130HD PDK**, the Greatest Common Divisor (GCD) core was pushed from behavioral logic to a structural physical layout. The final signoff data was audited and compared directly against previous cloud-based container execution logs to measure cross-platform PPA performance shifts.

---

## 🚀 Native Pipeline Execution & Step-by-Step Footprint Sizing

The local automated ASIC flow is launched via the top-level dependency coordinator:

```bash
make sky130hd_gcd

```

The automation framework tracks intermediate database outputs across each layout milestone, showing how physical structures steadily occupy the core's real estate:

### Stage 1: RTL Logic Synthesis

`Yosys` processes the structural hardware files, mapping behavioral expressions onto the discrete standard cells of the Sky130HD library while exporting the initial constraint profiles.

### Stage 2: Floorplan Boundary Initialization

The tool locks down the initial die perimeter boundaries to establish the baseline core geometry.

* **Core Area Allocation:** $2,750\,\mu\text{m}^2$
* **Initial Target Utilization:** 50% (Whitespace reserved for subsequent optimizations)

### Stage 3: Standard Cell Placement Optimization

The global and detailed placement engines legalize standard cells into designated rows while minimizing interconnect wirelength.

* **Core Area Growth:** $4,111\,\mu\text{m}^2$
* **Placement Utilization:** 74%

### Stage 4: Clock Tree Synthesis (CTS)

`TritonCTS` replaces ideal clock lines by constructing a physical distribution tree, inserting high-drive buffers to balance arrival lines.

* **Core Area Growth:** $4,927\,\mu\text{m}^2$
* **CTS Utilization:** 89% (Reflects hardware area overhead from clock buffer cells)

### Stage 5: Interconnect Route Locking

`TritonRoute` performs design-rule-aware wire routing, converting abstract global layout guides into actual metal traces with clean results:

```text
Antenna Violations = 0 | Net Violations = 0 | Pin Violations = 0

```

---

## 📊 Final Layout Composition & Cell Distribution Audit

Following routing closure, the physical database was checked to inventory the cell distribution across the core area:

```text
Final Layout Core Area   = 5,281 µm²
Final Core Utilization   = 95.3%

```

### Physical Cell Inventory

| Placed Cell Category | Structural Instance Count | Architectural Purpose |
| --- | --- | --- |
| **Combinational Cells** | 226 cells | Implements basic boolean logic, multiplexers, and arithmetic. |
| **Timing Repair Buffers** | 196 cells | Inserted along paths to fix hold violations and setup slack. |
| **Fill Cells** | 131 cells | Ensures layout continuity and uniform planarity across layers. |
| **Tap Cells** | 72 cells | Biases the substrate to prevent latch-up hazards. |
| **Sequential Cells** | 35 cells | State-holding registers and flip-flops driving data. |
| **Clock Tree Buffers** | 6 cells | Provides high-drive current balancing across the H-tree branches. |

---

## ⏱️ Post-Routing Timing Signoff Summary

Static Timing Analysis was executed using **OpenSTA** on the fully routed local netlist, pulling real wire parasitics to calculate operational limits:

```ini
Worst Negative Slack (WNS)  = -1.77 ns
Total Negative Slack (TNS)  = -85.06 ns
Worst Slack                 = -1.77 ns
Minimum Clock Period        = 2.87 ns
Maximum Operating Frequency = 348.80 MHz
Clock Setup Skew            = 0.01 ns

```

The small clock setup skew ($0.01\text{ ns}$) demonstrates that the clock tree synthesis engine balanced the propagation paths effectively across the local register sinks.

---

## 📊 Performance Analysis: Cloud vs. Native Host Environment

To measure the impact of software execution environments on physical design optimization, the local native metrics were audited against previous cloud container execution data:

### Cross-Platform PPA Metrics Matrix

| Hardware Signoff Metric | Cloud Platform (Container) | Local Host (Native Source Build) | Net Performance Delta |
| --- | --- | --- | --- |
| **Total Pipeline Runtime** | 5 min 39 sec | 4 min 14 sec | **-1 min 25 sec (~25% Faster)** |
| **Worst Negative Slack (WNS)** | -2.19 ns | -1.77 ns | +0.42 ns Setup Slack Restored |
| **Total Negative Slack (TNS)** | -92.21 ns | -85.06 ns | +7.15 ns Global Timing Recovery |
| **Max Clock Frequency ($F_{max}$)** | 303.91 MHz | 348.80 MHz | **+44.89 MHz Performance Boost** |
| **GDSII Layout Validation** | Generated Successfully | Generated Successfully | Verified Clean |

```markdown
![Local GDSII Layout Verification](Images/Week_4_1.png)
*Figure 1: Successful layout inspection of the natively generated 6_final.gds database file in KLayout.*

```

> 📊 **Core Mechanical Insight:** The performance differences stem directly from the underlying OpenROAD source versions, differing host hardware thread management, and unique optimization seed paths chosen by the layout engines. The native build completed the layout roughly 25% faster and achieved a higher maximum operating frequency ($F_{max} = 348.80\text{ MHz}$).

---

## 💡 Key Technical Takeaways

* **Native Speedups Match Compilation Overhead:** The native compilation completed the optimization pipeline 25% faster than containerized alternatives, showing that custom builds utilize host computing threads and memory channels more efficiently.
* **Environmental Variables Shape Downstream PPA:** Minor variations in physical timing slacks and operating frequencies across platforms prove that backend layout algorithms are sensitive to minor version shifts in the core binaries.
* **Cell Overhead is Dominated by Timing Fixes:** The inventory reveals that timing repair buffers (196 cells) heavily outnumber active sequential blocks (35 cells). This highlights that a massive portion of silicon real estate is dedicated entirely to closing setup and hold margins during physical design.

---

## Tooling Matrix

* **ASIC Implementation Pipeline:** OpenROAD Flow Scripts Platform (ORFS)
* **Design Orchestration Layer:** GNU Make Engine Sub-System
* **Logic Synthesis Engine:** Yosys Open SYnthesis Suite
* **Physical Layout Optimizer:** OpenROAD (incorporating TritonCTS and FastRoute)
* **Signoff Timing Engine:** OpenSTA (Propagated Parasitic Mode)
* **Graphical Layout Viewer:** KLayout Graphics Suite
* **Process Design Kit Library:** Google/SkyWater SKY130HD (High Density)
* **Host Operating System:** Linux Ubuntu 24.04 LTS (Native Environment)
