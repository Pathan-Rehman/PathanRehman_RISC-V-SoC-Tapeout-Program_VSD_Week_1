## Summary

This documentation details the theoretical foundations and key concepts behind the use of **if** and **case** statements in Verilog synthesis, with an emphasis on their hardware implications, inherent dangers (such as inferred latches), and critical best practices to avoid common pitfalls when writing combinational and sequential logic.

## Core Concepts

### If Statements and Priority Logic

- The **if-else** statement in Verilog enables conditional execution where certain conditions are evaluated in order of precedence.
- Each **if** condition is checked sequentially; the first true condition determines the executed block, establishing **priority logic** in hardware.
- The structure:
  - `if (condition1) ... else if (condition2) ... else if (condition3) ... else ...`
  - Only the first matching condition’s logic is executed, and the rest are ignored.
- In hardware, this is synthesized as a chain where the output is directly controlled by the first true condition. This naturally creates a priority among inputs or signals.

<img width="1130" height="619" alt="image" src="https://github.com/user-attachments/assets/7dc19e23-211b-4d2c-be80-582cdca7bc84" />


### Inferred Latches from Incomplete If Statements

- **Inferred latches** arise when not all cases for an output signal are covered in combinational logic.
- If an output (e.g., `Y`) is assigned in some, but not all, branches of an if-else chain, synthesis tools infer a latch to "remember" the previous value when no condition matches.
- This typically happens in incomplete **if** statements without an encompassing **else** or when all possible input conditions are not addressed.
- Inferred latches can be problematic unless explicitly required (e.g., intentional sequential logic).

For example, when the code omits assigning `Y` for certain cases:
- If neither `condition1` nor `condition2` is true and there is no `else`, `Y` will retain its last value via a latch.

This can be mathematically modeled as:
  - $Y_{next} = Y_{prev}$ when no condition matches.

<img width="1141" height="635" alt="image" src="https://github.com/user-attachments/assets/7505e814-4cff-476e-9207-476106c59633" />


### Intended vs. Unintended Latching

- **Intended latching** (e.g., counters) requires remembering a value across clock cycles, matching sequential behavior.
- Example: In a counter, if an enable is not asserted, the counter value remains latched (does not increment).
- Such latching is valid in sequential circuit design.

- **Unintended latching** is typically an error in combinational design and should be avoided by ensuring all outputs are assigned for every path through the **if** logic.


### Case Statements: Multiplexer Inference

- **Case statements** in Verilog create hardware similar to a multiplexer: each case represents a possible input combination, assigning outputs accordingly.
- Syntax allows for expressive handling of inputs, with each case label corresponding to a selection input.
- A complete case statement for a signal with $n$ possible values considers all valid combinations.
- If select input is 2 bits, you expect 4 possibilities: `00`, `01`, `10`, `11`, building a $4 \times 1$ multiplexer.

<img width="986" height="644" alt="image" src="https://github.com/user-attachments/assets/112c09e1-2879-482d-bddf-3ccd610d37c8" />


### Incomplete Case Statements and Latch Inference

- Omitting cases in a case statement (handling only a subset of all possible selections) can result in inferred latches if outputs aren’t assigned for every case.
- The best practice is to always include a `default` case that covers all unspecified possibilities, ensuring that no output is left unassigned:

Example structure:
- `case (select) 2'b00: ...; 2'b01: ...; default: ...; endcase`

- Incomplete case statements without a default case may leave outputs undefined for some input values, again synthesizing as unwanted latches.

<img width="1126" height="643" alt="image" src="https://github.com/user-attachments/assets/843c40e6-48d8-4b80-9551-96bd4aa2100e" />


### Partial Assignments and Latches in Case Statements

- Assign all outputs in **every branch** of a case statement. Assigning only some outputs in a particular case leads to latching for the unassigned signals.
- Even with `default`, failing to assign a value to every output variable in each case block results in inferred latches.

**Moral:** Always assign all output variables in all case branches, including the default.

<img width="1120" height="650" alt="image" src="https://github.com/user-attachments/assets/4fc40aee-8745-4980-90f1-63be5912a58e" />


### Case vs. If-Else: Priority and Overlapping Cases

- **If-else chains** create explicit *priority logic*, where the first satisfied condition is chosen.
- **Case statements** are generally used for equality-based selection and do not provide inherent priority—order matters only if cases overlap.
- **Overlapping cases** (e.g., using wildcards or improper ordering) can lead to unpredictable or undesired behavior. Unlike if-else, case statements may proceed through multiple matching branches unless uniquely defined.

- Avoid overlapping or ambiguous case labels for reliability.

<img width="1227" height="729" alt="image" src="https://github.com/user-attachments/assets/b703b6f6-71e9-4fde-a9f1-9a82558aaa0d" />


## Key Points

- **If statements** create priority logic; only the first satisfied condition determines the outcome.
- **Incomplete if or case statements** in combinational logic lead to **inferred latches**, which are generally undesirable unless intentional for sequential logic.
- Always **fully specify output assignments** in every combinational branch (if, else, case, default) to prevent latch inference.
- Use a **default** case in every combinational case statement to catch all unspecified input values.
- Assign every output variable in every case branch, including the default.
- **If-else** provides priority; **case** does not inherently but must avoid overlapping and ambiguous conditions for safe synthesis.
