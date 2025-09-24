## Summary

This module introduces **iVerilog** as a simulator for verifying Verilog RTL designs, explains the roles of the **design** and **test bench**, and walks through the fundamental simulation process using the iVerilog toolchain. It covers how simulators work, test benches are structured, and how simulation results are visualized using tools like GTKWave.

## Core Concepts

### Role of a Simulator

- A **simulator** is used to verify that an RTL (Register Transfer Level) design adheres to its intended specifications.
- Simulation applies test inputs (stimulus) to the design and observes the outputs, allowing designers to confirm correctness before hardware implementation.

### Design and Test Bench

- **Design**: Actual Verilog code (may span multiple files) representing the hardware's intended functionality.
- **Test Bench (TB)**: A Verilog module that *instantiates* the design, applies stimulus (test vectors), and observes outputs to validate the design’s behavior.
    - The test bench does **not** have primary inputs or outputs; these are properties of the design under test.
    - The TB generates input stimuli and monitors output responses internally.

### Simulator Operation

<img width="1737" height="963" alt="image" src="https://github.com/user-attachments/assets/7b624f2f-107f-4f42-8c8c-73a48e61d980" />

- The simulator evaluates outputs **only when input signals change**.
- Simulation tools generate a **VCD (Value Change Dump) file**, capturing signal changes over time — essential for timing and functional analysis.

### Simulation Flow

<img width="1835" height="840" alt="image" src="https://github.com/user-attachments/assets/58b40665-07b8-43c1-baad-ee47a583d676" />

- **Design and test bench files** are compiled together using iVerilog.
- The compiled design is executed, producing simulation results.
- **GTKWave** or similar tools open the VCD file to visually inspect input and output waveforms.

## Lab Steps

### 1. Prepare Your Environment

- Ensure iVerilog and GTKWave are installed on your system.

### 2. Write the Design

Example: An **Inverter** (NOT gate), saved as `inverter.v`:

```verilog
module inverter(
    input a,
    output y
);
    not(y, a);
endmodule
```

<img width="689" height="230" alt="image" src="https://github.com/user-attachments/assets/7ce3ff9b-1d66-479f-ab85-a5d4e1c2e6b4" />


### 3. Write the Test Bench

Save the following as `inverter_tb.v` in the same directory:

```verilog
module inverter_test;

reg a;
wire y;

inverter uut(a, y); // Instantiate inverter

initial begin
    $dumpfile("inverter.vcd");  // Set VCD output filename
    $dumpvars(0, inverter_test); // Dump all variables in scope

    a = 0;
    #10 a = 1;
    #10 a = 0;
    #10 a = 1;
    #10 $finish();
end

endmodule
```

<img width="765" height="599" alt="image" src="https://github.com/user-attachments/assets/fd2a18af-41a5-4e73-89a7-b6188941c9e2" />


### 4. Open Terminal in Project Directory

Navigate to the directory containing both `inverter.v` and `inverter_tb.v`. On Linux/Mac:

```bash
cd /path/to/your/project
```

<img width="855" height="330" alt="image" src="https://github.com/user-attachments/assets/7b9781ff-4c37-48fa-acdc-3d61525e94d9" />


### 5. Compile the Design and Test Bench

Use iVerilog to compile both files:

```bash
iverilog -o inverter inverter.v inverter_tb.v
```

- `-o inverter` sets the name of the compiled output.
- Both source files are specified.

<img width="850" height="351" alt="image" src="https://github.com/user-attachments/assets/6358658a-1c1b-4c57-bb2e-88c138d73cc6" />


### 6. Run the Simulation

Execute the compiled simulation:

```bash
vvp inverter
```

- This generates the VCD file (`inverter.vcd`) as specified in the test bench.

<img width="849" height="413" alt="image" src="https://github.com/user-attachments/assets/f5061154-6ed0-4c67-bc0e-1e5d1fb6501f" />


### 7. View the Waveforms

Open the VCD file with GTKWave to visualize input and output signal changes:

```bash
gtkwave inverter.vcd
```

- Use GTKWave’s GUI to inspect signal transitions and verify they match expected results.

<img width="869" height="722" alt="image" src="https://github.com/user-attachments/assets/578859fb-1a14-496b-9ed5-2582074408ff" />


## Additional Notes

- Designs can span multiple files; compile by listing all files or using a file list.
- The test bench may be named differently for various modules but always *instantiates* the design under test, sets up stimulus, and observes outputs.
- **VCD files** are standard output for digital simulation, supporting post-simulation waveform analysis.
- No actual hardware is produced; simulation strictly checks logical correctness at the RTL level.

---

This documentation outlines the purpose and workflow of iVerilog-based simulation, emphasizing both *concept* and *step-by-step practical use* for beginners.
