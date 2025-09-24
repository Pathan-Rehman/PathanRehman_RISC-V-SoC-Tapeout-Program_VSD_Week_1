## Lab Introduction

This lab investigates the difference between **hierarchical and flat synthesis** using the `synth -top` switch in Yosys. Using a simple Verilog file (`multiple_modules.v`) consisting of two submodules (an AND gate and an OR gate) and a top module that instantiates them, you will:
- Read and examine the hierarchical structure of the Verilog design.
- Synthesize the design hierarchy and observe the resulting netlist.
- Explore how synthesis tools implement logic (e.g., via De Morgan's theorem).
- Flatten the hierarchy and compare with the hierarchical netlist.
- Synthesize submodules independently using `synth -top`.
- Discuss scenarios where submodule-level synthesis is advantageous.

## Lab Steps

### 1. Open and Review the Design File

- Open `multiple_modules.v` in your editor.

  - **Contents:**
    - `sub_module1`: AND gate.
    - `sub_module2`: OR gate.
    - `multiple_module`: Instantiates `sub_module1` (U1) and `sub_module2` (U2).
    - Final design: Inputs `A, B, C`, output `Y`. Output of U1 feeds into U2; `C` also connects to U2. Output of U2 is `Y`.
      

  <img width="660" height="433" alt="image" src="https://github.com/user-attachments/assets/8d8ceda2-5ef2-4c12-8781-5459519d2b5f" />


### 2. Launch Yosys and Read the Library

```sh
yosys
```


<img width="816" height="583" alt="image" src="https://github.com/user-attachments/assets/da772bbb-2847-4b67-87e0-10154e9f7f37" />


```sh
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```


<img width="812" height="579" alt="image" src="https://github.com/user-attachments/assets/6854eaa7-b601-4906-bd8c-df357d6f1748" />


### 3. Read the Verilog Design

```sh
read_verilog multiple_modules.v
```


<img width="818" height="581" alt="image" src="https://github.com/user-attachments/assets/9bcaa461-84d1-4e06-bff4-2493dfe6b3d5" />


### 4. Synthesize With Top Module

```sh
synth -top multiple_modules
```


<img width="401" height="595" alt="image" src="https://github.com/user-attachments/assets/c1d1a48d-25e8-45af-8b6d-ab14e4d9a1d7" />

<img width="610" height="767" alt="image" src="https://github.com/user-attachments/assets/db2d5bb9-8181-4739-bb36-09c4b577cf7c" />


- Yosys reports the hierarchical structure:
  - `sub_module1`: 1 AND gate
  - `sub_module2`: 1 OR gate
  - `multiple_module`: instantiates both


### 5. Link the Design to the Library

```sh
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```


<img width="851" height="852" alt="image" src="https://github.com/user-attachments/assets/17dfd88b-a36d-405e-b993-c015fee379dd" />


### 6. Visualize the Hierarchy

- Show the top module.

```sh
show multiple_modules
```


<img width="824" height="217" alt="image" src="https://github.com/user-attachments/assets/8e943827-a436-402f-933f-71654a35f67d" />


- The diagram displays **U1** (submodule1 instance) and **U2** (submodule2 instance), maintaining hierarchy rather than showing primitive gates.

### 7. Write Hierarchical Netlist

```sh
write_verilog multiple_modules_hier.v
```


<img width="506" height="279" alt="image" src="https://github.com/user-attachments/assets/e762f3d7-7297-4842-badd-4b3f025890aa" />

<img width="722" height="763" alt="image" src="https://github.com/user-attachments/assets/d07aef30-246a-4448-8106-6e2ae7d660a9" />


- **Note:** To suppress attributes, use `-noattr`:

```sh
write_verilog -noattr multiple_modules_hier.v
```


<img width="540" height="237" alt="image" src="https://github.com/user-attachments/assets/18f238ff-b601-49ad-8e1d-0f965ecf4104" />


- Hierarchies are preserved in the netlist. Each submodule is present, and their instantiations are visible.


<img width="305" height="713" alt="image" src="https://github.com/user-attachments/assets/a975d3d7-b807-466b-a281-4aaf770b584b" />

<img width="284" height="336" alt="image" src="https://github.com/user-attachments/assets/750af537-4c3a-43b1-90b3-db4d84b63769" />


### 8. Analyze Gate-Level Implementation (De Morgan's Theorem)

- The OR gate is implemented using NAND and inverters:

  - Expected: OR gate ($A + B$)
  - Actual (after synthesis): Two inverters and a NAND gate, forming a functionally equivalent OR by De Morgan's law: $A + B = \overline{\overline{A} \cdot \overline{B}}$

- *Key equation:*
  - De Morgan’s: $\overline{A \cdot B} = \overline{A} + \overline{B}$
  - Synthesis uses NAND and inverters due to CMOS constraints and performance.

  *CMOS: Stacked PMOS is avoided due to poor mobility; synthesis prefers inverting functions, e.g., NAND with inverters.*

### 9. Flatten the Design

- Write out a flat netlist (hierarchies removed):

```sh
flatten
write_verilog -noattr multiple_modules_flat.v
```


<img width="474" height="197" alt="image" src="https://github.com/user-attachments/assets/473f32a9-7c2e-419e-88a7-f9ade14ea35a" />

<img width="550" height="231" alt="image" src="https://github.com/user-attachments/assets/2dab214d-8a9a-4c20-90aa-b03dee39d6c8" />


- The netlist now contains only one module (`multiple_module`), with direct instantiations of AND and OR gates—no `U1`/`U2` hierarchies.

```sh
/* Generated by Yosys 0.33 (git sha1 2584903a060) */

module multiple_modules(a, b, c, y);
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  wire _4_;
  wire _5_;
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  wire net1;
  wire \u1.a ;
  wire \u1.b ;
  wire \u1.y ;
  wire \u2.a ;
  wire \u2.b ;
  wire \u2.y ;
  output y;
  wire y;
  sky130_fd_sc_hd__and2_0 _6_ (
    .A(_1_),
    .B(_0_),
    .X(_2_)
  );
  sky130_fd_sc_hd__or2_0 _7_ (
    .A(_4_),
    .B(_3_),
    .X(_5_)
  );
  assign _4_ = \u2.b ;
  assign _3_ = \u2.a ;
  assign \u2.y  = _5_;
  assign \u2.a  = net1;
  assign \u2.b  = c;
  assign y = \u2.y ;
  assign _1_ = \u1.b ;
  assign _0_ = \u1.a ;
  assign \u1.y  = _2_;
  assign \u1.a  = a;
  assign \u1.b  = b;
  assign net1 = \u1.y ;
endmodule
```


### 10. Compare Hierarchical vs. Flat Netlist

- Open both hierarchical and flat netlists side-by-side for reference.
- Hierarchical: Preserves submodule declarations and instantiations.
- Flat: All logic is inlined into the top module; no submodules appear.

<img width="1209" height="873" alt="image" src="https://github.com/user-attachments/assets/bcc17827-accb-44a5-8601-56e506f0d76d" />


### 11. Visualize Flattened Design

```sh
show multiple_modules
```


<img width="1214" height="118" alt="image" src="https://github.com/user-attachments/assets/25b87ee7-a8f3-4201-b768-15a3d49bb12c" />


- U1 and U2 instances are gone; AND/OR structures are directly visible.

### 12. Synthesize at Submodule Level

- For submodule-only synthesis:

```sh
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top sub_module1
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```


<img width="877" height="866" alt="image" src="https://github.com/user-attachments/assets/14ccc6aa-12da-4306-9d7e-0c7fc5f5a0e3" />


- Only the AND gate of `sub_module1` is shown; `sub_module2` and hierarchy are absent.

### 13. Use-Case Discussion for Submodule Synthesis

- **Why perform submodule synthesis?**
  - If your design has **multiple instances of the same component** (e.g., multipliers `mult1`, `mult2`, ...), synthesize the component once and replicate to save time and tool effort.
  - For **large or complex designs**, synthesize subsystems independently to optimize each, then assemble at the top level (“divide and conquer” approach).


---
**Reminder:**  
To control which module is synthesized, always use:

```sh
synth -top <module_name>
```
