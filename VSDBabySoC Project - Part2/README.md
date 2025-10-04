# 🚀 VSDBabySoC: RISC-V with Analog Interface (Simulation & Synthesis Guide)

The **VSDBabySoC** is a compact System-on-Chip featuring a `rvmyth` RISC-V core, an `avsdpll` PLL, and an `avsddac` DAC. This guide covers the full flow from RTL (TL-Verilog) conversion through pre-synthesis functional verification to final gate-level timing-accurate simulation.

## 📄 Table of Contents

<details>
<summary><b>Project Summary & Architecture</b></summary>

* SoC Component Breakdown (`rvmyth`, `avsdpll`, `avsddac`)
* Project File Structure

  </details>

<details>
<summary><b>Step 1: Setup and Workspace</b></summary>

* Cloning the repository and directory structure.

  </details>

<details>
<summary><b>Step 2: TLV to Verilog Conversion</b></summary>

* Installing SandPiper-SaaS and converting `rvmyth.tlv` to `rvmyth.v`.

  </details>

<details>
<summary><b>Step 3: Pre-Synthesis Simulation</b></summary>

* Running the functional (RTL) check with Icarus Verilog.

  </details>

<details>
<summary><b>Step 4: Synthesize the Design (Yosys)</b></summary>

* Interactive Yosys commands to generate the gate-level netlist (`vsdbabysoc.synth.v`).

  </details>

<details>
<summary><b>Step 5: Post-Synthesis Simulation</b></summary>

* Running the timing-accurate simulation and troubleshooting module redefinition errors.

  </details>

<details>
<summary><b>Step 6: Troubleshooting & Command Summary</b></summary>

* Common issues (redefinition, shell errors) and the final command reference.

  </details>

---

## 🌟 SoC Architecture Summary

| Component       | IP Core   | Function                                                                         | Connection                   |
| :-------------- | :-------- | :------------------------------------------------------------------------------- | :--------------------------- |
| **Processor**   | `rvmyth`  | RISC-V core, outputs 10-bit digital data.                                        | Clocked by `CLK` from `PLL`. |
| **Clock Mgmt.** | `avsdpll` | Phase-Locked Loop, generates stable `CLK`.                                       | Supplies `CLK` to `rvmyth`.  |
| **Analog I/O**  | `avsddac` | 10-bit DAC, converts digital `RV_TO_DAC` to analog `OUT`. Controlled by `VREFH`. | Controlled by `VREFH`.       |

## 📁 Project File Structure

```
VSDBabySoC/
├── src/
│   ├── include/          # Header files (*.vh)
│   │   ├── sandpiper.vh
│   ├── module/
│   │   ├── vsdbabysoc.v      # Top-level RTL
│   │   ├── rvmyth.tlv        # RISC-V Core (Source)
│   │   ├── avsdpll.v         # PLL Behavioral Model
│   │   ├── avsddac.v         # DAC Behavioral Model
│   │   └── testbench.v       # Test environment
└── output/
    └── compiled_tlv/     # Holds compiled intermediate files if needed
```

---

## 🧩 Complete Step-by-Step Execution Guide

### 🧱 Step 1 — Create Project Workspace and Clone Repository

```bash
# Create a working directory
mkdir ~/VLSI (if creating freshly , orelse no need)
cd ~/VLSI

# Clone the BabySoC project
git clone https://github.com/manili/VSDBabySoC.git

# Enter the cloned repository
cd VSDBabySoC
```

### ⚙️ Step 2 — Convert RVMYTH (TLV) → Verilog

The `rvmyth` core is written in TL-Verilog and must be converted to `rvmyth.v` for simulation and synthesis.

```bash
# 1. Install Required Packages
sudo apt update
sudo apt install python3-venv python3-pip

# 2. Create and Activate a Virtual Environment
cd ~/VLSI/VSDBabySoC
python3 -m venv sp_env
source sp_env/bin/activate

# 3. Install SandPiper-SaaS
pip install pyyaml click sandpiper-saas
```

> 📝 **Note:**
> To use this environment in future sessions, always activate it first:
>
> ```bash
> source sp_env/bin/activate
> ```
>
> To exit:
>
> ```bash
> deactivate
> ```

```bash
# 4. Run TL-Verilog to Verilog Conversion
sandpiper-saas -i ./src/module/*.tlv \
               -o rvmyth.v \
               --bestsv --noline \
               -p verilog \
               --outdir ./src/module/
```

✅ **After running the above command**, `rvmyth.v` will be generated in the `src/module/` directory.
You can confirm this by listing the files:

```bash
spatha@spatha-VirtualBox:~$ cd VLSI/VSDBabySoC/
spatha@spatha-VirtualBox:~/VLSI/VSDBabySoC$ ls src/module/
avsddac.v  avsdpll.v  clk_gate.v  pseudo_rand_gen.sv  pseudo_rand.sv  rvmyth_gen.v  rvmyth.tlv  rvmyth.v  testbench.rvmyth.post-routing.v  testbench.v  vsdbabysoc.v
```

---

**After This Step - The Structure you will get**

```
VSDBabySoC/
├── src/
│   ├── include/          # Header files (*.vh)
│   │   ├── sandpiper.vh
│   ├── module/
│   │   ├── vsdbabysoc.v      # Top-level RTL
│   │   ├── rvmyth.v          # RISC-V Core (Source)
│   │   ├── avsdpll.v         # PLL Behavioral Model
│   │   ├── avsddac.v         # DAC Behavioral Model
│   │   └── testbench.v       # Test environment
└── output/
    └── compiled_tlv/     # Holds compiled intermediate files if needed
```


### 🧪 Step 3 — Run Pre-Synthesis Simulation

