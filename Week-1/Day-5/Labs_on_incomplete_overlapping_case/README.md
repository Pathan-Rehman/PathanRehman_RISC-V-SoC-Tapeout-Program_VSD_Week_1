## Lab Introduction

This lab aims to demonstrate the impact of incomplete, complete, partial, and overlapping case statements in Verilog coding on RTL design and synthesis. You will analyze how coding style influences inferred latches, functional simulation, and synthesizer outputs. The steps include running simulations, viewing waveforms, synthesizing RTL, inspecting nets/cells, and comparing Verilog coding for different case statement types.

## Lab Steps

### 1. Prepare Required Files

- Open the four Verilog files needed for this session:  
  - `incomp_case.v`
  - `comp_case.v`
  - `partial_case_assign.v`
  - `bad_case.v`

<img width="813" height="384" alt="image" src="https://github.com/user-attachments/assets/755f7a0f-ebf5-482b-a169-76a3c832e6a4" />


---

### 2. Incomplete Case Statement

#### System Setup    
Inputs:  
- **I0**, **I1**, **I2**  
Select Line: 2 bits (`select[1]`, `select`)  
Output: **Y**  
    
#### Case Statement Logic    
- If `select` = `00`, output Y = I0  
- If `select` = `01`, output Y = I1  
- `select` = `10` or `11` : No assignment, no `default`. Leads to inferred latch.

#### Truth Table

| select[1] | select | Y         |
|:----------|:----------|:----------|
| 0         | 0         | I0        |
| 0         | 1         | I1        |
| 1         | 0         | Latch     |
| 1         | 1         | Latch     |

![PLACEHOLDER: schematic]

#### Functional Simulation

Run the simulation:
```bash
iverilog incomp_case.v tb_incomp_case.v -o a.out
./a.out
gtkwave tb_incomp_case.vcd
```
<img width="1079" height="502" alt="image" src="https://github.com/user-attachments/assets/4084629c-4a5f-4604-b322-5795c264d1fb" />


- Observe that Y follows I0 when `select` is `00`, I1 when `select` is `01`, and is latched for `10` and `11`.

---

#### RTL Synthesis

Run synthesis:
```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
<img width="1068" height="253" alt="image" src="https://github.com/user-attachments/assets/9d8898b0-ae99-4607-acb9-79cba9cf5958" />


- Inspect the synthesized design to see an inferred latch driving Y.
- Enable condition for latch: `!select[1]`

---

### 3. Complete Case Statement

#### Modified Code  
Add a `default` branch to the case statement:
- For `select = 00`: Y = I0
- For `select = 01`: Y = I1
- For all other values (`default`): Y = I2

#### Simulation

```bash
iverilog comp_case.v tb_comp_case.v -o a.out
./a.out
gtkwave tb_comp_case.vcd
```
<img width="1071" height="492" alt="image" src="https://github.com/user-attachments/assets/419c0402-5fc0-4ade-80e0-f47814fca456" />


- When `select` is `10` or `11`, Y properly follows I2—no latching.

---

#### RTL Synthesis

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog comp_case.v
synth -top comp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="1070" height="254" alt="image" src="https://github.com/user-attachments/assets/2f8e8a67-7938-4ffb-9733-8e865fe2a311" />


- No inferred latches; purely combinational gates and mux logic, as expected.

---

### 4. Partial Case Assignment

#### Code Notes

Two outputs: Y and X.

- **Y output:**  
  - select `00`: I0  
  - select `01`: I1  
  - select `10`, `11`, `default`: I2  
  - No inferred latch.

- **X output:**  
  - select `00`: I0  
  - select `10` and `11`: I1 (via default)  
  - select `01`: not assigned, so latch inferred.

#### Table for X

| select[1] | select | X         |
|:----------|:----------|:----------|
| 0         | 0         | I0        |
| 0         | 1         | Latch     |
| 1         | 0         | I1        |
| 1         | 1         | I1        |

#### Latch Enable Condition for X

- Enable when `select[1] == 1` or `select == 0`
- Boolean simplification: $select[1] + \overline{select}$

---

#### Synthesis and Simulation

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog partial_case_assign.v
synth -top partial_case_assign
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```
<img width="1068" height="354" alt="image" src="https://github.com/user-attachments/assets/f251fe98-b43f-4783-94cc-0b808014a96d" />


- Only X has an inferred latch; Y remains purely combinational.

---

### 5. Overlapping Case Arms (bad case)

#### Verilog for Overlapping Case

Example:
- select `00`: Y = I0
- select `01`: Y = I1
- select `10`: Y = I2
- select `1?`: Y = I3 (matches both `10` and `11`)

#### Simulation

```bash
iverilog bad_case.v tb_bad_case.v -o a.out
./a.out
gtkwave tb_bad_case.vcd
```
<img width="1074" height="528" alt="image" src="https://github.com/user-attachments/assets/efa6cb00-fc23-49f8-901f-4e8c548a7ee4" />


- When select is `11`, simulator gets confused due to overlapping matches (both `10` and `1?`).

---

#### Synthesis and Netlist Simulation

```bash
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_case.v
synth -top bad_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_case_net.v
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_case_net.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```
<img width="270" height="296" alt="image" src="https://github.com/user-attachments/assets/05c5963f-0640-4c8f-9ea8-376e8510f1f6" />
<img width="1072" height="701" alt="image" src="https://github.com/user-attachments/assets/075868a8-0f17-4be0-b04e-a4223fec7595" />


- In netlist simulation, no latch is inferred, but routing of Y for `11` becomes consistent (selects I3)—unlike the mismatched RTL behavior.

---

### 6. Additional Coding Guidelines

- **Avoid incomplete or partial assignments** in your Verilog case statements to prevent inferred latches and unwanted synthesis behaviors.
- **Ensure all case labels are mutually exclusive**; do not overlap arms/tag conditions.
- Always validate resulting Boolean logic functions by simplifying expressions and mapping them to expected functional descriptions.
![PLACEHOLDER: screenshot/terminal-output/schematic]

---

### 7. For Further Study

- Practice simplifying Boolean expressions like the latch enable:
  - Example: $select[1] + \overline{select}$
- Use Verilog cell models to explicitly map the synthesized logic to your written RTL.
