## Summary

This is a comprehensive lab walkthrough for setting up the environment required for VLSI labs and running simulations with IVerilog and GTKWave. It guides users through directory creation, cloning necessary repositories, identifying the structure of the design and test bench files, compiling and simulating Verilog code, and analyzing results with GTKWave, focusing on a multiplexer example.

## Lab Environment Setup

### Tool Flow and Directory Preparation

1. **Create a Working Directory**
   ```bash
   mkdir VLSI
   cd VLSI
   ```
   
   <img width="808" height="143" alt="image" src="https://github.com/user-attachments/assets/5455e100-d39c-42a0-8ee0-bb97e8bf6c71" />


2. **Clone the VSD Flow Repository**
   ```bash
   git clone https://github.com/kunalg123/vsdflow
   ```
   - This creates a `VSDflow` directory containing essential workflow files.
  
   <img width="807" height="422" alt="image" src="https://github.com/user-attachments/assets/7761914d-8008-455c-8269-4c34a492907c" />


3. **Clone the Workshop Repository**
   - Go to the GitHub page for "Sky130 RTL Design and Synthesis Workshop" under kunalg123.
   - Copy the repository link.

   ```bash
   git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop
   ```
   - This creates a `Sky130 RTL Design and Synthesis Workshop` directory for lab exercises.
     
  <img width="853" height="258" alt="image" src="https://github.com/user-attachments/assets/a73122cf-d427-4646-b2c3-295fff87395a" />


### Workshop Directory Structure

- `mylib`: Contains library files needed for synthesis
    - `lib`: Sky130 standard cell library files
    - `verilog_model`: Verilog models of the standard cells
      
    <img width="932" height="494" alt="image" src="https://github.com/user-attachments/assets/6310e307-b387-4818-989a-c64373174172" />


- `Verilog_files`: Contains all lab-related Verilog source and test bench files
  
    <img width="1137" height="891" alt="image" src="https://github.com/user-attachments/assets/58c47b80-7ba5-4c8d-8e3e-8e9088c8cc95" />


## Lab 2: Running IVerilog and GTKWave

### Understanding File Structure

- Each design file (e.g., `good_mux.v`) has a corresponding test bench file (e.g., `tb_good_mux.v`)
- Test bench files are named with `tb_` prefix matching the design's name
- All lab exercises are run inside the `Verilog_files` directory

### Compiling and Simulating a Verilog Design

1. **Navigate to the Verilog Files Directory**
   ```bash
   cd Verilog_files
   ```
   
   <img width="1210" height="155" alt="image" src="https://github.com/user-attachments/assets/a50fb959-8d64-4a98-9308-bcfcf469a285" />


2. **List Design and Test Bench Files**
   ```bash
   ls
   ```
   - Identify the design and its associated test bench (e.g., `good_mux.v` and `tb_good_mux.v`)
     
   <img width="1123" height="872" alt="image" src="https://github.com/user-attachments/assets/3f90db3f-0cda-4be4-8e08-0f168d4167da" />


3. **Compile with IVerilog**
   ```bash
   iverilog good_mux.v tb_good_mux.v
   ```
   - Compiling both the design and test bench creates an executable file named `a.out`.
   

4. **Run the Simulation**
   ```bash
   ./a.out
   ```
   - This generates a Value Change Dump (VCD) file for waveform viewing.
     
   <img width="857" height="285" alt="image" src="https://github.com/user-attachments/assets/7ac77ef1-158f-4cd6-9a44-8b4de3806b26" />


5. **Launch GTKWave to View Waveforms**
   ```bash
   gtkwave tb_good_mux.vcd
   ```
   - The waveform viewer opens, displaying signals from the test bench and design.
     
   <img width="1172" height="865" alt="image" src="https://github.com/user-attachments/assets/bf40cdfa-2bac-4901-8f49-3b11bb730fc7" />


### Analyzing Results in GTKWave

- **Drag and Drop Inputs and Outputs**
  - Move the relevant input and output signals from the test bench and UUT (Unit Under Test) into the waveform panel for analysis.
    
  <img width="774" height="490" alt="image" src="https://github.com/user-attachments/assets/ac3adf44-c546-4c7f-8609-8c9884eab204" />


- **Fit and Zoom Functions**
  - Utilize the "zoom fit" function to view the entire simulation timeline.
  - Use zoom controls to focus on specific waveform regions.
    
  <img width="1055" height="183" alt="image" src="https://github.com/user-attachments/assets/f5c6030a-b2d8-4ddd-a489-c4a11f9eb9fc" />


- **Signal Transition Tracing**
  - Select any signal, and use GTKWave's transition navigation arrows to jump between changes in value, helping trace logic correctness.
  

- **Multiplexer Example Validation**
  - When `select=0`, output follows `I0`.
  - When `select=1`, output follows `I1`.
  

## Design and Test Bench File Overview

### good_mux Design (good_mux.v)
```verilog
module good_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```
- Implements a classic 2:1 multiplexer.

### Test Bench (tb_good_mux.v)
- Instantiates `GoodMux` as UUT (Unit Under Test).
- Generates stimulus:
    - Initial values: `select=0`, `I0=0`, `I1=0`.
    - Simulation runs for 300ns (adjustable).
    - `select` toggles every 75ns.
    - `I0` and `I1` also change at set intervals.
    - VCD dump is configured to capture all transitions.

```verilog
`timescale 1ns / 1ps
module tb_good_mux;
	// Inputs
	reg i0,i1,sel;
	// Outputs
	wire y;

        // Instantiate the Unit Under Test (UUT)
	good_mux uut (
		.sel(sel),
		.i0(i0),
		.i1(i1),
		.y(y)
	);

	initial begin
	$dumpfile("tb_good_mux.vcd");
	$dumpvars(0,tb_good_mux);
	// Initialize Inputs
	sel = 0;
	i0 = 0;
	i1 = 0;
	#300 $finish;
	end

always #75 sel = ~sel;
always #10 i0 = ~i0;
always #55 i1 = ~i1;
endmodule

```

## Additional Tips

- You can adjust simulation duration by changing the `$finish` time in the test bench.
- Modify stimulus sections for varied test scenarios.
- Explore other design files and their test benches in the folder for additional practice.

## Recap

- Set up the environment by creating directories and cloning required Git repositories.
- Understand the structure of design and test bench files.
- Compile and simulate using IVerilog.
- Analyze output waveforms and logic in GTKWave.
- Validate design functionality by tracing stimulus and output responses.
