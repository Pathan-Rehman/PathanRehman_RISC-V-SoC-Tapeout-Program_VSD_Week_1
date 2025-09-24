## Lab Introduction

This lab introduces basic digital synthesis using Yosys for a 2-input multiplexer in Verilog. The aim is to synthesize a Verilog mux design using the SKY130 standard cell library, review command-line synthesis flow, visualize the logic, and examine the generated gate-level netlist. Steps include invoking Yosys, loading libraries, reading and synthesizing the Verilog design, reviewing the logic schematic, and generating a clean netlist.

## Lab Steps

### 1. Starting Yosys

Open Yosys from the terminal.

```sh
yosys
```


<img width="809" height="612" alt="image" src="https://github.com/user-attachments/assets/4a9a3fff-791d-497e-bd4e-5645b2e7ba03" />


If installed via the VSD flow as instructed in previous sessions, you will see the Yosys prompt.

Exit Yosys to verify your current working directory:

```sh
exit
```


<img width="804" height="167" alt="image" src="https://github.com/user-attachments/assets/d46aa7a5-c12a-4f66-ad33-8dd12890b4ff" />


Your directory should be something like `.../verilog_files/`, containing both the `lib` and `verilog_files` directories (from a cloned repository). All standard cell libraries reside in `lib`.

### 2. Invoking Yosys in the Correct Directory

Re-enter Yosys:

```sh
yosys
```


<img width="809" height="612" alt="image" src="https://github.com/user-attachments/assets/4a9a3fff-791d-497e-bd4e-5645b2e7ba03" />


### 3. Reading the Liberty Library

Read the standard cell library for synthesis. Paths may be absolute or relative; here we use a relative path:

```sh
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```


<img width="804" height="635" alt="image" src="https://github.com/user-attachments/assets/69e06749-db7a-4273-9a91-cf0d7f5ff5ba" />


_Note: The specific `.lib` filename follows a convention, such as `sky130_fd_sc_hd__tt_025C_1v80.lib`._

### 4. Reading the Verilog Source File

Read your Verilog design (in this case: `good_mux.v`):

```sh
read_verilog good_mux.v
```


<img width="763" height="277" alt="image" src="https://github.com/user-attachments/assets/88886d5d-48f8-4f6e-aea8-b323473cd1c7" />


On success, Yosys should report:  
`Successfully finished Verilog frontend.`

If your design consists of multiple files, repeat the `read_verilog` command as needed for each file.

### 5. Setting the Top Module for Synthesis

Specify the module to synthesize using the `synth` flow. In our example, the module name is `good_mux`:

```sh
synth -top good_mux
```


<img width="819" height="694" alt="image" src="https://github.com/user-attachments/assets/65cf461f-3875-4a3a-ad3b-cb6bd6e2a1aa" />


### 6. Synthesizing with Technology Mapping

Run technology mapping to your chosen standard cell library:

```sh
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```


<img width="822" height="701" alt="image" src="https://github.com/user-attachments/assets/37e8d674-a932-4710-8846-18a32aa63182" />


Yosys will print details of inferred cells and IOs, such as:
- Number of input signals: 3
- Number of output signals: 1
- Number of internal signals: 0

It also lists cell usage, such as:
- 1 clock inverter
- 1 NAND gate
- 1 O2AI gate (OR-AND-Invert)
(or)
- 1 mux2_1 (In my case)

Compare these mappings to your original RTL code to ensure expected logic cell usage.

### 7. Visualizing the Logic Schematic

To view the synthesized logic structure:

```sh
show
```


<img width="1208" height="792" alt="image" src="https://github.com/user-attachments/assets/cb3addb1-0007-4cd5-9efa-a6a030fc354f" />


This opens a schematic viewer with the synthesized gate-level representation. Trace nets and cell types to match them to your Verilog logic.

### 8. Boolean Expression for the 2:1 MUX

The synthesized logic implements the following equation for 2-input MUX output $Y$:

$Y = I0 \cdot \overline{SEL} + I1 \cdot SEL$

Where:
- $I0$, $I1$: Data inputs  
- $SEL$: Select input

Boolean simplification steps shown in the lab allow you to map schematic gates to this canonical equation.

### 9. Writing the Gate-Level Netlist

Export the synthesized netlist in Verilog format:

```sh
write_verilog good_mux_netlist.v
```


<img width="453" height="198" alt="image" src="https://github.com/user-attachments/assets/a6ecb707-4dbe-4430-bae8-1ce64915a322" />

<img width="863" height="725" alt="image" src="https://github.com/user-attachments/assets/1a921533-4077-4ebb-aec9-74e5822d2861" />


This initial output may contain attributes and extra info. For a cleaner, attribute-free netlist, use:

```sh
write_verilog -noattr good_mux_netlist.v
```


<img width="525" height="191" alt="image" src="https://github.com/user-attachments/assets/5c37fab9-9630-4502-8c09-cdda2e821975" />

<img width="860" height="561" alt="image" src="https://github.com/user-attachments/assets/7ae6954b-d2c1-4f64-9758-58db543f680f" />


Open and inspect your netlist (example using `gvim`):

```sh
!gvim good_mux_netlist.v
```

#### Netlist Structure

- Module name at top matches what was set in `synth -top`.
- Instantiations for each logic gate (inverter, NAND, O2AI).
- Signals (nets) internally connect gate outputs to subsequent gate inputs.
- Primary inputs and outputs clearly mapped to original Verilog I/O.

Trace through each net in your netlist to verify logic connectivity as seen in the schematic.

---

You have now completed the synthesis workflow for a simple Verilog mux using Yosys and SKY130 standard cells.
