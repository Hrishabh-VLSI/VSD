# Day 1: RTL Synthesis & Timing Analysis (PicoRV32A)

## Overview

Day 1 focuses on the critical transition where behavioral Verilog stops looking like software and is structurally transformed into silicon-ready gate-level hardware. Using the **OpenLane** automated ASIC flow, the **Yosys** synthesis engine, and **OpenSTA**, the **PicoRV32A RISC-V** core was mapped onto the **SKY130A PDK** to evaluate initial Power, Performance, and Area (PPA) metrics.

---

## Environment Initialization & Architecture

Before executing synthesis, the OpenLane container was initialized. OpenLane builds a strict, traceable workspace by loading the design configurations alongside the `sky130_fd_sc_hd` (High Density) standard cell library.

![OpenLane Initialization](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_1.png)


### The Run Directory Structure

Every execution spawns a isolated, unique timestamped directory. This strict separation ensures reproducibility and prevents data overwrites during iterative design exploration:

* `/logs/` – Detailed execution steps and tracking tool warnings/errors.
* `/reports/` – Raw analytical data detailing area, timing slack, and gate distribution.
* `/results/` – Synthesized output artifacts, including the final gate-level netlist (`.v`).
* `/tmp/` – Intermediate generic logic representations prior to technology mapping.

![Run Directory Structure](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_2.png)

---

## Synthesis Metrics & Physical Footprint

Running `run_synthesis` triggers logic optimization, technology mapping, and a single-corner pre-layout Static Timing Analysis (STA).

![Synthesis Invocation](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_3.png)


### Design Profile & Cell Utilization

Synthesis expands a relatively compact RTL codebase into thousands of physical components. The structural composition of the synthesized PicoRV32A core yielded the following metrics:

![Area Profile Report](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_5.png)


| Metric | Synthesized Value / Report Value |
| --- | --- |
| **Total Cell Count** | 15,762 cells |
| **Total Cell Area** | 160,816.736 $\mu m^2$ |
| **Sequential Elements (DFF)** | 1,613 cells |
| **Sequential-to-Total Ratio** | ~8.7% |

>  **Key Finding:** Flip-flops and state-holding elements account for less than 10% of the total design footprint. The remaining ~91.3% of the silicon area is entirely dedicated to combinational computation (logic gates, multiplexers, and arithmetic units).

---

##  Pre-Layout Timing Performance

To validate the temporal integrity of the netlist prior to physical placement, a single-corner Static Timing Analysis was completed using **OpenSTA**.

### Timing Slack Summary

![Flip-Flop Distribution](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_6.png)


```ini
Worst Negative Slack (WNS)  = 0.00 ns
Total Negative Slack (TNS)  = 0.00 ns
Worst Setup Slack           = 0.52 ns (MET)
Worst Hold Slack            = 0.16 ns (MET)

```

>  **Analysis:** The design successfully met all setup and hold timing constraints. The positive slacks indicate that signals safely propagate through the complex combinational paths and register stages within the targeted clock period, confirming a functionally and temporally valid netlist.

---

##  Netlist & Technology Mapping Discovery

Inspecting the final gate-level netlist demonstrates that behavioral descriptions (`always @` blocks, assignments) have been entirely replaced by explicit instances of the `sky130_fd_sc_hd` standard cells.


![Pre-Mapping Logic Analysis](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_7.png)


![Pre-Layout Static Timing Analysis](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_8.png)



![Gate-Level Netlist Snippet](https://raw.githubusercontent.com/Hrishabh-VLSI/VSD/9c4b5d11392128e5e4617495a68631db3ff9b2d2/Week_1/Images/Day_1_9.png)


```verilog
// Example snippet of the generated gate-level hardware description
sky130_fd_sc_hd__buf_2 _00034_ (
    .A(net_g1),
    .X(net_g2)
);

```

Technology mapping acts as a non-linear optimization pipeline. Yosys restructures and remaps generic boolean expressions continuously to optimize for the tightest path delay and minimal cell area.

---

##  Key Engineering Takeaways

* **ASIC Design is Multi-Objective:** Synthesis is never a basic text-to-gate compilation. The engine continuously runs an optimization matrix balancing area bounds, timing closure, and leakage power.
* **Downstream Dependency:** Frontend architectural choices and synthesis constraints set the absolute upper bound for the backend flow. Bad logic optimization here creates un-routable congestion or un-fixable setup violations on Day 2 and Day 4.
* **The GDSII Gateway:** The resulting structural Verilog file serves as the definitive hardware description required to proceed to Floorplanning and Clock Tree Synthesis.

---

##  Tooling Matrix

* **Flow Orchestration:** OpenLane Container Platform
* **Logic Synthesis & Tech Mapping:** Yosys Open SYnthesis Suite
* **Static Timing Analysis (STA):** OpenSTA (Integrated inside OpenROAD)
* **Process Design Kit (PDK):** Google/SkyWater SKY130A (130nm)
* **Development Environment:** Linux / GitHub Codespaces