This verifies the functional correctness of the RTL before synthesis.

```bash
cd ~/VLSI/VSDBabySoC

# Compile using Icarus Verilog
mkdir -p output/pre_synth_sim
iverilog -o output/pre_synth_sim/pre_synth_sim.out \
    -DPRE_SYNTH_SIM \
    -I src/include -I src/module \
    src/module/testbench.v

# Run the simulation, generates pre_synth_sim.vcd
cd output/pre_synth_sim
./pre_synth_sim.out

# View in GTKWave
cd ~/VLSI/VSDBabySoC
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```
📊 **GTKWave Output**

<img width="1920" height="1080" alt="GTKwave presynthesis" src="https://github.com/user-attachments/assets/7258f4ae-5249-4995-9e6d-a20dc37d091c" />

⚙ **Change Data Format of OUT** 

<img width="1920" height="1080" alt="GTk Wave Setting" src="https://github.com/user-attachments/assets/30100c1b-cf2a-45fe-839e-ca8e8b615f0b" />

---
## **For Post-synthesis**

### 🧰 Step 4 — Synthesize the Design (Yosys)

This step generates the `vsdbabysoc.synth.v` gate-level netlist by mapping the RTL to standard cells defined in the Liberty files.

**Tool:** Yosys

```bash
cd ~/VLSI/VSDBabySoC
yosys
```

**Commands inside Yosys shell:**

```yosys
# Read RTL files (including the cores to be synthesized)
read_verilog -sv -I src/include/ -I src/module/ src/module/vsdbabysoc.v src/module/clk_gate.v src/module/rvmyth.v

# Read Standard Cell Library (for mapping logic gates)
read_liberty -lib /home/harshithapati/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read Behavioral/Analog IP Liberty Files (for black-boxing during synthesis)
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/avsdpll.lib

# Synthesize and map the design to standard cells
synth -top vsdbabysoc

# Optimization pass using ABC tool
abc -liberty /home/harshithapati/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Output the gate-level netlist
write_verilog vsdbabysoc.synth.v

# Optional: Display the synthesized schematic
show vsdbabysoc

# Exit Yosys
exit
```

📈 **Schematic of VSDBabysoc** 

<img width="1920" height="1080" alt="VSDBabySoc Schematic" src="https://github.com/user-attachments/assets/cda5a231-a25b-4101-a126-18020078db86" />

✅ Output file generated in the current directory: **vsdbabysoc.synth.v**

<img width="1920" height="1080" alt="Yosys Output" src="https://github.com/user-attachments/assets/47305e0a-ba6a-4f9d-b1e0-80625006913c" />

---

### 🧪 Step 5 — Post-Synthesis Simulation

This verifies the netlist **functionality and timing** (if SDF is available) using Icarus Verilog.

#### 5.1 Copy Library Files (Pre-requisite for Icarus)

The standard cell models (`primitives.v`, `sky130_fd_sc_hd.v`) are required by the gate-level netlist for simulation. We copy them locally so `iverilog` can easily find and link them as library components.

```bash
cd ~/VLSI/VSDBabySoC
cp ../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v src/module/
cp ../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v src/module/
```

#### 5.2 Compile for Post-Synthesis Simulation

```bash
cd ~/VLSI/VSDBabySoC
mkdir -p output/post_synth_sim

iverilog -o output/post_synth_sim/post_synth_sim.out \
    -DPOST_SYSTH_SIM \
    -I src/include \
    -I output/synthesized \
    src/module/testbench.v \
    output/synthesized/vsdbabysoc.synth.v \
    src/module/primitives.v \
    src/module/sky130_fd_sc_hd.v \
    src/module/avsddac.v \
    src/module/avsdpll.v
```

#### 5.3 Run Simulation and View Waveforms

```bash
cd output/post_synth_sim
./post_synth_sim.out

cd ~/VLSI/VSDBabySoC
gtkwave output/post_synth_sim/post_synth_sim.vcd
```
📊 **Post-Synth GTKWave Output**
<img width="1920" height="1080" alt="Post Synth GTKWave" src="https://github.com/user-attachments/assets/42539cef-8042-425f-babd-99015880a564" />

---

### 🧱 Step 6 — Troubleshooting & Command Summary

| Issue                                                       | Cause                                                                                     | Solution                                                                                                                          |
| :---------------------------------------------------------- | :---------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| **Module Redefinition**                                     | Compiling the synthesized netlist and original RTL simultaneously.                        | **Use `-y src/module` and `-v src/module/*.v`** to treat behavioral and standard cell files as libraries, avoiding duplicates.    |
| **Shell Errors** (`command not found`, `Permission denied`) | Inline comments in multi-line commands confuse the bash shell.                            | **Remove all inline comments (`#`)** from `iverilog` commands to ensure proper execution.                                         |
| **`Unknown module type: avsddac`**                          | Netlist instantiates a module (`avsddac`) whose definition was not found by the compiler. | **Fix:** Ensure **`-y src/module`** is included, as this tells the compiler where to search for the behavioral model definitions. |

### 🧾 Command Reference Summary

| Task                       | Command                                                                     |
| :------------------------- | :-------------------------------------------------------------------------- |
| **Activate `sp_env`**      | `source sp_env/bin/activate`                                                |
| **Convert TLV to Verilog** | `sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v ...`                      |
| **Run Pre-Synthesis Sim**  | `iverilog -o output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM ...`    |
| **Synthesize (Yosys)**     | `yosys` (then enter commands)                                               |
| **Run Post-Synth Sim**     | `iverilog -o output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM ...` |
| **View Waveform**          | `gtkwave <path_to_vcd_file>`                                                |
| **Deactivate `sp_env`**    | `deactivate`                                                                |

---
