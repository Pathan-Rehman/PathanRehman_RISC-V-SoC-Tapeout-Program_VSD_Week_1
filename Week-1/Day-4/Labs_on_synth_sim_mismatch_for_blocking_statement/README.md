## Lab Introduction

This lab demonstrates a synthesis and simulation mismatch in Verilog due to the use of blocking assignments. The aim is to implement the logic function $Y = (A \lor B) \land C$ and observe how blocking statements in Verilog can result in different behavior between simulation (RTL) and synthesis (GLS). The steps include analyzing the Verilog code, running RTL simulation, synthesizing the design, generating and inspecting the netlist, and running gate-level simulation to compare the results.

## Lab Steps

### 1. Review and Understand the Objective

- **Aim:** Implement the logic $Y = (A \lor B) \land C$ using Verilog.
- **Note:** Blocking assignments are used, which can cause simulation-synthesis mismatches.
- The intermediate signal $X$ is computed as $X = A \lor B$, and $D$ (output) as $D = X \land C$.

### 2. Examine the Verilog Source

- The relevant file is `blocking_caveat.v`.
- The code structure applies blocking assignments. This results in $D$ being assigned using the previous value of $X$ rather than its freshly computed value.
- The effect is as though there is an unintended register (flop) on $X$.

<img width="659" height="440" alt="image" src="https://github.com/user-attachments/assets/989bca4d-80b8-431b-9de5-bb78db3e9879" />


### 3. Run RTL Simulation

- **Command:**
  ```sh
  iverilog blocking_caveat.v tb_blocking_caveat.v -o a.out
  ```
  <img width="817" height="71" alt="image" src="https://github.com/user-attachments/assets/a684d0c6-6f94-4d32-be3f-36fad2917244" />


- **Run simulation output:**
  ```sh
  ./a.out
  ```
  <img width="823" height="98" alt="image" src="https://github.com/user-attachments/assets/e1e6f64d-3901-47b7-838b-8a945fc55fd1" />


- **View Results in GTKWave**
  ```sh
  gtkwave tb_blocking_caveat.vcd
  ```
  <img width="1073" height="495" alt="image" src="https://github.com/user-attachments/assets/fcfea76e-5bf0-4069-820d-48cfb783717f" />


- In GTKWave, observe signals:
  - Inputs: **A**, **B**, **C**
  - Output: **D**
  - Note that **X** is intermediate and may be skipped in waveform viewing.

- **Key Observation:**
  - At time steps where both $A = 0$ and $B = 0$, $X$ should be $0$. $D$ should be $X \land C$.
  - In the waveform, see if $D$ ever reflects a value as if it used a previous value of $X$ (simulation result may incorrectly appear as if there's a flop).

### 4. Synthesize the Design

- Open the synthesis tool.

- **Commands:**
  ```sh
  yosys
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog blocking_caveat.v
  synth -top blocking_caveat
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  show
  ```
  <img width="646" height="150" alt="image" src="https://github.com/user-attachments/assets/f6513e5e-c553-49ee-88e1-3c0f9da75d35" />
<img width="594" height="178" alt="image" src="https://github.com/user-attachments/assets/6f0c05fa-d096-437d-9796-9e2155cb9121" />


- **Write netlist:**
  ```sh
  write_verilog -noattr blocking_caveat_net.v
  ```
  <img width="660" height="573" alt="image" src="https://github.com/user-attachments/assets/66492326-9ae6-488e-b199-f6651157b258" />


### 5. Inspect the Synthesized Netlist

- Examine `blocking_caveat_net.v` to see if any latches or flops have been inferred.
- Expected: The synthesized netlist should only use combinational logic gates (OR, AND), not sequential elements.
- The logic implements $Y = (A \lor B) \land C$ as expected, without any unintended latches.


### 6. Run Gate-Level Simulation (GLS)

- Simulate the synthesized netlist using the same testbench.

- **Command:**
  ```sh
  iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v -o a.out
  ```
  <img width="817" height="92" alt="image" src="https://github.com/user-attachments/assets/b98f466a-8a96-46f3-8609-b9086c2c4759" />


  ```sh
  ./a.out
  ```
  <img width="822" height="95" alt="image" src="https://github.com/user-attachments/assets/0076e347-6508-455b-9634-636e7c7c40cd" />


- **View Results in GTKWave**
  ```sh
  gtkwave tb_blocking_caveat.vcd
  ```
  <img width="1064" height="569" alt="image" src="https://github.com/user-attachments/assets/fdfe6c13-decf-485b-9731-9907b47edbb2" />


- Confirm netlist-based design is loaded (GLS) — look for indications in scope/signals hierarchy (e.g., module names contain underscores or standard cell identifiers).

- **Observation:**
  - The gate-level waveforms reflect pure combinational logic behavior: output $D$ always matches $(A \lor B) \land C$ without reference to previous cycle values.

### 7. Compare and Analyze the Results

- **RTL Simulation:** May show a mismatch—output $D$ appears as if it uses a delayed (previous) value for $X$ due to blocking assignments in the code.
- **GLS (Gate-Level):** Correctly implements $D = (A \lor B) \land C$ with no latches; only instantaneous values are evaluated.
- **Conclusion/Takeaway:** Synthesis and simulation mismatch occurs due to misuse of blocking assignments. Always exercise caution and clarity when using blocking assignments in sequential logic; unintended bugs may arise such as ghost latches/flops during simulation only.


### 8. Final Note

- Blocking statements in Verilog should be avoided unless clearly needed.
- Double-check design intent against synthesized logic.
- Always simulate both RTL and gate-level designs to catch such mismatches.
