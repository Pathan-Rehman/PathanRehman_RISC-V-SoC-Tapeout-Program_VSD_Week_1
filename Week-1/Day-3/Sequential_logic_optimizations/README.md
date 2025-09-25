## Lab Introduction

This lab introduces **sequential optimization techniques** using a set of example circuits (named `dff_const*`) implemented in Verilog. The main objectives are to analyze, simulate, and synthesize each circuit, observing how synthesis tools optimize sequential elements, specifically flip-flops under various constant input conditions.

In this lab, you will:
- Inspect Verilog designs and identify their sequential behavior.
- Simulate the designs with testbenches and examine the resulting waveforms.
- Synthesize the circuits using USYS, observe mapping to standard cells, and understand the optimization decisions.
- Analyze the impact of reset/set polarities and constant assignments on flip-flop retention and removal.
- As exercises, extend your understanding to similar designs (`dff_const4`, `dff_const5`).

## Lab Steps

### 1. Examine `dff_const1.v`

- Open `dff_const1.v` to inspect the design for sequential elements.
- Observe the always block written as:
  ```verilog
  always @(posedge clk, posedge reset)
      if (reset)
          Q <= 1'b0;
      else
          Q <= 1'b1;
  ```
  <img width="580" height="253" alt="image" src="https://github.com/user-attachments/assets/1d9261d3-6a56-49f0-9614-2e61db484fdb" />


- Notation:
    - `Q` is set to $0$ when reset is asserted.
    - On all other clock edges, $Q$ is set to $1$.
    - The D input is tied to constant $1'b1$.

#### Signal Analysis

- When reset is de-asserted, $Q$ waits for the next clock edge to become one (not immediately on reset release).
- This is not an inverter between reset and $Q$ but a **clock-synchronized flop**.
- The circuit is expected to infer a D Flip-Flop due to synchronization to the clock.

### 2. Simulate `dff_const1.v`

- Use the associated testbench: `tb_dff_const1.v`.
- Compile the design:
  ```bash
  iverilog dff_const1.v tb_dff_const1.v
  ```

- Run the simulator:
  ```bash
  ./a.out
  ```
  <img width="569" height="76" alt="image" src="https://github.com/user-attachments/assets/66647aca-09c9-4d9b-9f62-b276a141e82b" />


- Observe the waveform using GTKWave:
  ```bash
  gtkwave tb_dff_const1.vcd
  ```
  <img width="1074" height="832" alt="image" src="https://github.com/user-attachments/assets/1afa6050-5e4c-4e4a-a2db-555a3c60aaae" />


- Key observation: $Q$ becomes $1$ only at the **next rising edge** of the clock after reset is de-asserted.

### 3. Synthesize `dff_const1.v`

- Launch yosys.
- Load the standard cell library:
  ```shell
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  ```
  <img width="738" height="83" alt="image" src="https://github.com/user-attachments/assets/8411abf8-7559-4831-a22c-f70e832db39b" />


- Read the design:
  ```shell
  read_verilog dff_const1.v
  ```
  <img width="647" height="103" alt="image" src="https://github.com/user-attachments/assets/56b8c38e-bd82-4890-b7e3-164efa4e2362" />


- Synthesize specifying the top module:
  ```shell
  synth -top dff_const1
  ```
  <img width="606" height="394" alt="image" src="https://github.com/user-attachments/assets/76d3bfe3-cfe2-4c00-9fe2-006864b79564" />


- Perform DFF library mapping:
  ```shell
  dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  ```
  <img width="801" height="82" alt="image" src="https://github.com/user-attachments/assets/1214c943-4144-43c2-9e1b-953c2af08c1e" />


- Technology map the design:
  ```shell
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  ```
  <img width="592" height="154" alt="image" src="https://github.com/user-attachments/assets/01901e89-995a-4bd7-bd79-b07fd8b273e6" />


- Visualize the resulting netlist:
  ```shell
  show
  ```
  <img width="1065" height="208" alt="image" src="https://github.com/user-attachments/assets/3f3784d5-243d-4ecc-b4e4-7040945aae77" />


