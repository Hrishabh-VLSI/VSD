# Week 2: Automated RTL-to-GDS Flow via OpenROAD Flow Scripts (ORFS)

## Overview

Week 2 transitions the physical design workflow from interactive tool scripting to fully automated, industrial-style ASIC implementation pipelines. Utilizing **OpenROAD Flow Scripts (ORFS)** mapped to the **Sky130HD PDK**, this curriculum explores the end-to-end hardening of a Greatest Common Divisor (GCD) core. The progression spans initial cloud-based toolchain provisioning within GitHub Codespaces, low-level native compilation of the OpenROAD source suite on Ubuntu 24.04, and deep-dive post-routing report forensics.

---

## Toolchain Architecture & Cloud Development Environment

Modern Electronic Design Automation (EDA) flows depend heavily on stable tool environments. Instead of dealing with host operating system dependency clashes, ORFS abstracts the physical implementation pipeline through a containerized Dev Container development system.

```text
Host Environment (Local OS / Cloud Workspace)
       └── VS Code Workspace (.devcontainer.json / settings.json)
                 └── Docker Container Sandbox (Ubuntu 22.04 LTS Base)
                           ├── Core Engines: OpenROAD, Yosys, OpenSTA
                           └── Safety Utilities: Magic, Netgen, KLayout, GTKWave

```

Analyzing the container environment shows that ORFS wraps multiple specialized open-source engines into a unified execution framework orchestrated by GNU Make:

* **Logic Synthesis:** Yosys Open SYnthesis Suite.
* **Physical Design Engine:** OpenROAD (incorporating TritonCTS, FastRoute, and TritonRoute).
* **Static Timing Analysis (STA):** OpenSTA signoff engine.
* **Layout Verification:** Magic (DRC), Netgen (LVS), and KLayout (GDSII structural viewing).

---

## Cloud-Based RTL-to-GDS Automated Flow (Codespaces)

Hardening the GCD core within the container is managed by a top-level dependency-driven system execution file:

```bash
make sky130hd_gcd

```

The automation tool orchestrates the pipeline sequentially, passing intermediate database artifacts across the design phases:

### 1. Logic Synthesis & Technology Mapping

Yosys translates behavioral Verilog code into a structural netlist mapped to the high-density standard cells of the `sky130_fd_sc_hd` library, establishing the foundational logical gates.

### 2. Floorplan & Power Grid Initialization

The floorplan establishes the core boundaries with an initial placement utilization target of approximately 43%. This leaves ample whitespace to prevent wire congestion down the line. The power delivery network (`gen_pdn`) automatically overlays horizontal and vertical power straps to secure stable power delivery throughout the core.

### 3. Physical Placement & Clock Tree Synthesis (CTS)

Standard cells are globally distributed and legally aligned to row tracks. The design then moves to `TritonCTS`, which inserts a balanced tree of clock buffers. This step handles extreme clock fanout and ensures synchronized clock arrival across the register arrays, naturally increasing cell area overhead.

### 4. Interconnect Routing & Signoff Reports

`FastRoute` generates coarse routing guides, which `TritonRoute` converts into physical metal traces (Li1 through Met5) and interlayer vias. The post-routing verification pass confirmed clean routing execution:

```text
Antenna Violations = 0
Net Violations     = 0
Pin Violations     = 0

```

### 5. Final Timing Analysis Profile

Post-routing timing calculations pull the actual physical characteristics of the network into the signoff reports:

```ini
Worst Negative Slack (WNS)  = -2.19 ns
Total Negative Slack (TNS)  = -92.21 ns
Worst Slack                 = -2.19 ns
Minimum Clock Period        = 3.29 ns
Maximum Operating Frequency = 303.91 MHz

```

---

## Native OpenROAD Build & Compilation on Ubuntu 24.04

To understand the tool architecture beyond pre-built container setups, a native source compilation of the OpenROAD environment was performed on an Ubuntu 24.04 host system.

### Dependency Management & Interface Generation Conflict

Initial native installation scripts failed due to system package mismatches and runtime conflicts with the Simple Wrapper and Interface Generator (SWIG):

```text
Could NOT find SWIG:
Found unsuitable version "4.2.0"
Required version is at least "4.3"

```

The host OS environment was pointing to an older SWIG binary path, causing compilation to fail. Resolving this required building SWIG 4.3.0 from source and manually updating the system's search path variables:

```bash
export PATH=$HOME/swig/bin:$PATH
swig -version # Verifies active version reads 4.3.0

```

### Compiling from Source

Once the required developer packages (CMake, Boost, Bison, Flex, Eigen, and Google OR-Tools) were linked, the native compilation was executed:

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)

```

The system completed the native build successfully: `[100%] Built target openroad`. Compilation required an explicit execution runtime: **3 hours 23 minutes (202m 57s)**.

Verifying the generated binary confirmed a fully functional custom build:

```text
OpenROAD 26Q2-1895-gba65a4e291 (+GPU +GUI +Python)

