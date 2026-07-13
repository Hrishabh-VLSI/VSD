# Week 3 (SoC): Functional Verification & System Integration (VSDSquadron)

## Overview

This phase moves the design workflow into **firmware-driven functional verification** and system integration. Using the open-source **VSDSquadron SoC**—built around the **VexRiscv RISC-V** processor core and integrated within the **Efabless Caravel** system harness—this curriculum implements block-level and system-level verification pipelines. Testing was executed across two distinct configurations: **Standalone Block Verification** to isolate peripheral logic, and **Caravel-Integrated System Verification** to analyze inter-module communications, bus contention, and hardware-software interactions under realistic operating conditions.

---

## System Architecture & Peripheral Mapping

The VSDSquadron SoC acts as an educational and production-ready reference platform for open-source silicon validation. It bridges the gap between individual, isolated RTL blocks and complete system-level chips by exposing standard industrial interfaces through a Wishbone interconnect fabric:

* **Processing Core:** 32-bit VexRiscv RISC-V CPU.
* **Primary Interconnect:** Standardized Wishbone Bus Architecture.
* **Memory Architecture:** On-chip SRAM alongside external SPI Flash execution units.
* **Peripherals:** SPI Master, UART Controller, Timer Modules, and Configurable GPIO Arrays.
* **Management & System Control:** Housekeeping SPI (`HKSPI`), Phase-Locked Loops (`PLL`), and Interrupt Request (`IRQ`) controllers.

---

## Unified Firmware-Driven Verification Flow

Unlike raw hardware testbenches that rely strictly on forced signal inputs, the VSDSquadron framework uses a **firmware-driven verification methodology** that mirrors actual silicon operation. The execution pipeline converts embedded C programs into behavioral hardware simulations through a multi-tier compilation and simulation engine:

```text
  [ C Firmware Source ]
           │
           ▼  (riscv32-unknown-elf-gcc)
  [ ELF Executable Binary ]
           │
           ▼  (riscv32-unknown-elf-objcopy)
  [ Intel HEX Memory Map ]
           │
           ├──> [ Icarus Verilog Compiler (iverilog) ] <── [ Peripherals + Caravel Wrapper RTL ]
           │                       │
           │                       ▼
           └───> [ Simulation Runtime Engine (vvp) ]
                                   │
                                   ├──> [ Value Change Dump (.vcd File) ] ──> [ GTKWave Analysis ]
                                   │
                                   └──> [ Console Logging Check ] ──────────> [ PASS / FAIL Signoff ]

```

The compiled C code is loaded directly into the simulated memory space of the SoC at startup. The RISC-V processor boots, executes the firmware instructions, reads/writes to peripheral registers across the Wishbone bus, and toggles specific monitor pins to log a `PASS` or `FAIL` status.

---

## Phase 1 & 2: Standalone Peripheral Verification Analysis

To establish a functional baseline, peripherals were isolated from the main processor wrapper and verified independently. This step isolates block bugs from system-level integration issues.

### Standalone Test Matrix & Coverage Status

| Isolated Test Target | Functional Verification Scope | Simulation Status | Failure Diagnostics / Behavioral Notes |
| --- | --- | --- | --- |
| `spi_master` | Validates clock polarity, phase control, and data shifting. | **PASS** | Data registers match expected transmit patterns perfectly. |
| `uart` | Checks baud rate division, start/stop bit generation, and parity. | **PASS** | Transmit/Receive lines synchronized properly under nominal rates. |
| `memory` | Validates SRAM read/write byte-enables and back-to-back cycles. | **PASS** | Data bus reads match written patterns with zero wait-states. |
| `gpio_mgmt` | Verifies directional input/output buffers and interrupt pads. | **PASS** | Direction registers configure pad states properly. |
| `timer` | Evaluates down-counter registers and expiration triggers. | **FAIL** | **Timeout Error:** Internal match condition failed to trigger. |
| `irq` | Verifies prioritized interrupt nesting and status registers. | **FAIL** | **Timeout Error:** Core remained trapped in an ISR loop. |
| `debug` | Checks JTAG interface tap states and debug register mapping. | **FAIL** | **Timeout Error:** Debug request lines failed to handshake. |

