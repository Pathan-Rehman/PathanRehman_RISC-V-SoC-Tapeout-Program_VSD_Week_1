## Lab Introduction

This lab demonstrates logic optimization using Yosys on various Verilog modules (opt_check, opt_check2, opt_check3, etc.) following Boolean simplification steps. The aim is to observe how different multiplexer-based logic expressions are optimized to basic gates, and to analyze the resulting schematic after each optimization pass.

You will:
- Open and inspect a set of Verilog files (opt_check, opt_check2, opt_check3, etc.).
- Synthesize and optimize each design using Yosys.
- Examine the effect of `opt_clean -purge` and verify schematic simplification.
- Compare the logical equations before and after optimization.
- Try optimization on additional modules as directed.

---

## Lab Steps

### 1. Prepare the Lab Environment

- Ensure you are in the folder containing all relevant Verilog files (e.g., `opt_check.v`, `opt_check2.v`, `opt_check3.v`, etc.).
- Open the files to confirm their availability.

<img width="823" height="138" alt="image" src="https://github.com/user-attachments/assets/e0bda727-6b53-4159-8f59-45b7bbac8931" />


---

### 2. opt_check: AND Gate Optimization

**Examining opt_check logic**:  
- $Y$ is assigned based on $A$:
    - If $A = 1$, $Y = B$
    - If $A = 0$, $Y = 0$
- Equivalent to: $Y = \overline{A} \cdot 0 + A \cdot B$, which simplifies to $Y = AB$ (a 2-input AND gate).

  <img width="661" height="198" alt="image" src="https://github.com/user-attachments/assets/ce7d0ba2-71a2-4042-a668-f3e65bbb38ad" />


**Yosys Workflow:**

```sh
yosys
```
<img width="813" height="579" alt="image" src="https://github.com/user-attachments/assets/305d3cd3-9298-446f-aa6c-5d91983855a3" />


```sh
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="747" height="98" alt="image" src="https://github.com/user-attachments/assets/4d3898e2-b354-4074-9eb7-ee7d30dd0ebb" />


```sh
read_verilog opt_check.v
```
<img width="658" height="111" alt="image" src="https://github.com/user-attachments/assets/8857582a-610c-426f-a8a0-3d886d8daab3" />


```sh
synth -top opt_check
```
<img width="605" height="392" alt="image" src="https://github.com/user-attachments/assets/432709d6-84d8-4a89-8ecb-a1a13e3565e8" />


```sh
opt_clean -purge
```
<img width="616" height="104" alt="image" src="https://github.com/user-attachments/assets/e31ac7a4-79bb-498e-83cf-a3d96018e2e7" />


```sh
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="595" height="149" alt="image" src="https://github.com/user-attachments/assets/347c37f9-6896-4ba6-a9cf-593fb4e5aa9e" />


**Verify the schematic:**  
Expecting to see a 2-input AND gate.

<img width="598" height="164" alt="image" src="https://github.com/user-attachments/assets/1c029f30-ec60-46bc-9c24-f957808de4ab" />


---

### 3. opt_check2: OR Gate Optimization

**Examining opt_check2 logic**:  
- Depending on $A$:
    - If $A = 1$, $Y = 1$
    - If $A = 0$, $Y = B$
- Logic: $Y = \overline{A}B + A$, which simplifies to $Y = A + B$ (a 2-input OR gate).

  <img width="662" height="178" alt="image" src="https://github.com/user-attachments/assets/e071b25f-b5a1-4e65-a405-ab1caabc2658" />


**Yosys Workflow:**

```sh
read_verilog opt_check2.v
```
<img width="697" height="128" alt="image" src="https://github.com/user-attachments/assets/a54a5f52-9dc4-4a51-a487-1227b61f6e1c" />


```sh
synth -top opt_check2
```
<img width="613" height="391" alt="image" src="https://github.com/user-attachments/assets/24b3bffd-032f-413d-9dcc-ddb3016fe151" />


```sh
opt_clean -purge
```
<img width="622" height="114" alt="image" src="https://github.com/user-attachments/assets/5c09c7b0-ae4d-4ae3-a8b4-d0afd6a9d699" />


```sh
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="591" height="152" alt="image" src="https://github.com/user-attachments/assets/9a4a22cd-5674-44a1-83b0-ff3dc10d39bd" />


**Note:** The schematic may show a NAND-inverters structure due to implementation preferences, but logically it is equivalent to an OR gate using De Morgan’s theorem:  
$Y = (A\cdot B)^{\prime} = \overline{A} + \overline{B}$.

<img width="1203" height="448" alt="image" src="https://github.com/user-attachments/assets/2837c4e1-13b4-495f-a953-01cdfd857366" />


---

### 4. opt_check3: 3-Input AND Gate Optimization

**Examining opt_check3 logic**:
- $Y$ is determined by values of $A$ and $C$:
    - If $A = 0$: $Y = 0$
    - If $A = 1$ and $C = 1$: $Y = B$
    - Else: $Y = 0$
- Expression simplifies to: $Y = ABC$ (3-input AND gate).

**Yosys Workflow:**

```sh
read_verilog opt_check3.v
```
<img width="653" height="158" alt="image" src="https://github.com/user-attachments/assets/93d72ce6-3098-4d45-baad-8c7d9f1d0278" />


```sh
synth -top opt_check3
```
<img width="598" height="415" alt="image" src="https://github.com/user-attachments/assets/985e80fc-53fb-43b5-8d0f-f39cfda28ef7" />


```sh
opt_clean -purge
```
<img width="624" height="110" alt="image" src="https://github.com/user-attachments/assets/a2e703bb-cbfa-43d9-9d40-858f1c2a3a02" />


```sh
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="572" height="155" alt="image" src="https://github.com/user-attachments/assets/adb915a6-81ba-47b6-8d2d-f6119490b208" />


**Verify the schematic:**  
Expecting a 3-input AND gate.

<img width="1206" height="536" alt="image" src="https://github.com/user-attachments/assets/7353d050-e2d0-44f0-9852-ff76565f7dbb" />


---

### 5. Further Modules: Student Exercises

- For `opt_check4` and modules with multiple submodules:
    - **Tip:** Use `flatten` before running `opt_clean -purge` to ensure proper optimization across multiple modules.

**Sample Yosys commands:**

```sh
read_verilog opt_check4.v
flatten
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="814" height="217" alt="image" src="https://github.com/user-attachments/assets/b6dbd154-b5ea-4954-9ec1-e4d4339f1c52" />
<img width="599" height="262" alt="image" src="https://github.com/user-attachments/assets/3cd6747c-098f-44c7-a9a9-ee67393a39f8" />


- Write out the netlist and analyze the schematic to verify optimization results.

**Explore:**  
- Try these steps for `opt_check4` and any modules you wish.  
- Observe the schematic to understand how logic is minimized.

---

**End of Lab**