```

---

## Local ORFS Flow Execution vs. Cloud Environment

The Sky130HD GCD design was re-run using the locally compiled OpenROAD binary. This verified that the native environment could successfully perform the entire design flow.

### Post-Routing Core Density Summary

The local run compiled cleanly, producing the following physical allocation metrics at signoff:

| Physical Parameter Metric | Local Report Value | Asset Allocation |
| --- | --- | --- |
| **Total Layout Die Area** | $5,281\,\mu\text{m}^2$ | • Fill Cells: 131 instances |
| **Final Layout Utilization** | 95.3% | • Tap Cells: 72 instances |
| **Combinational Logic Footprint** | 226 cells | • Clock Tree Buffers: 6 cells |
| **Sequential Logic Footprint** | 35 cells | • Timing Repair Buffers: 196 cells |

### Performance Metrics: Cloud vs. Local Environment

| Signoff Metric Group | Cloud Environment (Container) | Local Environment (Native Source Build) | Performance Variance |
| --- | --- | --- | --- |
| **Total Compilation Runtime** | 5 min 39 sec | 4 min 14 sec | **-1 min 25 sec (Native Speedup)** |
| **Worst Negative Slack (WNS)** | -2.19 ns | -1.77 ns | +0.42 ns Slack Recovery |
| **Total Negative Slack (TNS)** | -92.21 ns | -85.06 ns | +7.15 ns Total Slack Restored |
| **Max Clock Frequency ($F_{max}$)** | 303.91 MHz | 348.80 MHz | **+44.89 MHz Performance Gain** |

 **Core Mechanical Insight:** The performance differences between the cloud container and the local build stem from variations in the underlying OpenROAD source versions, differing host hardware thread management, and distinct optimization paths chosen by the layout engines. The native build ran roughly 25% faster and achieved a higher operating frequency ($F_{max}$).

The final database file, located at `results/sky130hd/gcd/base/6_final.gds`, was verified in KLayout to ensure structural layout and manufacturing data integrity.

---

## Log Forensics, Report Scraping & Floorplan Density Audits

Post-flow analysis focused on extracting runtime metrics directly from the log files. Using recursive shell search utilities and pattern matching engines, changes in core density were tracked across the entire implementation pipeline:

```bash
# Locate and inspect synthesis statistics and timing logs
find . -name "*.log"
cat reports/sky130hd/gcd/base/synth_stat.txt
grep -i "tns" logs/sky130hd/gcd/base/*.log
grep -i "utilization" logs/sky130hd/gcd/base/*.log

```

### Footprint Sizing History

Tracking cell area metrics across the pipeline shows how physical structures systematically occupy the core's real estate:

```text
Initial Floorplan Utilization  ──> 43.0% (Whitespace Reserved)
  └─ Global Placement          ──> 53.8% (Logic Assigned)
       └─ Optimization Pass    ──> 64.7% (Buffering Added)
            └─ Timing Resizer  ──> 78.9% (Drive-Strength Sizing)
                 └─ Detailed CTS ──> 88.9% (Clock Buffers Placed)
                      └─ Routing ──> 95.3% (Final Signoff Layout Locked)

```

This structural tracking demonstrates that timing and area optimization is a continuous process. Gate resizing, timing buffer insertion, and clock tree placement occur at every stage of the backend flow, steadily expanding the layout density until final routing closure.

---

## Automated Physical Layout Visualizations

During execution, ORFS exports high-resolution graphical reports directly into the workspace. Below are the relative image paths linking to the layout visualizations generated during the automated build:

```markdown
![Final Integrated Layout](Images/final_all.webp.png)
*Figure 3: Combined final layout view displaying the fully assembled silicon die.*

![Standard Cell Placement Density](Images/final_placement.webp.png)
*Figure 4: Standard cell distribution map across the core rows prior to routing.*

![Final Routed Signal Interconnects](Images/final_routing.webp.png)
*Figure 5: Physical wire traces and interlayer via contacts generated by TritonRoute.*

![Clock Network Distribution Tree](Images/final_clocks.webp.png)
*Figure 6: Global clock distribution network mapping signal delivery pathways.*

![CTS H-Tree Schematic View](Images/cts_core_clock.webp.png)
*Figure 7: Schematic H-tree visualization tracking the balanced paths of the clock distribution network.*

![CTS Buffer Spatial Layout](Images/cts_core_clock_layout.webp.png)
*Figure 8: Physical layout map highlighting the strategic placement of clock buffers.*

![Localized Routing Congestion Heatmap](Images/final_congestion.webp.png)
*Figure 9: Congestion map validating that the interconnect paths stay safely below local layer bounds.*

![Static IR-Drop Supply Integrity Report](Images/final_ir_drop.webp.png)
*Figure 10: Voltage drop map showing stable power distribution across the core grid.*

![Timing Optimization Sizing Footprint](Images/final_resizer.webp.png)
*Figure 11: Gate sizing map showing the modifications made by the timing resizer.*

![Worst Timing Critical Path Layout](Images/final_worst_path.webp.png)
*Figure 12: Layout overlay highlighting the critical path driving the Worst Negative Slack.*

```

---

## Key Technical Takeaways

* **Automation Complements Architectural Insight:** While ORFS manages tool orchestration automatically using Makefiles, a deep understanding of each phase is still essential for debugging dependency errors and fixing timing violations.
* **Environment Configuration is Decisive:** Native software compilation highlights that an EDA tool's performance depends heavily on its build configuration. Properly setting environment variables and handling system dependencies is just as critical as the hardware logic itself.
* **ASIC Implementation is Non-Linear:** The steady increase in core utilization from 43% to 95.3% demonstrates that physical design is an iterative optimization process. Every stage inserts additional layout structures to safely close global timing and power requirements.

---

## Tooling Matrix

* **ASIC Orchestration System:** OpenROAD Flow Scripts Framework (GNU Make)
* **Logic Synthesis Engine:** Yosys Open SYnthesis Suite
* **Physical Design Engine Suite:** OpenROAD (incorporating TritonCTS and TritonRoute)
* **Multi-Corner STA Signoff:** OpenSTA Tool (Integrated Mode)
* **Layout Layout & GDSII Viewer:** KLayout Graphics Engine
* **Process Design Kit Library:** Google/SkyWater SKY130HD (High Density)
* **Build Configuration Tooling:** CMake / SWIG Sub-Process Engine
* **Execution Infrastructure:** Host Ubuntu 24.04 / GitHub Codespaces Sandbox
