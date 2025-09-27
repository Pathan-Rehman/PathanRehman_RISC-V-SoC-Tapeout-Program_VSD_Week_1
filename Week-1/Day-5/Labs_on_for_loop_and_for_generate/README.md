## Lab Introduction

This lab demonstrates how to elegantly design and simulate parameterizable hardware components using Verilog’s `for` loop and `for generate` constructs. You will:
- Implement a 4-to-1 multiplexer using a `for` loop.
- Compare the `for` loop approach to the traditional `case` statement for scalability.
- Implement an 8-output demultiplexer using both direct assignments and a `for` loop, analyzing code efficiency and maintainability.
- Use the `for generate` construct to instantiate a parameterized ripple-carry adder (RCA) composed of full adder modules.
- Simulate RTL using Icarus Verilog and view waveforms in GTKWave.

## Lab Steps

### 1. Multiplexer Using `for` Loop

1. **Locate the Multiplexer Design**
   - File: `verilog_files/mux_generate.v`

2. **Review Verilog Module Structure**
   - The module has these ports:
     - Inputs: `I0`, `I1`, `I2`, `I3`
     - Select lines: 2-bit `select`
     - Output: 1-bit `Y`
   - An internal wire bus is created:
     ```verilog
     wire [3:0] I_int;
     ```
     <img width="798" height="319" alt="image" src="https://github.com/user-attachments/assets/41bda60a-f091-4004-ab14-2c011feb21fe" />


3. **Multiplexer Functionality with `for` Loop**
   - Assign each input to the internal bus:
     ```verilog
     assign i_int[0] = i0;
     assign i_int[1] = i1;
     assign i_int[2] = i2;
     assign i_int[3] = i3;
     ```
   - Use a `for` loop inside `always @*` to select the correct input:
     ```verilog
     always @*
     begin
       integer K;
       for (K = 0; K < 4; K = K + 1) begin
         if (K == sel)
           Y = i_int[K];
       end
     end
     ```

4. **Simulate the Multiplexer**
   - Create corresponding testbench: `tb_mux_generate.v`
   - Compile and run:
     ```sh
     iverilog mux_generate.v tb_mux_generate.v -o a.out
     ```
     <img width="806" height="49" alt="image" src="https://github.com/user-attachments/assets/bb81210b-c758-4861-8066-7147f6ac37ca" />


   - Generate simulation waveforms:
     ```
     ./a.out
     ```
     <img width="813" height="95" alt="image" src="https://github.com/user-attachments/assets/857dbe55-63b2-48e0-88a7-a58194202bfb" />


   - View the waveform:
     ```
     gtkwave tb_mux_generate.vcd
     ```
     <img width="1070" height="564" alt="image" src="https://github.com/user-attachments/assets/ecd634b1-e3aa-49fc-a8f5-ed94363abeea" />


   - Analyze: Output `Y` matches `I0`, `I1`, `I2`, or `I3` based on the `select` input.

### 2. Multiplexer Using `case` Statement (Scalability Discussion)

- Equivalent code via `case`:
  ```verilog
  always @* begin
    case (select)
      2'b00: Y = I0;
      2'b01: Y = I1;
      2'b10: Y = I2;
      2'b11: Y = I3;
    endcase
  end
  ```

- For a larger mux (e.g., 256-to-1), the `for` approach requires minimal changes (bus width and loop limit), but a `case` statement would need 256 lines.

### 3. Demultiplexer: Direct Assignment vs. `for` Loop

1. **Direct Assignment Approach**
   - Outputs: `O0`–`O7`
   - Inputs: 3-bit `select`, 1-bit `I`
   - Assign all outputs to 0, set one as input based on `select`:
     ```verilog
     always @* begin
       O0 = 0; O1 = 0; O2 = 0; O3 = 0;
       O4 = 0; O5 = 0; O6 = 0; O7 = 0;
       case (select)
         3'b000: O0 = I;
         3'b001: O1 = I;
         ...
         3'b111: O7 = I;
       endcase
     end
     ```

2. **Demux Using `for` Loop**
   - Assign all outputs in a succinct way:
     ```verilog
     always @* begin
       integer K;
       for (K = 0; K < 8; K = K + 1)
         O[K] = (K == select) ? I : 0;
     end
     ```

   - For larger demuxes, `for` keeps code short and scalable while `case` grows unwieldy.

3. **Simulate Both Demux Designs**
   - Testbenches: `tb_demux_case.v`, `tb_demux_generate.v`
   - Compile and run:
     ```sh
     iverilog demux_case.v tb_demux_case.v -o a.out
     ./a.out
     gtkwave tb_demux_case.vcd
     ```
     <img width="1069" height="630" alt="image" src="https://github.com/user-attachments/assets/40d362ec-7c19-4eff-92cc-d17addcbf41d" />


   - Output results: Only the selected output follows input; others stay 0.

### 4. Parameterized Ripple-Carry Adder (RCA) with `for generate`

1. **Full Adder Module: `fa.v`**
   - Computes sum and carry for one-bit addition:
     ```verilog
     module fa (input a , input b , input c, output co , output sum);
	      assign {co,sum}  = a + b + c ;
      endmodule
     ```

