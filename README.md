# Week 8: Post-Layout STA & Timing Graphs Across PVT Corners for Routed VSDBabySoC 
 
The focus of this week is

---

## 📜 Table of Contents
[📋 Prerequisites](#-prerequisites) <br>
[1. Required Files](#1-required-files)<br>

---

## 📋 Prerequisites
- Basic understanding of Linux commands.
- Successful installation of the OpenROAD in [Week 5.](https://github.com/BitopanBaishya/VSD-Tapeout-Program-2025---Week-0.git)
- Successful completion of Physical Design Flow of VSDBabySoC till SPEF generation in [Week 7.]()
- Required Files for post-SPEF STA analysis. [Check here](#1-required-files)

---

## 1. Required Files 
This section gives a concise but clear breakdown of all the files needed for Post-Layout STA - what each file represents, how it was produced in earlier stages of the flow, and where it can be found within the project directory. This ensures that anyone following the flow can trace the origin of every input before running multi-corner STA.

### <ins>1. Gate-Level Netlist (Post-Route).</ins>
- File name: `vsdbabysoc_post_place.v`
- Generated in: Week 7 – After SPEF generation. [Check here]()
- Purpose: This Verilog netlist is the output after placement and routing, representing the final connectivity of all standard cells along with any routing-driven buffer insertions or optimizations done by OpenROAD. It reflects the “as-routed” logical structure of the chip.
- Path: `/home/bitopan/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v`

### <ins>2. SPEF File (Extracted Parasitics).</ins>
- File name: `vsdbabysoc.spef`
- Generated in: Week 7 – Parasitic Extraction (OpenROAD’s ARC or OpenRCX). [Check here]()
- Purpose: This file contains all extracted parasitics from the routed layout — wire resistance, net capacitances, coupling values, pin capacitances. Annotating this into OpenSTA enables real, RC-aware timing analysis instead of ideal wires.
- Path: `/home/bitopan/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef`

### <ins>3. Liberty Files for PVT Corners.</ins>
- File name:
  * `sky130_fd_sc_hd__tt_025C_1v80.lib`
  * `sky130_fd_sc_hd__ff_100C_1v65.lib`
  * `sky130_fd_sc_hd__ff_100C_1v95.lib`
  * `sky130_fd_sc_hd__ff_n40C_1v56.lib`
  * `sky130_fd_sc_hd__ff_n40C_1v65.lib`
  * `sky130_fd_sc_hd__ff_n40C_1v76.lib`
  * `sky130_fd_sc_hd__ss_100C_1v40.lib`
  * `sky130_fd_sc_hd__ss_100C_1v60.lib`
  * `sky130_fd_sc_hd__ss_n40C_1v28.lib`
  * `sky130_fd_sc_hd__ss_n40C_1v35.lib`
  * `sky130_fd_sc_hd__ss_n40C_1v40.lib`
  * `sky130_fd_sc_hd__ss_n40C_1v44.lib`
  * `sky130_fd_sc_hd__ss_n40C_1v76.lib`
- Generated in: Provided within the PDK.
- Purpose: Each `.lib` contains cell delays, transition tables, and setup/hold constraints for a specific Process-Voltage-Temperature (PVT) corner. OpenSTA loads these libraries to compute timing under worst, best, and typical operating conditions.
- Path: `/home/bitopan/VSD_Tapeout_Program/open_pdks/sources/sky130_fd_sc_hd/timing/`

### <ins>4. Timing Constraints (SDC File).</ins>
- File name: `4_cts.sdc`
- Generated in: Week 7 – During Clock Tree Synthesis. [Check here]()
- Purpose: Defines the design’s timing intent: clock definitions, I/O delays, false paths, multicycle paths, and all other STA constraints. This same SDC must be used consistently across synthesis, post-route STA, and multi-corner analysis.
- Path: `/home/bitopan/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/4_cts.sdc`

### <ins>5. Custom IP Liberty Files (PLL & DAC).</ins>
- File name:
  * `avsddac.lib`
  * `avsdpll.lib`
- Generated in: Provided by the VSDBabySoC design package
- Purpose: Models the timing of the DAC macro’s digital pins so that its interactions with the SoC’s digital logic are correctly timing-checked.
- Path: `/home/bitopan/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/avsddac.lib`

### <ins>6. Multi-Corner OpenSTA TCL Script.</ins>
- File name: `sta_across_pvt_route_custom.tcl`
- Generated in: Need to make it now.
- Purpose: Automates the entire STA flow — loading netlist, libraries, constraints, SPEF, defining corners, and running `report_checks` for both setup and hold across all corners. This script ensures repeatable, consistent analysis.
- Use the following content (modify the paths of files wherever required):
  ```
  ### ============================================================
  ###   VSDBabySoC  –  Post-Route STA  (Standalone OpenSTA)
  ### ============================================================

  puts "\n▶ Starting Post-Route STA for VSDBabySoC...\n"

  # --------------------------------------------------------------
  # Units
  # --------------------------------------------------------------
  puts "→ Setting units..."
  set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
  set_units      -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um


  # --------------------------------------------------------------
  # Liberty file list
  # --------------------------------------------------------------
  puts "\n→ Loading corner list..."

  set list_of_lib_files(1)  "sky130_fd_sc_hd__tt_025C_1v80.lib"
  set list_of_lib_files(2)  "sky130_fd_sc_hd__ff_100C_1v65.lib"
  set list_of_lib_files(3)  "sky130_fd_sc_hd__ff_100C_1v95.lib"
  set list_of_lib_files(4)  "sky130_fd_sc_hd__ff_n40C_1v56.lib"
  set list_of_lib_files(5)  "sky130_fd_sc_hd__ff_n40C_1v65.lib"
  set list_of_lib_files(6)  "sky130_fd_sc_hd__ff_n40C_1v76.lib"
  set list_of_lib_files(7)  "sky130_fd_sc_hd__ss_100C_1v40.lib"
  set list_of_lib_files(8)  "sky130_fd_sc_hd__ss_100C_1v60.lib"
  set list_of_lib_files(9)  "sky130_fd_sc_hd__ss_n40C_1v28.lib"
  set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
  set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
  set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
  set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"


  # --------------------------------------------------------------
  # Load analog macro Liberty files once
  # --------------------------------------------------------------
  puts "\n→ Loading DAC/PLL Liberty models..."
  read_liberty /data/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/avsdpll.lib
  read_liberty /data/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/avsddac.lib


  # --------------------------------------------------------------
  # Begin corner loop
  # --------------------------------------------------------------
  puts "\n============================================================"
  puts "              BEGIN PVT CORNER SWEEP"
  puts "============================================================\n"

  for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
      puts "\n------------------------------------------------------------"
      puts " Running Corner $i  →  $list_of_lib_files($i)"
      puts "------------------------------------------------------------"

      # Load Liberty for this corner
      read_liberty /data/VSD_Tapeout_Program/open_pdks/sources/sky130_fd_sc_hd/timing/$list_of_lib_files($i)

      # Reload design
      read_verilog /data/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_2_route.v
      link_design vsdbabysoc
      current_design

      # Constraints & parasitics
      read_sdc  /data/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/4_cts.sdc
      read_spef /data/VSD_Tapeout_Program/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef
    
      puts " → Timing graph built."
      puts " → Running setup/hold analysis..."

      # Full checks
      report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} \
          > /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/min_max_$list_of_lib_files($i).txt

      # WNS & TNS summary logs
      exec echo "\n$list_of_lib_files($i)" >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_worst_max_slack.txt
      report_worst_slack -max -digits {4} >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_worst_max_slack.txt

      exec echo "\n$list_of_lib_files($i)" >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_worst_min_slack.txt
      report_worst_slack -min -digits {4} >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_worst_min_slack.txt

      exec echo "\n$list_of_lib_files($i)" >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_tns.txt
      report_tns -digits {4} >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_tns.txt

      exec echo "\n$list_of_lib_files($i)" >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_wns.txt
      report_wns -digits {4} >> /data/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/STA_OUTPUT/route/sta_wns.txt

      puts " → Corner $i complete."
  }

  puts "\n============================================================"
  puts "              END OF MULTI-CORNER STA"
  puts "============================================================\n"

  exit
  ```
- File saved in this path: `/home/bitopan/VSD_Tapeout_Program/VSDBabySoC/OpenSTA/examples/BabySoC/sta_across_pvt_route_custom.tcl`


