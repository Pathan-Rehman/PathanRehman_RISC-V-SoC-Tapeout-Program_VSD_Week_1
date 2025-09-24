## Lab Introduction

This lab focuses on simulating and synthesizing various D flip-flop (DFF) designs in Verilog, examining their waveform behavior, and understanding how synthesis tools optimize their implementation. You will:
- Simulate asynchronous and synchronous reset/set DFFs using **Icarus Verilog (iverilog)** and waveform viewers.
- Analyze the simulation waveforms to confirm correct behavior.
- Synthesize the DFF designs with **Yosys**, observing how the tool maps Verilog RTL to gates or simple rewiring.
- Explore specific multiplication-by-powers-of-two optimizations at the RTL and gate level, confirming that synthesis can entirely eliminate hardware for certain operations.

---

## Lab Steps

### 1. Simulating D Flip-Flops with Icarus Verilog

- Prepare the relevant Verilog source and associated testbench files in your working directory:
  - `dff_asyncres_syncres.v`
  - `dff_asyncres.v`
  - `dff_async_set.v`
  - Testbenches: `tb_dff_asyncres_syncres.v`, `tb_dff_asyncres.v`, `tb_dff_async_set.v`
 
  <img width="806" height="207" alt="image" src="https://github.com/user-attachments/assets/fb459e49-f353-4806-942a-3df72100533d" />


#### a. Simulate Asynchronous Reset DFF

1. Compile with iverilog:
   ```bash
   iverilog dff_asyncres.v tb_dff_asyncres.v
   ```


   <img width="810" height="56" alt="image" src="https://github.com/user-attachments/assets/19b3005c-6d76-49bc-920d-470fa692e118" />


1. Run the simulation to generate a VCD waveform:
   ```bash
   ./a.out
   ```

   <img width="816" height="94" alt="image" src="https://github.com/user-attachments/assets/564936ff-e152-4806-940d-120b4b4dac23" />


2. View the resulting waveform with GTKWave:
   ```bash
   gtkwave tb_dff_asyncres.vcd
   ```

   
   <img width="1214" height="880" alt="image" src="https://github.com/user-attachments/assets/aa698e96-4329-47cf-8412-58347cd57798" />


   - Inspect how the output `Q` responds **immediately** to `reset` going low, independent of the clock edge.
   - Confirm that DFF output only updates to input `D` at the clock's positive edge, but `reset` forces `Q` low asynchronously.

#### b. Simulate Asynchronous Set DFF

1. Compile and run:
   ```bash
   iverilog dff_async_set.v tb_dff_async_set.v
   ```
   
   <img width="812" height="50" alt="image" src="https://github.com/user-attachments/assets/04b72804-9083-4c98-8f6c-8bd5361dc5c0" />

   ```bash
   ./a.out
   ```
   
   <img width="822" height="90" alt="image" src="https://github.com/user-attachments/assets/af4bf9db-6375-418a-a5a9-4f5b979fcdf1" />

   ```bash
   gtkwave tb_dff_async_set.vcd
   ```
   
  <img width="1212" height="875" alt="image" src="https://github.com/user-attachments/assets/598950f4-5cfc-4812-b316-6bc157791fc5" />


#### c. Simulate Synchronous Reset DFF

1. Compile and run:
   ```bash
   iverilog dff_syncres.v tb_dff_syncres.v
   ```
   
   <img width="811" height="54" alt="image" src="https://github.com/user-attachments/assets/a41dad97-5a71-4207-9f81-449a62b7a695" />

   ```bash
   ./a.out
   ```
   
   <img width="819" height="92" alt="image" src="https://github.com/user-attachments/assets/a14c5c99-9bb4-48b1-a569-abd11afa49f4" />

   ```bash
   gtkwave tb_dff_syncres.vcd
   ```
   
   <img width="1212" height="880" alt="image" src="https://github.com/user-attachments/assets/0c3a531c-315e-4c51-9b96-adc1e9183791" />                             

   - Confirm that `Q` only responds to `reset` **on the next clock edge**, not immediately.

---

### 2. Synthesis with Yosys

#### a. Synthesize DFF with Asynchronous Reset

1. Start Yosys:
   ```bash
   yosys
   ```
   <img width="816" height="525" alt="image" src="https://github.com/user-attachments/assets/33e02dba-7178-4e97-8b26-ae0789937f24" />


2. Read the library and design files:
   ```tcl
   read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   read_verilog dff_asyncres.v
   synth -top dff_asyncres
   dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="759" height="236" alt="image" src="https://github.com/user-attachments/assets/23212e7d-64b6-45e0-8d70-7a8a2108250e" />
<img width="616" height="392" alt="image" src="https://github.com/user-attachments/assets/3e89880e-297f-4983-8333-cfb209233f19" />
<img width="806" height="89" alt="image" src="https://github.com/user-attachments/assets/2047c608-d2fd-40c3-aaf0-f3f5f61361e6" />


3. Synthesize and map:
   ```tcl
   abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   show
   ```
   <img width="500" height="153" alt="image" src="https://github.com/user-attachments/assets/736e045f-d151-484e-877b-5c910fdafe5b" />

   <img width="1201" height="285" alt="image" src="https://github.com/user-attachments/assets/69af1c36-3b1f-45ea-b14a-c487f191a35b" />



   - Note that the synthesized flop includes an inverter if your RTL and library use different reset polarities.

#### b. Synthesize DFF with Asynchronous Set

1. Read design:
   ```tcl
   read_verilog dff_async_set.v
   synth -top dff_async_set
   dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
   <img width="805" height="87" alt="image" src="https://github.com/user-attachments/assets/b063d925-7aa3-4a5c-8661-7a75497c7a8d" />


