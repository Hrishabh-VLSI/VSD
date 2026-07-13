# ASIC Design Flow using OpenLane (SKY130)

This directory documents the end-to-end RTL-to-GDSII physical design implementation of the PicoRV32A RISC-V processor core. The design was hardened using the OpenLane flow and mapped to the SKY130A PDK node. The project highlights how a high-level hardware description evolves into a tape-out ready physical layout through iterative optimization of timing, power, and area (PPA) constraints.

# The Implementation Pipeline

| Timeline | Focus Stage | Key Milestones & Deliverables | Core Metrics & Analysis |
| :--- | :--- | :--- | :--- |
| **Day 1** | **RTL Synthesis** | • Technology mapping to standard cells<br>• Gate-level netlist generation | • Structural cell count profiling<br>• Pre-layout area & timing analysis |
| **Day 2** | **Floorplanning & Placement** | • Core/Die area utilization sizing<br>• Tap/IO cell distribution & macro placement | • Aspect ratio definition<br>• Global vs. detailed placement density |
| **Day 3** | **Custom Cell Integration & STA** | • Custom `LEF` / `Liberty` file injection<br>• Post-synthesis netlist optimization | • Multi-corner Static Timing Analysis<br>• Constraint & slack engineering |
| **Day 4** | **Clock Tree Synthesis (CTS)** | • Clock distribution network construction<br>• Clock buffer insertion | • Skew & max fanout minimization<br>• Pre-CTS vs. Post-CTS setup/hold timing |
| **Day 5** | **PDN, Routing & Signoff** | • Power Distribution Network (`PDN`) generation<br>• Global & detailed signal routing | • Parasitic extraction (`SPEF`) generation<br>• Physical signoff: `DRC` & `LVS` checks |

# Core Engineering Takeaways

  1. Gained practical, hands-on control over the physical layout pipeline using industry-relevant open-source EDA tools.

 2. Developed an understanding of the iterative nature of chip design—analyzing how synthesis constraints impact downstream routing density and timing slack.

  3. Mastered both front-end evaluation and back-end signoff verification paradigms.