- **Note:** Library expects reset to be active-low, but the code uses active-high. An inverter is inserted on the reset path.

### 4. Examine and Simulate `dff_const2.v`

- Open and review `dff_const2.v`.
  
  <img width="819" height="272" alt="image" src="https://github.com/user-attachments/assets/deb78a3d-230a-4b63-a451-9c938d66e15f" />

- Observe: Both reset and D input result in $Q$ always being $1$ (constant value regardless of reset).
- Simulate as before using the appropriately named testbench:
  ```bash
  iverilog dff_const2.v tb_dff_const2.v
  ./a.out
  gtkwave tb_dff_const2.vcd
  ```
  <img width="1077" height="835" alt="image" src="https://github.com/user-attachments/assets/78523548-2a8b-4bd3-8901-28c66f99b3d9" />


- $Q$ remains high at all times.

### 5. Synthesize `dff_const2.v`

- Read the Verilog file:
  ```shell
  read_verilog dff_const2.v
  ```
  <img width="652" height="129" alt="image" src="https://github.com/user-attachments/assets/a37c52dd-bab7-45c9-a133-dfc3f667f211" />


- Synthesize:
  ```shell
  synth -top dff_const2
  ```
  <img width="600" height="380" alt="image" src="https://github.com/user-attachments/assets/d13fb4f9-d49a-446e-b416-5e95ee7044f6" />


- Technology map (no need to repeat `dfflibmap` if in same shell):
  ```shell
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  show
  ```
  <img width="807" height="129" alt="image" src="https://github.com/user-attachments/assets/79b8144c-2481-4850-9f4e-4e7d7438b561" />
<img width="609" height="653" alt="image" src="https://github.com/user-attachments/assets/242a251f-5dfb-405b-a33c-43fafc3aee55" />


- Observe: **No flip-flops inferred.** The output is just the constant $1$ connected to $Q$, with clock and reset unused.

### 6. Compare Synthesis Reports

- For `dff_const1.v`, synthesis infers one DFF in the statistics.
- For `dff_const2.v`, synthesis infers zero DFFs, wires, or memories.

### 7. Explore `dff_const3.v`

- Open and review the design in `dff_const3.v`.
  <img width="814" height="452" alt="image" src="https://github.com/user-attachments/assets/53e6950d-6094-4177-b16f-0c76ae717fc1" />

- Contains two flip-flops, sharing clock and reset.
    - $Q1$ is a reset flop: goes to $0$ upon reset, then $1$ at next clock edge.
    - $Q$ is a set flop: $Q$ goes to $1$ upon reset; afterwards, $Q$ gets value from $Q1$.

#### Functional Analysis

- Simulate design using:
  ```bash
  iverilog dff_const3.v tb_dff_const3.v
  ./a.out
  gtkwave tb_dff_const3.vcd
  ```
  <img width="1076" height="833" alt="image" src="https://github.com/user-attachments/assets/dd145777-04e6-482d-b4d9-e32276fad40a" />


- Pull both $Q$ and internal $Q1$ into GTKWave for inspection.
- On reset, $Q1$ goes to $0$, becomes $1$ on next clock; $Q$ goes to $1$ upon reset, temporarily drops for one clock, then follows $Q1$.

#### Synthesize `dff_const3.v`

- In yosys:
  ```shell
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog dff_const3.v
  synth -top dff_const3
  dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  show
  ```
  <img width="1067" height="214" alt="image" src="https://github.com/user-attachments/assets/5af2610e-495d-4e22-a383-727e3eb86aec" />


- Expectation: **Two flip-flops** are inferred and neither can be optimized away, despite constant connections.

- Schematic review: Both standard-cell DFFs appear, one with reset logic, the other with set logic, including inverters as required by library polarity.

### 8. Lab Extension: Exercises

- For practice, repeat similar procedures for `dff_const4.v` and `dff_const5.v`. Follow the above steps: inspect, simulate, synthesize, and analyze the netlists/schematics.
- Explore how reset/set polarities and logic modifications impact sequential optimization.

---

**End of lab.**
