## Lab Introduction

This lab demonstrates **unused output optimization** during hardware synthesis using Yosys, focusing on a simple 3-bit counter example. You will observe how the synthesis tool eliminates unused signals and optimizes circuits based on actual output dependencies. The lab steps include exploring two variations of Verilog code, synthesizing with Yosys and Sky130 libraries, and analyzing RTL changes and schematic reductions.

## Lab Steps

### 1. Initial Counter Design Overview

- The given Verilog module (`counteropt`) implements a 3-bit up counter with inputs: **clock**, **reset**, and output **Q**.
- Internally, it has a `count` register (`count[2:0]`), incremented every clock cycle unless reset.
- In the initial version, only `count` feeds the module output `Q`, with `count[1]` and `count[2]` unused.

Schematic of the 3-bit counter:

![PLACEHOLDER: schematic]

### 2. Examining the Output Dependency

- **Output Q:** Directly taps `count`.  
  - Thus, `count[1]` and `count[2]` are unused.
- **Binary roll-over:** Since `count` is 3-bits, its values go from 0 to 7 and then repeat.

Count pattern:

$0 \to 1 \to 2 \to 3 \to 4 \to 5 \to 6 \to 7 \to 0 \ldots$

Each clock cycle, `count` toggles:

![PLACEHOLDER: waveform/schematic]

### 3. Unused Output Optimization Explanation

- Since only `count` affects the primary output, synthesis tools **remove** logic for `count[1]` and `count[2]`.
- Only essential logic and a single flip-flop for `count` remains.

Simplified schematic for this case:

![PLACEHOLDER: schematic]

### 4. Synthesize with Yosys (First Case: Only `count` Used)

**Step-by-step commands:**

1. Start Yosys synthesis.

   ```
   yosys
   ```
   ![PLACEHOLDER: terminal-output]

2. Read the cell library:
   
   ```
   read_liberty mylib_lib.sky130
   ```
   ![PLACEHOLDER: terminal-output]

3. Read the Verilog design:

   ```
   read_verilog counter_opt.v
   ```
   ![PLACEHOLDER: terminal-output]

4. Set the top module for synthesis:

   ```
   synth -top counter_opt
   ```
   ![PLACEHOLDER: terminal-output]

5. Observe inferred flip-flops:

   - Only **one flip-flop** is inferred, matching output dependency.

   ![PLACEHOLDER: synthesis-report]

6. Map DFFs via the library:

   ```
   dfflibmap -liberty mylib_lib.sky130
   ```
   ![PLACEHOLDER: terminal-output]

7. Final logic optimization and mapping:

   ```
   abc -liberty mylib_lib.sky130
   ```
   ![PLACEHOLDER: terminal-output]

8. View synthesized circuit:

   ```
   show
   ```
   ![PLACEHOLDER: schematic]

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
   assign Q = (count == 3'b100);
   ```

New dependency: **All bits of `count` now affect the output**.

### 6. Synthesize with Yosys (Second Case: All Bits Used)

**Step-by-step commands:**

1. Copy and edit the file:

   ```
   cp counter_opt.v counter_opt2.v
   ```
   ![PLACEHOLDER: terminal-output]

2. Edit with vi/gvim (example):

   ```
   gvim counter_opt2.v
   ```
   ![PLACEHOLDER: editor-screenshot]

3. (In Yosys) Read the library:

   ```
   read_liberty mylib_lib.sky130
   ```
   ![PLACEHOLDER: terminal-output]

4. Read the modified Verilog design:

   ```
   read_verilog counter_opt2.v
   ```
   ![PLACEHOLDER: terminal-output]

5. Synthesize with top as before:

   ```
   synth -top counter_opt
   ```
   ![PLACEHOLDER: terminal-output]

6. Look for flip-flop count:

   - **Three flip-flops** are inferred, one per bit.

   ![PLACEHOLDER: synthesis-report]

7. D flip-flop mapping:

   ```
   dfflibmap -liberty mylib_lib.sky130
   ```
   ![PLACEHOLDER: terminal-output]

8. ABC technology mapping:

   ```
   abc -liberty mylib_lib.sky130
   ```
   ![PLACEHOLDER: terminal-output]

9. View resulting schematic:

   ```
   show
   ```
   ![PLACEHOLDER: schematic]

#### Schematic Details

- All three D flip-flops for `count[2]`, `count[1]`, `count` are present.
- Additional logic for incrementing the counter appears.
- Output Q is computed using a combination of gates for the condition

   $Q = \overline{count[1]} \\, \&\,\, \overline{count} \\, \&\,\, count[2]$

   (Q is high only when $count = 3'b100$).

Gate-level schematic for output logic:

![PLACEHOLDER: schematic/logic diagram]

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
