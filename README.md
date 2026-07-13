# Opensource RTL to GDSII ASIC Design

# About
A comprehensive, hands-on deep dive into the end-to-end open-source ASIC design flow, transforming digital logic into tape-out ready GDSII layouts.

Key Highlights:

1. RTL to GDSII Flow: Implemented and optimized designs using OpenLane, ORFS, and the SKY130 process node.

2. SoC Integration: Integrated designs into the Caravel SoC wrapper and worked with the VSDSquadron SoC ecosystem.

3. Advanced Verification: Conducted standalone verification and Mixed-Mode Gate-Level Simulations (GLS) to bridge the gap between RTL behavior and post-layout netlists.

4. Engineering Problem-Solving: Tackled real-world hardware description challenges, including timing closure, routing congestion, and environment configuration debugging.

![image alt](https://github.com/Hrishabh-VLSI/VSD/blob/2c1222d42d4a2d243415ccb7ddd5a33dc9f6c304/Week_1/Images/flow.webp)

An open-source RISC-V SoC integration harness developed by Efabless for the Open MPW program. It provides standard infrastructure (Wishbone, GPIO, memory, clocking) and a dedicated user project area, allowing designers to focus on custom hardware implementation and silicon-ready verification workflows.

# Repository Structure
Week_1 PicoRV32A ASIC Design Flow
Week_2 OpenROAD Flow Scripts (ORFS)
Week_3 SoC Block-Level Verification
Week_4 User Project Wrapper
Week_5 Mixed-Mode Gate-Level Simulation
Week_6 Independent ASIC Block Implementation

This repository is structured sequentially to mirror the standard ASIC implementation pipeline—from RTL to GDSII layout. Each folder provides a self-contained, highly reproducible documentation package containing the step-by-step implementation, exact commands used, generated reports, layout screenshots, and simulation waveforms. Additionally, each section highlights the specific challenges encountered, the debugging methodology applied, and key technical takeaways.

# Tools
OpenLane
OpenROAD
ORFS
Yosys
OpenSTA
Magic
KLayout
GTKWave
Icarus Verilog
Caravel SoC

# Technical Accomplishments
1. Mastered the complete open-source RTL-to-GDSII flow using OpenLane and ORFS on the SKY130 node.

2. Integrated custom digital modules into a complete RISC-V ecosystem via the Caravel SoC harness.

3. Validated post-synthesis netlists against timing and functional bugs using Mixed-Mode Gate-Level Simulation (GLS).

4. Developed rigorous engineering problem-solving skills by troubleshooting real-world EDA tool environments, routing bottlenecks, and timing violations.
