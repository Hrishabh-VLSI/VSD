
# Week 3: Native EDA Compilations & Host Environment Engineering

## 📌 Overview

Week 3 focuses on migrating away from containerized sandboxes to build the **OpenROAD** physical design platform natively from source code on an **Ubuntu 24.04 LTS** host system. This phase explores the internal architecture of large-scale Electronic Design Automation (EDA) frameworks, uncovering compilation dependencies, resolving interface wrapper generation tool version conflicts, and validating custom binaries.

---

## 🔬 Containerized Sandboxing vs. Native Environments

Relying on preconfigured Docker environments inside automation suites like OpenROAD Flow Scripts (ORFS) hides the underlying system architectures. Prior to source compilation, an evaluation of the host system's native EDA capabilities was performed:

```bash
which openroad
openroad --version

```

The initial command returned no valid paths, indicating a clean host system.

An installation attempt using standard binary setup scripts failed due to system package mismatches, missing shared runtime paths, and absent graphical toolkit elements (`Qt5`) and specialized `Python` extension runtimes.

### Environment Status Matrix

| Infrastructure Platform | OpenROAD Execution Status | Structural Environment Realities |
| --- | --- | --- |
| **Host Ubuntu 24.04 OS** | Not Installed | Bare runtime environment; no native EDA configurations present. |
| **Binary Setup Script** | **FAILED** | Aborted due to missing shared libraries, dynamic links, and Qt/Python clashes. |
| **ORFS Dev Container** | Functional | Isolated, pre-built sandbox packaging verified production binaries. |

---

## 🛠️ Repository Provisioning & Dependency Engineering

To initiate a custom native build, the complete OpenROAD source hierarchy was cloned recursively from GitHub to capture all structural submodules:

```bash
git clone --recursive https://github.com/TheOpenROADProject/OpenROAD.git
cd OpenROAD

```

The repository footprint extends beyond **900 MB**, enclosing core optimization engines, unit test benches, and third-party mathematical wrappers.

To link the required system dependencies, the internal automated provisioning utilities were launched:

```bash
sudo ./etc/DependencyInstaller.sh

```

This routine installs a complex ecosystem of developer packages, linking compiler toolchains and math libraries:

* **Build Tools:** `CMake` build generators, `Bison` parser setups, and `Flex` lexical analyzers.
* **Core Libraries:** `Boost` C++ utility arrays, `Eigen` template algebras, and `Abseil` common runtimes.
* **Optimization Frameworks:** Google `OR-Tools` linear programming routines.

---

## ⚙️ SWIG Version Conflict Diagnostics & Resolution

During the initial build initialization, the compiler suite threw a fatal environmental dependency error:

```text
CMake Error at cmake/FindSWIG.cmake:
  Could NOT find SWIG:
  Found unsuitable version "4.2.0"
  Required version is at least "4.3"

```

### Diagnostic Investigation

Although the dependency utilities successfully compiled the Simple Wrapper and Interface Generator (SWIG) v4.3.0 into localized directories, the system's active environment path was still discovering an older SWIG binary (v4.2.0) packaged by the base operating system.

### Engineering Resolution

To bypass the host OS path priority and enforce the correct version, the active shell environment path variable was updated to point directly to the new binary location:

```bash
export PATH=/usr/local/bin:$PATH
swig -version

```

```text
SWIG Version 4.3.0
Compiled with C++17 optimization hooks.

```

Re-running the configuration engine picked up the updated tool path immediately, allowing pre-compilation system validation checks to pass cleanly.

---

## 🚀 Native Compilation & Binary Verification

With all dependency conflicts resolved, the source tree compilation was initiated across all available processor cores:

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)

```

The system executed the compilation loops without errors, locking down the final executable target:

```text
[100%] Built target openroad

```

### Build & Run Diagnostics

* **Total Compilation Time:** **3 hours 23 minutes (202m 57s)** of continuous multi-threaded execution.
* **Active Binary Generation:** Located at `./src/openroad`.

The freshly generated custom executable was verified by launching its interactive shell configuration:

```text
OpenROAD 26Q2-1895-gba65a4e291
Features enabled: +GPU +GUI +Python

```


*Figure 1: Successful invocation of the locally compiled OpenROAD interactive terminal.*

The operational shell confirms that the physical design engine is fully functional, with hardware GPU acceleration, full graphical layout visualization (GUI), and Python scripting bindings compiled cleanly into the tool.

---

## 📋 Compilation Challenge Log

| Identified Roadblock | Root Cause | Selected Engineering Resolution |
| --- | --- | --- |
| **Missing Shared Libraries** | Broken links during script execution. | Abandoned pre-built binaries for a clean source compilation. |
| **Absent Git Submodules** | Cloned without secondary trees. | Executed `git submodule update --init --recursive`. |
| **Unsuitable SWIG Tooling** | OS path mapped to an older SWIG version. | Reconfigured the `PATH` environment variable in the terminal. |
| **Compiler Configuration Drop** | Dynamic library discovery errors. | Ran the explicit `DependencyInstaller.sh` routine. |

---

## 💡 Key Technical Takeaways

* **Containers are for Execution, Source is for Understanding:** While containerized sandboxes are convenient for running automated flows, compiling tools from source code clarifies how different software components interact.
* **Path Management Controls the Environment:** Software compilation depends heavily on environment paths. Simply installing a package isn't enough; system variables must be configured properly so the compiler finds the correct tool versions.
* **EDA Infrastructure is Dominated by Math Frameworks:** Physical design tools are built on advanced mathematics. OpenROAD relies on a deep foundation of open-source frameworks like Google OR-Tools and Eigen to solve complex routing and placement algorithms.

---

## Tooling Matrix

* **Physical Design Base Platform:** OpenROAD Open-Source Source Tree Suite
* **Build System Orchestration:** CMake Engine / GNU Make Engine
* **Interface Generation Layer:** SWIG v4.3.0 Custom Build
* **Algorithmic Optimization Suite:** Google OR-Tools Pipeline
* **Mathematical Template Matrix:** Eigen C++ Algebra Library
* **Host Development System:** Linux Ubuntu 24.04 LTS Platform
