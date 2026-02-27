---
name: stylus-contract
description: >
  Creates a new Stylus/Rust smart contract for DIU OS.
  Use when user mentions "new contract", "create contract",
  "DIUProgress", "DIUCrowdfunding", "новый контракт", "создай контракт".
---

# Skill: New Stylus Contract

Activate this skill whenever the user mentions creating, adding, or designing a new smart contract for DIU OS.

---

## Critical Patterns

### 1. Constructor — INSIDE `#[public]` impl

> ⚠️ **Critical**: `constructor` MUST live inside the `#[public]` impl block so Stylus ABI export sees it.
> ❌ **WARNING**: `registry.rs` has constructor OUTSIDE `#[public]` — this is a **known bug** causing `owner = 0x0000` on deploy. Do NOT copy this pattern.
> ✅ All new contracts: constructor strictly inside `#[public]`. Fix for existing contracts: migrate to `initialize()` pattern (see below).

```rust
#[public]
impl MyContract {
    /// Initializes the contract. Called once at deploy time.
    pub fn constructor(&mut self) {
        let deployer = self.vm().tx_origin();
        self.owner.set(deployer);
        self.admins.setter(deployer).set(true);
    }

    // ... rest of public functions
}
```

### 2. Initialize Pattern (upgradeable / proxy-ready)

When the contract must support re-initialization via proxy:

```rust
sol! {
    error AlreadyInitialized();
}

#[derive(SolidityError)]
pub enum ContractError {
    AlreadyInitialized(AlreadyInitialized),
    // ...
}

#[public]
impl MyContract {
    /// One-time initializer. Reverts if already called.
    pub fn initialize(&mut self, owner: Address) -> Result<(), ContractError> {
        if self.initialized.get() {
            return Err(ContractError::AlreadyInitialized(AlreadyInitialized {}));
        }
        self.initialized.set(true);
        self.owner.set(owner);
        self.admins.setter(owner).set(true);
        Ok(())
    }
}
```

> Use `constructor` for non-upgradeable contracts; use `initialize` for proxy/upgradeable contracts.

---

## File Structure (follow registry.rs as canonical reference)

```
diu-contracts/src/<name>.rs
```

Section order must match `registry.rs`:

```
// EVENTS      — sol! { event ... }
// ERRORS      — sol! { error ... } + #[derive(SolidityError)] enum
// STORAGE     — sol_storage! { #[entrypoint] pub struct ... }
// INTERNAL    — impl (private helpers: require_owner, require_admin, ...)
// CONSTRUCTOR — impl (constructor or initialize)
// PUBLIC      — #[public] impl (all external functions)
// TESTS       — #[cfg(test)] mod tests
```

---

## Storage Template

```rust
sol_storage! {
    #[entrypoint]
    pub struct MyContract {
        /// Contract owner.
        address owner;

        /// Addresses with admin privileges.
        mapping(address => bool) admins;

        // ... domain-specific fields
    }
}
```

`mapping(address => bool)` in Rust = `StorageMap<Address, StorageBool>`.

Read:  `self.admins.get(addr)`  → `bool`
Write: `self.admins.setter(addr).set(true)`

---

## Access Control Helpers (copy from registry.rs)

```rust
impl MyContract {
    fn require_owner(&self) -> Result<(), ContractError> {
        if self.vm().msg_sender() != self.owner.get() {
            return Err(ContractError::Unauthorized(Unauthorized {}));
        }
        Ok(())
    }

    fn require_admin(&self) -> Result<(), ContractError> {
        let caller = self.vm().msg_sender();
        if caller != self.owner.get() && !self.admins.get(caller) {
            return Err(ContractError::Unauthorized(Unauthorized {}));
        }
        Ok(())
    }
}
```

---

## Required Errors (always include)

```rust
sol! {
    error Unauthorized();
    error ZeroAddress();
    error AlreadyInitialized();   // if using initialize pattern
    error EmptyString();          // if accepting string params
}
```

---

## Feature Gate in Cargo.toml

Every new contract needs a feature gate. Add to `diu-contracts/Cargo.toml`:

```toml
[features]
my_contract = []   # snake_case name of the new contract
```

In `src/lib.rs`, wrap with:

```rust
#[cfg(feature = "my_contract")]
pub mod my_contract;
```

Only one `#[entrypoint]` can be active at a time — feature gates ensure this.

Deploy/check a single contract:
```bash
cargo stylus check --features my_contract --endpoint ...
cargo stylus deploy --features my_contract --endpoint ...
```

---

## Test Requirements (mandatory before any commit)

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use stylus_sdk::testing::*;
    use alloy_primitives::address;

    const OWNER: Address = address!("1111111111111111111111111111111111111111");
    const ALICE: Address = address!("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa");

    fn setup() -> (TestVM, MyContract) {
        let vm = TestVMBuilder::new().sender(OWNER).build();
        let mut contract = MyContract::from(&vm);
        contract.constructor();   // or contract.initialize(OWNER).unwrap()
        (vm, contract)
    }

    // Minimum test coverage required:
    // ✓ constructor sets owner and admin
    // ✓ happy path for every public write function
    // ✓ unauthorized caller reverts
    // ✓ duplicate/invalid input reverts
    // ✓ view functions return correct data
}
```

**Before every commit:**

```bash
cd diu-contracts
cargo test                        # all tests must pass
cargo clippy -- -D warnings       # zero warnings
```

---

## Checklist Before Marking Contract Done

- [ ] Constructor/initialize inside `#[public]` (or documented exception)
- [ ] `AlreadyInitialized` error if using initialize pattern
- [ ] `Unauthorized` + `ZeroAddress` errors defined
- [ ] `require_owner` / `require_admin` helpers in private `impl`
- [ ] Feature gate added to `Cargo.toml`
- [ ] Module added to `src/lib.rs` behind `#[cfg(feature)]`
- [ ] Tests: constructor + happy paths + error paths
- [ ] `cargo test` passes
- [ ] `cargo clippy -- -D warnings` passes (zero warnings)
- [ ] `cargo stylus check --features <name>` passes

---

## Reference

- Canonical template: `diu-contracts/src/registry.rs`
- Stylus by Example: https://stylus-by-example.org/
- Stylus SDK 0.10.0: https://docs.rs/stylus-sdk/0.10.0/stylus_sdk/