2. Map and view:
   ```tcl
   abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   show
   ```
   <img width="583" height="149" alt="image" src="https://github.com/user-attachments/assets/f6fa74ba-c4b6-495c-8f1b-899718061131" />

   <img width="1204" height="275" alt="image" src="https://github.com/user-attachments/assets/eda342b7-914f-47b9-b5e5-a854857cb3d7" />


   - An inverter may be present for polarity adjustment.

#### c. Synthesize DFF with Synchronous Reset

1. Run synthesis:
   ```tcl
   read_verilog dff_syncres.v
   synth -top dff_syncres
   dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   show
   ```
   <img width="592" height="151" alt="image" src="https://github.com/user-attachments/assets/49b10664-7c7f-4d4d-a680-138bab9bfc56" />
   
<img width="1204" height="280" alt="image" src="https://github.com/user-attachments/assets/b38cda54-1350-4338-ad25-98fb41180cf5" />


   - The synthesized circuit replaces the synchronous reset with logic before the D input, as expected for synchronous logic.

- Examine the schematic for AND gates and absence of direct reset/set wires, confirming only synchronous behavior.

---

### 3. Boolean Expression Analysis from Schematic

- For synchronous reset, analyze the apparent logic:
  - The schematic includes a 2x1 mux before the DFF.
  - Output equation:
    - For a 2x1 mux with select line $S$:
      - $Y = S . I_1 + \overline{S} . I_0$
    - In this design: 
      - Select sync_reset
      - $I_1 = 0$, $I_0 = D$
      - $Y = \text{sync-reset} \cdot 0 + \overline{\text{sync-reset}} \cdot D = \overline{\text{sync-reset}} \cdot D$
  - This matches the intended synchronous reset behavior.

---

### 4. Synthesis Optimizations for Multiplication by Constants

#### a. Examine Multiplication by 2 (`mult2.v`)

- Verilog code: 3-bit input $A$, 4-bit output $Y$, $Y = 2 \times A$.
<img width="387" height="61" alt="image" src="https://github.com/user-attachments/assets/a3687973-36c0-48db-b924-fe4128a56533" />


- Synthesize:
  ```tcl
  yosys
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog mult_2.v
  synth -top mult2
  ```
  <img width="606" height="365" alt="image" src="https://github.com/user-attachments/assets/891dc9cd-5b29-4b9a-97db-eac4e8fa66db" />
<img width="809" height="200" alt="image" src="https://github.com/user-attachments/assets/ab71f6dc-df37-4c8f-894c-884a0e8687d3" />
<img width="607" height="210" alt="image" src="https://github.com/user-attachments/assets/40149609-2303-4e16-ab21-6bf54ace389e" />


- Observe the synthesis log:
  - No standard cells inferred (hardware count is zero).
  - This is because appending a zero (left shift) implements $Y = 2 \times A$ without gates.

- To write out the synthesized netlist:
  ```tcl
  write_verilog -noattr mul2_net.v
  ```
  <img width="654" height="268" alt="image" src="https://github.com/user-attachments/assets/3c75ee8a-8a0d-4305-ae37-a1c94440b33d" />


  - Inspect that $Y$ is just $a$ with one zero appended (rewired).

#### b. Multiplication by 9 (`mult8.v` with $Y = 9 \times A$)

- Verilog code: 3-bit input $A$, 6-bit output $Y$, $Y = 9 \times A$

<img width="382" height="64" alt="image" src="https://github.com/user-attachments/assets/5a896c99-15ec-4fcb-9e16-df28d3c06fb9" />

- Interpretation: $Y = (A << 3) + A$ (i.e., $Y = 8 \times A + A$)
- After synthesis:
  - Again, no standard cells inferred.
  - The output $Y$ is just $AA$ (input A concatenated with itself).
  - This is an example where RTL arithmetic is mapped to pure wiring.

  ```tcl
  yosys
  read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  read_verilog mult_8.v
  synth -top mult8
  abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
  show
  ```
  <img width="813" height="194" alt="image" src="https://github.com/user-attachments/assets/687c619b-5f21-4459-954b-1631f2c70435" />


  - Inspect the netlist:
    ```tcl
    write_verilog -noattr mult8_net.v
    ```
    <img width="659" height="259" alt="image" src="https://github.com/user-attachments/assets/7aada764-6caa-4b2e-a04b-9dfe5301c33d" />


    - Confirm that the netlist duplicates A ("AA" wires for Y), verifying there is zero hardware cost.

---

### 5. Interpretation and Key Observations

- In both simple multiply-by-two and multiply-by-nine cases, Yosys can remove all logic and optimize to direct wiring.
- This shows the effect of *strength reduction* in hardware: multipliers by powers of two (or sums of shifted values) often do not require arithmetic gates, just wiring.

---

This concludes the lab on D flip-flop simulation, synthesis techniques, and optimization cases for arithmetic in hardware description and synthesis.
