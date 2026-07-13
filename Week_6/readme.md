# Week 6: RTL-to-GDSII Hardening & Gate-Level Simulation of Housekeeping SPI

## 📌 Overview

Week 6 focuses on the end-to-end block-level hardening and post-layout verification of the **Housekeeping SPI** (`housekeeping_spi`) peripheral from the Caravel SoC ecosystem. This implementation utilizes **OpenROAD Flow Scripts (ORFS)** targeted to the **Sky130HD PDK** to convert finite state machine (FSM) logic into a manufacturable layout. Post-layout structural integrity was signed off by running **Gate-Level Simulation (GLS)** to verify functional correctness against the baseline RTL description under realistic gate delays.

---

## 🏗️ Phase 1: Block Selection & Structural RTL Analysis

The `housekeeping_spi` module acts as the communication bridge linking an external SPI master to the interior housekeeping configuration registers of the Caravel SoC. It also manages pass-through data routing channels to both the management and user SPI Flash interfaces.

### Internal Finite State Machine (FSM) Architecture

The module relies on an explicit five-state FSM to decode incoming command bytes and route serial streams:

* **COMMAND:** Captures the initial 8-bit command stream to decode the transfer direction (Read/Write) and target profile.
* **ADDRESS:** Shifts in the 8-bit target register destination address.
* **DATA:** Manages parallel-to-serial (Read) and serial-to-parallel (Write) data bus shifting.
* **MGMTPASS:** Routes external SPI lines directly to the Management Flash pins.
* **USERPASS:** Links external SPI lines directly to the User Flash pins.

> ⏱️ **SPI Timing Alignment:** To maintain strict protocol alignment, incoming data bits on the Serial Data Input (`SDI`) pin are sampled on the positive edge of the SPI Clock (`SCK`). Outgoing bits on the Serial Data Output (`SDO`) line are launched on the negative edge of `SCK`, preventing setup or hold race conditions at the master receiver.

### Synopsys Design Constraints (`constraints.sdc`)

To ensure timing closure during technology mapping, the external serial clock input was constrained to operate at an aggressive block-level target frequency:

```tcl
create_clock -name SCK -period 10.0 [get_ports SCK]

```

A **10.0 ns clock period** translates to a **100 MHz operating speed**, providing a rigorous boundary target for the downstream placement and routing tools.

---

## 🚀 Phase 2: RTL-to-GDSII Physical Design Pipeline

The automated block-level hardening pipeline was launched via the standalone Make configuration:

```bash
make DESIGN_CONFIG=./designs/sky130hd/housekeeping_spi/config.mk \
     YOSYS_EXE=../dependencies/bin/yosys \
     OPENROAD_EXE=$HOME/openroad/OpenROAD/build/bin/openroad

```

### Step-by-Step Layout Footprint Evolution

The physical design engine adjusted the layout core bounding boxes at each successive milestone:

```text
[RTL Synthesis] ──> Mapped Verilog code to structural Sky130 standard cells
      │
      ▼
[Floorplanning] ──> Die Area: 2,278 µm²  |  Core Utilization: 9.0% (whitespace reserved)
      │
      ▼
[Cell Placement] ──> Die Area: 3,107 µm²  |  Core Utilization: 12.0% (legalized rows locked)
      │
      ▼
[Clock Tree (CTS)] ──> Die Area: 3,273 µm²  |  Core Utilization: 13.0% (clock buffers added)
      │
      ▼
[Signal Routing] ──> Detailed interconnect traces routed across multi-layer metal grids
      │
      ▼
[Cell Filling] ──> Appended 2,846 physical filler cells to secure layer planarity

```

---

## 📊 Phase 3: Signed-Off Implementation Outputs & PPA Summary

Following the completion of detailed routing and physical cell filling, the signoff reports were parsed to build the performance optimization matrix:

### Post-Routing Timing Summary

```ini
Worst Negative Slack (WNS)  = 0.00 ns (MET)
Total Negative Slack (TNS)  = 0.00 ns (MET)
Worst Setup Path Slack      = +3.11 ns (Clean Margin)
Minimum Clock Period        = 2.08 ns
Maximum Clock Speed (Fmax)  = 481.25 MHz

```

Achieving a positive worst slack of **+3.11 ns** means the block easily outpaces the 100 MHz constraint, confirming it is fully capable of running at a maximum frequency of **481.25 MHz** on silicon.

---

## 🧪 Phase 4 & 5: Mixed-Mode Gate-Level Simulation & Waveform Auditing

To verify that physical optimization did not alter the core logic functionality, a post-layout verification setup was configured using `iverilog` and `vvp`:

```text
Testbench (`housekeeping_spi_tb.v`) ──> Drives SPI read/write cycles and pass-through states
  ├── RTL Mode: Compiles behavioral Verilog files
  └── GL Mode: Compiles post-route netlist (`6_final.v`) + Sky130 Primitives

```

For Gate-Level Simulation (GLS), the technology primitive wrappers (`primitives.v`, `sky130_fd_sc_hd.v`) must be passed to the compiler to define the underlying hardware gates.

### Signal Timing Verification in GTKWave

Auditing the generated Value Change Dump (`.vcd`) traces confirms that the serial protocols match the target specifications perfectly:

```text
[CSB Asserted LOW] ──> SPI Transaction starts initialization sequence
  └── [SCK Toggles] ──> Commands/addresses shift into SDI register (MSB-First format)
        └── [Data Phase] ──> Internal registers decode read/write bytes successfully

```

While the external serial protocols match the RTL behavior, the interior signal traces show realistic post-layout gate delays. This demonstrates how physical routing parasitics change signal transition times without corrupting the active data.

---

## 🛠️ Phase 6 & 7: Cross-Verification Challenges & Debugging Insights

Running functional signoff on a physical netlist introduces verification issues that do not appear during behavioral testing.

### 📋 Post-Layout Resolution Log

* **Library Dependency Drops:** The initial GLS compilation failed immediately because the tool could not resolve structural cell names (e.g., `sky130_fd_sc_hd__dfxtp_2`). This was resolved by modifying the compilation scripts to explicitly link the Sky130 target technology cell libraries.
* **Internal Signal Obscurity:** During waveform debugging, tracking the internal FSM state registers directly was difficult because the synthesis engine had optimized the behavioral registers into logic gates. Verification was completed by shifting focus to audit the top-level boundary ports (`CSB`, `SCK`, `SDI`, `SDO`), confirming the external interfaces matched the protocol specification.
* **Makefile Target Automation:** The simulation infrastructure was simplified by adding a configuration flag to the central Makefile. This allows engineers to toggle between behavioral RTL and structural GLS modes using a single terminal variable, ensuring a clean, repeatable test pipeline.

---

## Tooling Matrix

* **ASIC Implementation Pipeline:** OpenROAD Flow Scripts Automation Suite (ORFS)
* **Logic Synthesis Engine:** Yosys Open SYnthesis Suite Suite
* **Physical Layout Optimizer:** OpenROAD (incorporating TritonCTS and TritonRoute)
* **Functional Verification Compiler:** Icarus Verilog Simulator Framework (`iverilog`)
* **Simulation Execution Runtime:** Verilog Virtual Processor Sub-System (`vvp`)
* **Waveform Forensic Analyzer:** GTKWave Visual Signal Workspace
* **Graphical Layout Viewer:** KLayout Layout Verification Suite
* **Process Node Library Target:** Google/SkyWater SKY130HD (High Density)

---
