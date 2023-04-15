# Understanding how the EVM executes the smart contract

- Bytecode
- Opcode
- Creation code
- Runtime code
- Simple Smart Contract
- Constructor
- Function Selector

- JUMPI: Change the Program counter to value the topmost item on the stack holds(i.e Change PC to the value stack[0] holds) if the value stack[1] hold is a non zero value. In a case where the value stack[1] holds is non zero, the PC is usually a JUMPDEST which indicates a target/valid location for JUMPs.
  Otherwise, if the value stack[1] holds is zero(0), just continue exceution.
