## Summary

This session covers the theoretical foundations and significance of **Gate Level Simulation (GLS)** in digital design verification, with a particular focus on the concept of **synthesis and simulation mismatches**. GLS is essential for validating a design's logical correctness post-synthesis and identifying discrepancies that may arise between register-transfer level (RTL) and gate-level behaviors, primarily due to misunderstandings in Verilog coding practices and timing.


## Core Concepts

### What is Gate Level Simulation (GLS)?

**Gate Level Simulation** is the process of simulating a synthesized digital circuit using the gate-level netlist rather than the RTL code. The netlist contains logical gates and flip-flops that represent the actual synthesized design. GLS is performed by applying the same testbench used in RTL simulation, but substituting the RTL code with the netlist design.

- The netlist is logically expected to be functionally equivalent to the RTL from which it was derived.
- Standard cell definitions must be provided so the simulator understands the gate-level primitives used in the netlist.
- GLS can be run with or without timing information. When timing-aware (e.g., delay annotation from .SDF files), GLS checks both logic and precise signal timing, though only functional checking is discussed in this context.


### Why Run GLS?

- **Logical Equivalence Verification:** Ensures the synthesized (netlist) design remains functionally identical to its RTL specification.
- **Catching Synthesis-Simulation Mismatches:** Identifies discrepancies introduced by coding errors or synthesis tool limitations.
- **Timing Validation:** With timing models, GLS verifies that design meets setup and hold constraints (not fully addressed here).
- **Realistic Hardware Validation:** Confirms actual hardware behavior by simulating the real gate-level structure, catching issues not seen at RTL.


### Synthesis and Simulation Mismatches

A **synthesis-simulation mismatch** occurs when the synthesized hardware behaves differently from simulation due to various pitfalls in writing RTL code. The primary causes discussed are:

- **Missing Sensitivity List:** Incomplete sensitivity lists in Verilog "always" blocks can make the functional simulation differ from synthesized hardware.
- **Blocking (`=`) vs. Non-blocking (`<=`) Assignments:** Incorrect use, especially in sequential logic, can result in mismatched behavior between simulation and actual synthesized hardware.
- **Non-standard Verilog Coding:** Any syntax or constructs not interpreted identically by the simulator and synthesizer can induce mismatches.

<img width="944" height="506" alt="image" src="https://github.com/user-attachments/assets/84555f30-d9eb-4289-8759-3204ac3ae240" />


## Key Points

- **Testbench Reuse:** The same testbench used for RTL simulation is used for GLS, but the "design under test" is now the gate-level netlist. Inputs and outputs remain consistent between RTL and netlist.
- **Netlist and Gate-Level Models:** Netlists are composed of standard gates (AND, OR, etc.), which are defined by gate-level Verilog models provided to the simulator.
- **Functional vs. Timing Models:**
  - *Functional models* allow checking logic equivalence.
  - *Timing-aware models* allow verification of temporal behavior, such as meeting setup/hold requirements.
- **Importance of Complete Sensitivity Lists:** In "always" blocks, failing to include all relevant signals causes the simulator to miss events that the synthesizer will capture, e.g., causing latching or "memory" effects in the simulated design.
    - Correct practice: Use "always @*" (or "always @(*)") to ensure all inputs are in the sensitivity list.
- **Blocking Assignment Pitfalls:**
  - Order of assignments affects behavior. Statements are executed sequentially, so earlier assignments impact later ones within the same block.
  - For example, shifting values in a register chain with blocking assignments may yield incorrect logic if the order is wrong.
- **Non-blocking Assignment for Sequential Logic:** Non-blocking assignments make all right-hand sides (RHS) evaluate before any left-hand side (LHS) assignment. This models hardware flip-flop behavior faithfully, where assignment order is irrelevant.
    - Always prefer non-blocking assignments (`<=`) for sequential logic to avoid mismatches.
- **Blocking Assignment in Combinational Logic:** Blocking assignments are suitable (and often necessary) for purely combinational logic where statement order mirrors real hardware path.
- **Examples of Synthesis-Simulation Mismatches:**
    - Shift register implemented with blocking assignments or misordered assignments may result in RTL simulation behaving differently from synthesized hardware.
    - An always block with an incomplete sensitivity list can make a modeled multiplexer act like a latch or a double-edge flip-flop in simulation.
- **GLS Mandatory for Verification:** Due to these potential mismatches, running GLS ensures that what will be fabricated (the netlist-driven hardware) matches expectations from functional simulation.


- **Order of Evaluation:**
  - *Blocking assignments* (`=`): Statements executed top-to-bottom; each assignment is immediate.
  - *Non-blocking assignments* (`<=`): All RHSes are sampled before any LHS is updated; thus, order does not matter.

- **Moral for Design Practice:**
  - Use non-blocking assignments in sequential logic.
  - Ensure sensitivity lists cover all relevant inputs in combinational logic.
