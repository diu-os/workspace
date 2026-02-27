Analyze the OLD Solidity contracts in the `contracts/` directory.
This code is DEPRECATED and must NOT be used directly. Purpose: extract business logic for migration to Rust/Stylus.

For each .sol file found:
1. List all functions and their signatures
2. Extract business logic and state variables
3. Identify security patterns used
4. Note what can be directly translated to Rust/Stylus
5. Flag Solidity-specific patterns that need different approach in Stylus

Output a migration reference document showing old Solidity → new Rust/Stylus mapping.
Do NOT suggest keeping any Solidity code.
