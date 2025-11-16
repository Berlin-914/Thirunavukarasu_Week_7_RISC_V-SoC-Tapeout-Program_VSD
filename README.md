# 🚀 Week 7: BabySoC Physical Design Flow (OpenROAD + noVNC)

## 🌟 Overview

This repo documents the **complete physical design flow** for the BabySoC RISC-V project using OpenROAD, all executed via a browser-accessible XFCE desktop with noVNC in GitHub Codespaces. Learn how real RTL-to-GDSII design, debug, and parasitic-aware analysis is accomplished from scratch.

---

## 🏗️ Initial Project Setup

**1. Create Project and Source Directories**

mkdir -p OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/src


**2. Copy Design Collateral**
- Place all Verilog source files in:

OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/src/

- Add required subfolders to `vsdbabysoc`:
- `gds/` — (e.g. avsdbasic.gds, avsdp1.gds)
- `include/` — (e.g. sandpiper_gn.vh, sp_verilog.vh)
- `lef/` — (e.g. avsdbasic.lef, avsdp1.lef)
- `lib/` — (timing libs)
- Place `vsdbabysoc_synthesis.sdc` constraint file in the design directory.

**3. Add Constraints and Configuration**
- Add your own:
- `macro.cfg`
- `pin_order.cfg`
- Create a `config.mk` as shown below. Adjust to match your file structure:

export DESIGN_NICKNAME = vsdbabysoc
export DESIGN_NAME = vsdbabysoc
export PLATFORM = sky130hd

export VERILOG_FILES = $(wildcard $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/*.v)
export SDC_FILE = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/vsdbabysoc_synthesis.sdc

Add folder references for include, lef, lib, gds as needed
export VERILOG_INCLUDE_DIRS = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/include
export ADDITIONAL_LEFS = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/lef/avsdbasic.lef
export ADDITIONAL_GDS = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/gds/avsdbasic.gds
export ADDITIONAL_LIBS = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/lib/avsdbasic.lib

export DIE_AREA = 0 0 1000 1500


---

## 🌐 Using OpenROAD via Codespace + noVNC

Setting up OpenROAD natively on Ubuntu can fail due to dependency, CMake, and Python errors. With **noVNC Codespace**, everything just works—no installations to break:

**A) Launch Codespace and Connect XFCE Desktop**
- On GitHub:  
  `Code` → `Codespaces` → `Create codespace on main`

**B) From the Codespace Terminal:**

./helpers/start_desktop.sh

- Click/open the noVNC URL displayed in the console to access XFCE desktop from your browser!

---

## 🛠️ OpenROAD Physical Design Flow: Commands for Each Stage

All commands are executed in a terminal within the XFCE desktop (noVNC session):

<details>
  <summary><strong>Floorplanning</strong></summary>
  
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan

</details>

<details>
<summary><strong>Placement</strong></summary>

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place

</details>

<details>
<summary><strong>Routing</strong></summary>

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_route

</details>

<details>
<summary><strong>Post-Route SPEF Extraction</strong></summary>

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk spef

</details>

---

## 🖼️ Snapshots

View-stage results (floorplan, placement, routing, etc.) are stored in the [`Week7_Snapshots`](./Week7_Snapshots) folder. These provide visual verification at every design stage.

---

## ⚡ Key Observations & Challenges

- **Tool Installation:** OpenROAD native install often failed on Ubuntu due to blocker dependencies and complex CMake issues. Codespace + noVNC solved all setup pain.
- **GUI Access:** XFCE/noVNC allowed both CLI scripting and live GUI view—far more effective than headless/SSH workflows.
- **Design Tweaks:**  
- *Floorplanning*: Macro congestion and poor initial utilization needed repeated boundary and macro edits.
- *Placement*: Legalization and timing constraints required iterative runs.
- *Routing*: Initial DRC shorts/opens fixed by parameter tuning and reruns.
- *SPEF*: Only post-route RC values give trustworthy STA. After each physical tweak, rebuilt SPEF for signoff.
- **Performance:** Codespaces are slower to start but offer a 100% reproducible EDA lab anywhere.

---

## 🔍 What is SPEF? Why Does STA Need It?

**SPEF (Standard Parasitic Exchange Format)** captures all net/pin parasitics (RC) after actual layout:
- **Why it matters:**  
- Real-world wire and pin delays depend on post-route RC, not library timing.
- Accurate STA only possible when SPEF is provided—otherwise signoff could be misleading.
- Always rerun SPEF after any physical design change and before final STA.

**Typical use:**
- Run after `make ... route`
- Analyze with your favorite STA tool.

---

## 📊 Example Workflow Summary

| Stage        | Command Example                                 | Description                      | Typical Challenges         |
|--------------|-------------------------------------------------|----------------------------------|---------------------------|
| Floorplan    | `make ... floorplan`                            | Define die, input/output regions | Macro congestion          |
| Placement    | `make ... place`                                | Arrange cells, macros            | Legalization, timing      |
| Routing      | `make ... route`                                | Connect all nets/wires           | DRC, shorts/opens         |
| SPEF Extract | `make ... spef`                                 | Output post-route parasitics      | Must re-run after changes |

---


