## Summary

This document explains the theoretical foundations of flip-flops in digital design. It focuses on the role of flip-flops as storage elements to stabilize digital circuit outputs, reduce glitches, and describes the differences between combinational and sequential logic. Various types of reset and set functionalities for flip-flops, including synchronous and asynchronous modes, are discussed along with conceptual distinctions between them.

## Core Concepts

### Combinational Circuits and Glitches

- **Combinational circuits** produce outputs based solely on their current inputs, with outputs updating after a certain propagation delay.
- When multiple inputs to a combinational circuit change simultaneously, outputs can temporarily switch to incorrect values before stabilizing, a phenomenon known as a **glitch**.
- Example: For a circuit with Boolean function

Y = (A & B) + C

if inputs change from $A=0$, $B=0$, $C=1$ to $A=1$, $B=1$, $C=0$ in the presence of gate delays, $Y$ may momentarily become $0$ (a glitch), even though its steady-state value before and after the transition is $1$.



### The Problem of Glitch Propagation

- In systems composed of chains of combinational circuits, glitches in the output of one stage can propagate and amplify through subsequent stages, leading to persistent instability in complex circuits.
- The more interconnected combinational logic stages present, the more frequent and severe the glitches can become.


### Flip-Flops as Storage and Shielding Elements

- A **flip-flop** is a sequential logic device used to store a single bit of data, maintaining its output value until explicitly changed by a control signal.
- Inserting flip-flops between combinational logic stages serves two purposes:
  - It shields downstream circuits from input glitches by only updating the stored value on a clock edge.
  - It allows system outputs to stabilize and prevents glitches from propagating beyond the flip-flop.
- The output $Q$ of a flip-flop remains stable and isolated from input $D$ except when a clock edge occurs.


### Initialization and Reset/Set Functionality

- Flip-flops must begin operation from a known, initialized state; otherwise, circuits depending on their outputs will evaluate incorrect or undefined results.
- Initialization is achieved using additional control inputs like **reset** and **set** pins.
- These controls can operate in two modes:
  - **Asynchronous**: Reset/set takes immediate effect, regardless of the clock.
  - **Synchronous**: Reset/set only takes effect during a clock event.


## Key Points

- **Glitches** in combinational circuits occur due to non-uniform propagation delays and can be suppressed by using flip-flops as synchronizing elements.
- **Flip-flops** update their state only on a clock edge, providing immunity to transient input changes and isolating stable outputs from upstream glitches.
- To guarantee known power-up conditions, **reset** or **set** inputs must be used; their operation modes (asynchronous or synchronous) directly affect circuit initialization behavior.
- **Asynchronous reset/set** controls react instantly to input changes, independently of the clock, immediately forcing the output to a defined state.
- **Synchronous reset/set** controls only act during a clock event, setting or resetting the state on the next clock edge if the control signal is active.
- The distinction between asynchronous and synchronous modes is critical for designing robust, predictable sequential circuits and avoiding unintended behavior such as race conditions.
