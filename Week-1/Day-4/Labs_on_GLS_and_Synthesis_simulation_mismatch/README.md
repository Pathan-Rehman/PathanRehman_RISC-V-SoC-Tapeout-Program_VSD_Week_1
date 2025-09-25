## Lab Introduction

This lab demonstrates the synthesis and Verilog coding workflow for basic combinational logic simulation and Gate-Level Simulation (GLS) using a 2-to-1 multiplexer as an example. The aim is to show step-by-step procedures for functional simulation (RTL), synthesis, gate-level netlisting, and validation of GLS outputs using waveform visualization. The lab also explores a synthesis vs. simulation mismatch caused by an incorrect sensitivity list in Verilog.

The main steps are:
- Simulate a Verilog-coded mux (RTL).
- Synthesize it, generate its netlist, and perform gate-level simulation (GLS).
- Visualize and interpret waveforms in GTKWave.
- Demonstrate synthesis/simulation mismatch with an intentionally faulty mux.

## Lab Steps

### 1. Verilog Design: Ternary Operator MUX

Write a simple mux using the ternary operator in Verilog:

```verilog
assign Y = select ? I1 : I0;
```
<img width="659" height="173" alt="image" src="https://github.com/user-attachments/assets/ff7eae77-98bd-4146-9705-2f21913c9c19" />


### 2. RTL Simulation

Invoke `iverilog` to run functional simulation with your testbench.

```bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v -o a.out
```
<img width="813" height="122" alt="image" src="https://github.com/user-attachments/assets/9cc40929-4d31-4637-b328-71b1ebad81b5" />


Generate the waveform by running the simulation binary:

```bash
./a.out
```
<img width="817" height="94" alt="image" src="https://github.com/user-attachments/assets/b485dcad-73af-47ba-b898-d459ea05c79a" />


Visualize the waveform using GTKWave:

```bash
gtkwave tb_ternary_operator_mux.vcd
```
<img width="1077" height="465" alt="image" src="https://github.com/user-attachments/assets/a6e71c6f-b120-4b64-b38a-60f23365ab9a" />


**What to expect:**  
Signals like `I0`, `I1`, `select`, and `Y` will be visible.  
When `select` is low, `Y` follows `I0`; when `select` is high, `Y` follows `I1`.

### 3. Synthesis

Use `yosys` (or similar tool) to synthesize the RTL design:

```bash
yosys
```
Inside yosys, run these commands:

```bash
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_mux_net.v
```
<img width="454" height="462" alt="image" src="https://github.com/user-attachments/assets/ef2219c0-a4be-4c4f-b99a-54124450ae58" />


Inspect schematic/netlist:

```bash
show
```
<img width="599" height="358" alt="image" src="https://github.com/user-attachments/assets/e5be7258-560d-4b03-a36f-7106f4e3a1ab" />


**Netlist Boolean Equation:**  
You should trace that the synthesizer inferred gates correctly (NAND, inverter, OR-AND-Invert, etc.)

The mux function is mathematically:
$Y = \overline{I1 \cdot select} + \overline{select + \overline{I0}}$
Applying De Morgan’s theorem and simplifying:
$Y = (\overline{select} \cdot I0) + (select \cdot I1)$
This is the classic 2-to-1 mux equation.

### 4. Gate-Level Simulation (GLS)

Run gate-level simulation using iverilog:

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v -o a.out
```
<img width="812" height="190" alt="image" src="https://github.com/user-attachments/assets/08f3e243-a76f-4786-859a-b660944ca874" />


Generate waveform:

```bash
./a.out
```
<img width="809" height="94" alt="image" src="https://github.com/user-attachments/assets/7efc16e4-c883-4a0e-aea0-179895832c59" />


View the GLS waveform in GTKWave:

```bash
gtkwave tb_ternary_operator_mux.vcd
```
<img width="1078" height="469" alt="image" src="https://github.com/user-attachments/assets/ab3f7ec3-08e0-48ae-9e2d-142244b5e9ae" />


**Identifying GLS:**  
In the GTKWave window, you’ll notice signals with suffixes like `_4` indicating gate-level modules versus pure RTL simulation.
<img width="273" height="446" alt="image" src="https://github.com/user-attachments/assets/07c72742-e185-45e9-b7e9-7992fa58dfb5" />


The behavior should match the mux—`Y` following `I0` or `I1` based on `select`.

### 5. Synthesis/Simulation Mismatch Example (Bad MUX)

A purposely faulty mux Verilog is used to demonstrate mismatches due to a wrong sensitivity list:

```verilog
always @(select)
  if(select)
    Y = I1;
  else
    Y = I0;
```
<img width="820" height="286" alt="image" src="https://github.com/user-attachments/assets/a64d7cf6-6229-4d93-a250-7a71a19c833c" />


#### Functional Simulation

```bash
iverilog bad_mux.v tb_bad_mux.v -o a.out
```

Run and view waveform:

```bash
./a.out
gtkwave tb_bad_mux.vcd
```
<img width="1072" height="464" alt="image" src="https://github.com/user-attachments/assets/06e71c75-a14c-4d56-8a47-eb6012758747" />


**Observation:**  
In RTL simulation, output `Y` changes only when `select` changes—not when `I0` or `I1` changes. This looks like “flop-like” behavior but is NOT a proper mux.

Take a screenshot of this waveform for comparison.

#### Synthesis

```bash
yosys
```
Inside yosys:

```bash
read_verilog bad_mux.v
synth -top bad_mux
write_verilog -noattr bad_mux_net.v
```
<img width="595" height="362" alt="image" src="https://github.com/user-attachments/assets/6a9842a7-7aef-42aa-9485-d3c4ab531438" />
<img width="661" height="345" alt="image" src="https://github.com/user-attachments/assets/e2f93e78-4ec4-4b37-a339-ec66b8207b25" />


#### GLS of Faulty MUX

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v -o a.out
./a.out
gtkwave tb_bad_mux.vcd
```
<img width="1075" height="480" alt="image" src="https://github.com/user-attachments/assets/206a89a3-5d19-4b25-8273-70a42788b593" />


**Observation:**  
GLS output correctly follows mux behavior—`Y` is responsive to changes in `I0` and `I1` when `select` is low or high.

Compare this with the earlier RTL simulation to visualize the mismatch.

**Key learning:**  
Synthesis tools infer proper mux structures regardless of poor sensitivity list coding in Verilog simulation, causing a mismatch between simulated functional behavior and synthesized hardware.

---

End of lab.
