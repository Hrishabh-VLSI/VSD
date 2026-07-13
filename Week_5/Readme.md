
# Week 5: Log Forensics, Report Scraping & Flow Artifact Auditing

## 📌 Overview

Week 5 shifts focus from execution mechanics to data forensics within the **OpenROAD Flow Scripts (ORFS)** workspace. This phase covers a detailed analysis of auto-generated reports, runtime logs, dependency tracking files, and configuration files. By utilizing native Linux shell search utilities and pattern matching engines, this analysis maps out how the system organizes backend design data, handles automated file tracking, and tracks changing density constraints.

---

## 📂 ORFS Data Hierarchy & Directory Structures

The automation framework organizes layout artifacts into distinct, self-contained directories. This separation keeps raw log traces detached from signoff reports:

```text
orfs_workspace/
├── build/             # Contains active design runs and intermediate cell netlists
├── logs/              # Raw tool streams tracking console syntax warnings and errors
├── reports/           # Static data files detailing wire profiles, slacks, and PPA metrics
└── results/           # Final physical database files (.gds, .def) ready for manufacturing

```

The `/reports/` directory maintains sub-folders tracking each physical milestone—including Floorplanning, Placement, Clock Tree Synthesis, Routing, and Signoff. These reports provide full visibility into the design's structural evolution across the backend flow.

---

## 🔍 Log Scraping & Data Mining Text Streams

To unpack data across thousands of lines of terminal output, native shell filters were deployed to scan the logs recursively:

```bash
# Recursively locate every system log file in the build workspace
find . -name "*.log"

# Display structural synthesis metrics directly to the terminal
cat reports/sky130hd/gcd/base/synth_stat.txt

```

### Pre-Placement Synthesis Reports

The `synth_stat.txt` manifest displays the complexity of the synthesized Greatest Common Divisor (GCD) core before physical layout begins:

* **Structural Metrics:** Wire tracking indexes, top-level port counts, and net bit-widths.
* **Cell Distribution Profiles:** Detailed counts mapping behavioral blocks onto standard boolean logic, multiplexer arrays, and basic gates.

---

## 🛠️ Dependency-Driven Flow Orchestration via GNU Make

To analyze the coordination layer behind the automated toolchain, the root build configuration was audited:

```bash
ls -la
less README.md
cat Makefile

```

The `Makefile` serves as the central control unit for ORFS. Instead of executing individual physical design tools step-by-step, the file maps out the complete pipeline by defining explicit rules:

1. **Execution Sequencing:** Enforces the proper progression of tasks, ensuring synthesis runs completely before floorplanning begins.
2. **Data Continuity Constraints:** Explicitly defines the input database files required by each stage and maps where the corresponding output artifacts are stored.
3. **Automated Tool Dependency Tracking:** Leverages GNU Make to monitor timestamps on layout files. If a modification is made to an early file, the system automatically rebuilds only the downstream steps that depend on it.

---

## 📊 Pipeline Utilization Auditing

To track how physical cells gradually occupy the core's silicon real estate, pattern-matching utilities were used to pull placement metrics across the run files:

```bash
# Extract Total Negative Slack (TNS) iterations throughout the run
grep -i "tns" logs/sky130hd/gcd/base/*.log

# Track physical die utilization shifts across sequential stages
grep -i "utilization" logs/sky130hd/gcd/base/*.log

```

### Footprint Density History

The extracted logs show how the area profile changes as optimization structures are added to the core:

| Implementation Stage | Core Area Utilization | Dominant Layout Modifications |
| --- | --- | --- |
| **Initial Floorplan** | 43.0% | Core boundaries fixed; empty whitespace reserved for routing. |
| **Initial Placement** | 53.8% | Standard cells mapped onto legal row coordinates. |
| **Global Placement** | 64.7% | Placer spreads cells uniformly to prevent wire congestion hotspots. |
| **Resized Placement** | 78.9% | Timing resizer inserts buffers to optimize critical paths. |
| **Detailed Placement** | 74.2% | Logic positions adjusted and legal grid alignments locked. |
| **Clock Tree Synthesis (CTS)** | 88.9% | Hierarchical clock buffer trees inserted to balance global skew. |
| **Final Routing Closure** | **95.3%** | Metal wires and vias routed, locking final density prior to signoff. |

> 📊 **Core Mechanical Insight:** Shifting from a 43% floorplan target to a dense 95.3% routed layout proves that physical design is an iterative optimization process. The layout density increases steadily because tools continuously insert timing buffers, gate resizers, and clock buffers to safely close performance and power margins.

---

## ⚙️ Path Management via Environment Variables

To manage binary dependencies and point the scripts to the correct executables, environmental variables are used to handle tool paths globally:

```bash
# Register the custom, natively compiled physical design binary path
export OPENROAD_EXE=$HOME/openroad/OpenROAD/build/bin/openroad

# Verify the active path mapping inside the terminal session
echo $OPENROAD_EXE

```

Using shell environment variables keeps automation scripts portable and modular. Instead of hardcoding absolute file paths inside layout macros, the automation framework looks up system variables to launch the right tools instantly. This makes it simple to switch between containerized environments and local source builds without rewriting the pipeline code.

---

## 💡 Key Technical Takeaways

* **Signoff Requires Deep Report Forensics:** Building a successful chip involves more than just passing scripts without errors. A clean GDSII requires reviewing text logs and parsing structural reports to confirm area metrics, timing slacks, and signal integrity.
* **Makefiles Provide Deterministic Layout Control:** Utilizing dependency-driven Makefiles ensures that complex physical design steps are highly repeatable and traceable across different computing environments.
* **Layout Sizing is an Inward Expansion:** Tracking cell utilization figures show that the physical layout expands inward. Leaving empty space early on is a deliberate strategy to provide the room required for downstream timing fixes and clock tree buffers.

---

## Tooling Matrix

* **ASIC Automation Framework:** OpenROAD Flow Scripts Platform (ORFS)
* **Pipeline Infrastructure Layer:** GNU Make Engine System
* **Core Layout Architectures:** Yosys Synthesis / OpenROAD Physical Core
* **Log Forensic Parsing Utilities:** Linux Shell Sub-System (`find`, `cat`, `less`, `grep`)
* **Environment Configuration System:** Bash Variable Mapping Terminal
* **Target Process Nodes Library:** Google/SkyWater SKY130HD (High Density)
