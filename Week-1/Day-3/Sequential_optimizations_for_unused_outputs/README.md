## Lab Introduction

This lab demonstrates **unused output optimization** during hardware synthesis using Yosys, focusing on a simple 3-bit counter example. You will observe how the synthesis tool eliminates unused signals and optimizes circuits based on actual output dependencies. The lab steps include exploring two variations of Verilog code, synthesizing with Yosys and Sky130 libraries, and analyzing RTL changes and schematic reductions.

## Lab Steps

### 1. Initial Counter Design Overview

- The given Verilog module (`counter_opt`) implements a 3-bit up counter with inputs: **clock**, **reset**, and output **Q**.
- Internally, it has a `count` register (`count[2:0]`), incremented every clock cycle unless reset.
- In the initial version, only `count` feeds the module output `Q`, with `count[1]` and `count[2]` unused.

Schematic of the 3-bit counter:

<img width="810" height="341" alt="image" src="https://github.com/user-attachments/assets/60f8a1fd-d516-4080-85e5-f817848a8e31" />


### 2. Examining the Output Dependency

- **Output Q:** Directly taps `count`.  
  - Thus, `count[1]` and `count[2]` are unused.
- **Binary roll-over:** Since `count` is 3-bits, its values go from 0 to 7 and then repeat.

Count pattern:

$0 \to 1 \to 2 \to 3 \to 4 \to 5 \to 6 \to 7 \to 0 \ldots$

Each clock cycle, `count` toggles:

<img width="1079" height="833" alt="image" src="https://github.com/user-attachments/assets/bc27792c-3601-44a7-8822-1851ad93fd7e" />


### 3. Unused Output Optimization Explanation

- Since only `count` affects the primary output, synthesis tools **remove** logic for `count[1]` and `count[2]`.
- Only essential logic and a single flip-flop for `count` remains.


### 4. Synthesize with Yosys (First Case: Only `count` Used)

**Step-by-step commands:**

1. Start Yosys synthesis.

   ```
   yosys
   ```
   <img width="810" height="578" alt="image" src="https://github.com/user-attachments/assets/bda12185-ca31-4939-b3f3-5f97469b594a" />


2. Read the cell library:
   
   ```
   read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="732" height="88" alt="image" src="https://github.com/user-attachments/assets/6c2ddd27-8ed2-43a6-ac90-70e5a9fafce6" />


3. Read the Verilog design:

   ```
   read_verilog counter_opt.v
   ```
   <img width="672" height="98" alt="image" src="https://github.com/user-attachments/assets/f4f698dc-09e1-4098-ba7c-9f2b89957e22" />


4. Set the top module for synthesis:

   ```
   synth -top counter_opt
   ```
   <img width="618" height="409" alt="image" src="https://github.com/user-attachments/assets/2aec0fea-ed64-4390-ac4e-d80475173782" />


5. Observe inferred flip-flops:

   - Only **one flip-flop** is inferred, matching output dependency.


6. Map DFFs via the library:

   ```
   dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="803" height="88" alt="image" src="https://github.com/user-attachments/assets/a163d1c3-be49-45e0-82bb-19986be27a44" />


7. Final logic optimization and mapping:

   ```
   abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="591" height="148" alt="image" src="https://github.com/user-attachments/assets/b9383665-1916-4f85-8db4-d82b55634d04" />


8. View synthesized circuit:

   ```
   show
   ```
   <img width="1063" height="169" alt="image" src="https://github.com/user-attachments/assets/d1fbf597-907a-4afb-a02e-4258cdbe1ead" />


#### Schematic Details

- Only 1 D flip-flop is present.
- Q toggles on every clock.
- An inverter provides correct reset polarity.

- The logic implements:  
  $Q = \overline{Q}$ with each clock—i.e., toggle behavior.

### 5. Experiment: Modify Code to Use All Bits

- **Modify the code** so that Q is high only when the counter is at value `3'b100` (binary 4).

   ```
   // in counter_opt2.v:
   assign Q = (count [2:0] == 3'b100);
   ```

New dependency: **All bits of `count` now affect the output**.

### 6. Synthesize with Yosys (Second Case: All Bits Used)

**Step-by-step commands:**

1. Copy and edit the file:

   ```
   cp counter_opt.v counter_opt2.v
   ```

2. Edit with vi/gvim (example):

   ```
   !gvim counter_opt2.v
   ```

   <img width="661" height="522" alt="image" src="https://github.com/user-attachments/assets/4cd5b97d-ec90-42d7-9df5-60c2c349fb99" />


3. (In Yosys) Read the library:

   ```
   read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="728" height="81" alt="image" src="https://github.com/user-attachments/assets/bda8b279-87a8-4e4a-ac5d-9b4392931081" />


4. Read the modified Verilog design:

   ```
   read_verilog counter_opt2.v
   ```
   <img width="674" height="116" alt="image" src="https://github.com/user-attachments/assets/d1509f86-7db1-49b9-be3c-b1ebee6be244" />


5. Synthesize with top as before:

   ```
   synth -top counter_opt
   ```
   <img width="609" height="500" alt="image" src="https://github.com/user-attachments/assets/2a4812b8-d417-4182-9875-c0f34b0924e5" />


6. Look for flip-flop count:

   - **Three flip-flops** are inferred, one per bit.

   <img width="391" height="170" alt="image" src="https://github.com/user-attachments/assets/67c82d9e-050d-45de-8388-7f55caeebc0f" />


7. D flip-flop mapping:

   ```
   dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="800" height="93" alt="image" src="https://github.com/user-attachments/assets/3c6620c5-4ecb-4cac-98e2-7e0349740b7f" />


8. ABC technology mapping:

   ```
   abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="601" height="238" alt="image" src="https://github.com/user-attachments/assets/8a7fb179-1af2-4cff-9ee1-24de2505b720" />


9. View resulting schematic:

   ```
   show
   ```
   <img width="1067" height="646" alt="image" src="https://github.com/user-attachments/assets/875e4f38-8a4d-4d0d-991e-acd613faac15" />
   <img width="1058" height="377" alt="image" src="https://github.com/user-attachments/assets/4a83ea69-d7d5-4af0-967e-b06bb357052d" />
<img width="1067" height="302" alt="image" src="https://github.com/user-attachments/assets/369c2d46-2797-4b99-b428-6b53de2b3b32" />


#### Schematic Details

- All three D flip-flops for `count[2]`, `count[1]`, `count` are present.
- Additional logic for incrementing the counter appears.
- Output Q is computed using a combination of gates for the condition

   (Q is high only when $count = 3'b100$).

Gate-level schematic for output logic:


### 7. Analysing Synthesis Optimization

- **First case:** Only dependent signals (`count`) and supporting logic kept; all unused logic is optimized away.
- **Second case:** All signals used, so the synthesizer retains all logic and registers, including adder/increment logic.

Optimization principle:

- **Any intermediate logic not affecting a module’s primary outputs will be completely optimized away by synthesis tools.**

### 8. Summary

- Outputs not propagating to primary outputs are optimized away, as are any upstream logics feeding them.
- For **fully dependent outputs**, all logic and state elements are preserved.
- This optimization improves efficiency and reduces hardware resource usage.

---

Thank you for following this lab on unused output optimization using Yosys!
