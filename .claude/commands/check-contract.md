Run full contract verification pipeline:
1. `cargo clippy -- -D warnings` — lint check
2. `cargo test -- --nocapture` — run all tests with output
3. `cargo stylus check` — verify WASM compilation for Arbitrum

If any step fails, analyze the error and suggest a fix.
Show a summary table: step | status | details