---

## Phase 3: Caravel Integrated System-Level Verification

Following standalone isolation, the peripheral blocks were integrated into the full **Caravel SoC** harness. This system-level phase evaluates structural connectivity, shared address decoding, clock distribution networks, and real hardware-software interaction bugs.

### Caravel System-Level Test Matrix

| System Integration Test | Primary Evaluation Target | Status | Architectural Takeaways |
| --- | --- | --- | --- |
| `user_pass_thru` | Bypasses core logic to link external Flash to internal buses. | **PASS** | SPI Flash data reads pass cleanly through the wrapper logic. |
| `uart` | Traces full processor-to-UART data streaming over Wishbone. | **PASS** | CPU instructions stream data without data loss or buffer overflows. |
| `sram_exec` | Verifies code execution directly out of the internal SRAM block. | **PASS** | Instruction fetch routines boot cleanly out of arbitrary RAM spaces. |
| `spi_master` | Drives external devices via Wishbone register routing commands. | **PASS** | SPI transfers execute smoothly with stable peripheral clock lines. |
| `pass_thru_fix` | Tests secondary bypass paths and structural logic corrections. | **PASS** | Confirms critical connection bugs are completely resolved. |
| `mem` | Audits system address space allocations across the memory map. | **PASS** | Memory space partitions isolate system and user sectors safely. |
| `hkspi_power` | Evaluates low-power configurations and register state isolation. | **PASS** | Core drops current consumption without losing system control. |
| `hkspi` | Checks direct programming paths via the external housekeeping link. | **PASS** | Registers update correctly without needing main CPU control. |
| `gpio_mgmt` | Evaluates CPU control over management pads. | **PASS** | System controls input/output pins dynamically via firmware. |
| `pullupdown` | Verifies internal pull-up and pull-down resistor trim settings. | **PASS** | Pad electrical states switch properly based on config values. |
| `sysctrl` | Validates global system reset and power-on initialization. | **FAIL** | **Timeout:** Power control loops failed to settle within runtime bounds. |
| `pll` | Checks clock multiplication, lock state indicators, and feedback. | **FAIL** | **Timeout:** PLL feedback loop failed to assert the lock flag. |

---

##  Key Technical Takeaways

* **System Integration Uncovers Hidden Roadblocks:** A design block can pass standalone testing perfectly yet fail completely inside a full system wrapper. System simulation is essential to catch real-world issues like Wishbone address conflicts and clock domain handshake faults.
* **Firmware-Driven Verification Replicates Real Silicon:** Using actual C code to test hardware bridges the gap between software development and chip verification. This ensures the peripheral registers match the software drivers exactly before physical manufacturing.
* **Timeout Triage Dominates Debug Cycles:** The system timeouts observed in the `PLL` and `SysCtrl` tests highlight that simulation limits must be carefully balanced against physical hardware stabilization times (such as PLL lock loops), requiring targeted configuration tweaks during verification.

---

##  Tooling Matrix

* **Verification Environment Orchestration:** GNU Make Engine Platform
* **Hardware Compiler Toolchain:** Icarus Verilog Compiler (`iverilog`)
* **Firmware Cross-Compiler Suite:** RISC-V Embedded Toolchain (`riscv32-unknown-elf-gcc`)
* **Simulation Runtime Engine:** Verilog Virtual Processor (`vvp`)
* **Waveform Forensic Engine:** GTKWave Waveform Visualization Tool
* **Target SoC Interconnect:** Efabless Caravel System Harness / Wishbone Bus
* **Target Embedded Core:** VexRiscv RISC-V CPU Core Framework
