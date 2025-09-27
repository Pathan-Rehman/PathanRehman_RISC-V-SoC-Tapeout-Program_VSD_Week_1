## Lab Introduction

This lab explores the behavior of *incomplete if conditions* in Verilog, how such constructs can lead to inference of unintended latches by synthesis tools, and demonstrates the difference between intended multiplexer logic and synthesized latch hardware. The session proceeds through hands-on simulation and synthesis steps using provided Verilog files, observing RTL simulations and synthesis results for code samples with incomplete conditional structures.

## Lab Steps

### 1. Listing and Opening Files

All relevant files for this session are available in the `Verilog files` folder. Begin by listing and opening the files, focusing first on `incomp_if`.

```
# List and open files in the Verilog files directory
ls *in*
```
<img width="814" height="139" alt="image" src="https://github.com/user-attachments/assets/b95c95e8-d317-4330-a5ae-5fe7ca739ebd" />


### 2. Inspecting 'incomp_if' Verilog Code

Open the `incomp_if.v` file. The code features three inputs (`I0`, `I1`, `I2`) and one register output (`Y`). An incomplete `if` condition assigns `Y` only when `I0` is high; no action is specified for the else branch.

```verilog
module incomp_if (input i0 , input i1 , input i2 , output reg y);
always @ (*)
begin
	if(i0)
		y <= i1;
end
endmodule
```
<img width="660" height="249" alt="image" src="https://github.com/user-attachments/assets/03b11ea7-11ac-40d8-afbd-966e8b73204d" />


#### Expected Behavior

- The `always` block acts like a multiplexer with select line `I0`.
- If `I0` is high: `Y` follows `I1`.
- If `I0` is low: `Y` latches its last value (mimicking a D latch).

Block-level summary:
- Latch enable: `I0`
- D input: `I1`
- Q output: `Y`

### 3. Simulating 'incomp_if' Code

Simulate the design using Icarus Verilog and observe the results in GTKWave.

```sh
iverilog incomp_if.v tb_incomp_if.v
./a.out
# This generates tb_incomp_if.vcd for waveform analysis
gtkwave tb_incomp_if.vcd
```

#### Simulation Observations

- When `I0` is low, `Y` holds its value without changing.
- When `I0` transitions from high to low, `Y` latches the current `I1` value.
- When `I0` is high, `Y` mirrors `I1`.

Example waveform:
<img width="1078" height="464" alt="image" src="https://github.com/user-attachments/assets/dcc3fbca-8159-4c95-84fe-3729ab388b22" />

### 4. Synthesizing 'incomp_if' Code

Run synthesis using the `uses` tool. Set the top module accordingly.

```sh
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

- The tool infers a D latch, not a multiplexer.
- Synthesis result shows `I0` as enable, `I1` as D input, `Y` as output Q.

Latching schematic:
<img width="600" height="308" alt="image" src="https://github.com/user-attachments/assets/18090163-f409-4fe5-be81-ddd794d04219" />

### 5. Dangers of Incomplete If: Inferred Latches

- Intending a mux, but a latch is created instead.
- Synthesis and simulation both produce latching behavior due to incomplete conditional assignments.

### 6. Reviewing 'incomp_if2' Code

Open the `incomp_if2.v` file. The conditional structure is nested:

```verilog
module incomp_if2 (input i0 , input i1 , input i2 , input i3, output reg y);
always @ (*)
begin
	if(i0)
		y <= i1;
	else if (i2)
		y <= i3;

end
endmodule
```
<img width="659" height="299" alt="image" src="https://github.com/user-attachments/assets/3c61cc1a-fc54-4f71-a4a2-de37cf3a112c" />


#### Expected Combinational Logic

- Select line 1: `I0`
- Y = `I1` if `I0` is high
- Y = `I3` if `I0` is low and `I2` is high
- No assignment if both `I0` and `I2` are low ⇒ Output latches

Enable condition: `$I0 \vee I2$`

### 7. Simulating 'incomp_if2' Code

```sh
iverilog -o a.out tb_incomp_if2.v incomp_if2.v
./a.out
gtkwave tb_incomp_if2.vcd
```
<img width="808" height="185" alt="image" src="https://github.com/user-attachments/assets/5582a3f0-ec7d-4e78-898f-0967060fad1e" />


#### Waveform Analysis

- `I0` high: `Y` follows `I1`.
- `I0` low, `I2` high: `Y` follows `I3`.
- Both low: `Y` latches its last value.

Sample waveform:
<img width="1075" height="495" alt="image" src="https://github.com/user-attachments/assets/bbdbf2a6-e0fc-413c-acb7-c92cb451388b" />


### 8. Synthesizing 'incomp_if2' Code

```sh
yosys
read_verilog incomp_if2.v
synth -top incomp_if2
# Optionally, ABC post-processing
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="562" height="170" alt="image" src="https://github.com/user-attachments/assets/72539cdf-1f96-45de-8607-959e83579c46" />


#### Synthesis Results

- Combination logic: Output is a function of `I0`, `I1`, `I2`, `I3`.
- Latch-inference: Latch enable is `$I0 \vee I2$`; latching behavior occurs when both are low.

Schematic:
<img width="1061" height="419" alt="image" src="https://github.com/user-attachments/assets/335fafb3-bc3a-4059-98e7-3fc4cf7cf29a" />


### 9. Summary

- Incomplete conditional assignments in Verilog (`if` or `case` without covering all possibilities) typically result in **inferred latches**.
- Intended combinational multiplexers are synthesized as latches if assignments for all logic branches are not present.
- Simulation and synthesis confirm expected latching actions.

---

This concludes the steps for examining incomplete conditional behaviors. The next session will address the effects of incomplete case statements.