2. **RCA Module: `rca.v`**
   - Adds two 8-bit numbers; output is 9-bit.
   - Sum: `$sum = a + b$`, where a, b are 8-bit, so sum needs 9 bits.
      - General rule: If you add $N$-bit and $M$-bit numbers, width is `max(N, M) + 1`.

   - Connections:
     - First adder: input carry = 0
     - Subsequent adders: chain carries using internal wire arrays.

   - Instantiate first adder explicitly:
     ```verilog
     fa fa0 (.A(num1[0]), .B(num2[0]), .Cin(1'b0), .Sum(int_sum[0]), .Cout(int_co[0]));
     ```
     <img width="879" height="411" alt="image" src="https://github.com/user-attachments/assets/61d3b951-4cc2-4d42-9582-0f77674dc0dd" />


   - Use `for generate` for the rest:
     ```verilog
     genvar i;
     generate
       for (i = 1; i < 8; i = i + 1) begin : genblk
         fa fa_inst (
           .A(num1[i]),
           .B(num2[i]),
           .Cin(int_co[i-1]),
           .Sum(int_sum[i]),
           .Cout(int_co[i])
         );
       end
     endgenerate
     ```
     - Concatenate final outputs:
       ```verilog
       assign sum[7:0] = int_sum;
       assign sum[8] = int_co[7];
       ```
     ![PLACEHOLDER: screenshot/terminal-output/schematic]

3. **Simulate RCA**
   - Make sure to include both full adder and RCA files at simulation:
     ```sh
     iverilog rca.v fa.v tb_rca.v -o a.out
     ./a.out
     gtkwave tb_rca.vcd
     ```
     <img width="1079" height="493" alt="image" src="https://github.com/user-attachments/assets/413c509b-0f95-488b-812b-604341482acb" />


   - In waveforms, verify: For various num1, num2 inputs, the output sum matches $sum = num1 + num2$.

### 5. Synthesis and Advanced Steps (Assignment)

- The lab instructs you to synthesize these designs and check:
  - For the mux, verify whether the RTL and synthesized logic both yield the equation $Y = I_{select}$ for a 4-to-1 mux.
 
<img width="1066" height="702" alt="image" src="https://github.com/user-attachments/assets/a68c73eb-2a9e-47e2-ae93-dcc3326608b5" />
<img width="310" height="249" alt="image" src="https://github.com/user-attachments/assets/ef154bc1-76d0-4012-9711-c3b33574a792" />


  - For RCA, perform synthesis and gate-level simulation (GLS) to verify correctness post-synthesis.
- Example synthesis tool invocation and waveform view is left for you to experiment.

<img width="428" height="397" alt="image" src="https://github.com/user-attachments/assets/d10b7613-c5df-4ad1-a09b-7f4f6af53be8" />

<img width="887" height="739" alt="image" src="https://github.com/user-attachments/assets/c150e075-f65f-48d0-b946-04a8edff2cab" />

```sh
  /* Generated by Yosys 0.33 (git sha1 2584903a060) */

module fa(a, b, c, co, sum);
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  wire _4_;
  wire _5_;
  wire _6_;
  wire _7_;
  input a;
  wire a;
  input b;
  wire b;
  input c;
  wire c;
  output co;
  wire co;
  output sum;
  wire sum;
  sky130_fd_sc_hd__maj3_1 _8_ (
    .A(_5_),
    .B(_3_),
    .C(_4_),
    .X(_6_)
  );
  sky130_fd_sc_hd__xor3_1 _9_ (
    .A(_5_),
    .B(_3_),
    .C(_4_),
    .X(_7_)
  );
  assign _5_ = c;
  assign _3_ = a;
  assign _4_ = b;
  assign co = _6_;
  assign sum = _7_;
endmodule

module rca(num1, num2, sum);
  wire [7:0] int_co;
  wire [7:0] int_sum;
  input [7:0] num1;
  wire [7:0] num1;
  input [7:0] num2;
  wire [7:0] num2;
  output [8:0] sum;
  wire [8:0] sum;
  fa \genblk1[1].u_fa_1  (
    .a(num1[1]),
    .b(num2[1]),
    .c(int_co[0]),
    .co(int_co[1]),
    .sum(int_sum[1])
  );
  fa \genblk1[2].u_fa_1  (
    .a(num1[2]),
    .b(num2[2]),
    .c(int_co[1]),
    .co(int_co[2]),
    .sum(int_sum[2])
  );
  fa \genblk1[3].u_fa_1  (
    .a(num1[3]),
    .b(num2[3]),
    .c(int_co[2]),
    .co(int_co[3]),
    .sum(int_sum[3])
  );
  fa \genblk1[4].u_fa_1  (
    .a(num1[4]),
    .b(num2[4]),
    .c(int_co[3]),
    .co(int_co[4]),
    .sum(int_sum[4])
  );
  fa \genblk1[5].u_fa_1  (
    .a(num1[5]),
    .b(num2[5]),
    .c(int_co[4]),
    .co(int_co[5]),
    .sum(int_sum[5])
  );
  fa \genblk1[6].u_fa_1  (
    .a(num1[6]),
    .b(num2[6]),
    .c(int_co[5]),
    .co(int_co[6]),
    .sum(int_sum[6])
  );
  fa \genblk1[7].u_fa_1  (
    .a(num1[7]),
    .b(num2[7]),
    .c(int_co[6]),
    .co(int_co[7]),
    .sum(int_sum[7])
  );
  fa u_fa_0 (
    .a(num1[0]),
    .b(num2[0]),
    .c(1'h0),
    .co(int_co[0]),
    .sum(int_sum[0])
  );
  assign sum = { int_co[7], int_sum };
endmodule
```
<img width="804" height="158" alt="image" src="https://github.com/user-attachments/assets/ad4147b2-55a6-437d-b01d-64a4b862db2f" />
<img width="1077" height="527" alt="image" src="https://github.com/user-attachments/assets/1c80c2c8-8814-4441-9b0d-070b51dfc9aa" />

---

*End of lab instructions. Try out synthesis and GLS simulation as an exercise.*
