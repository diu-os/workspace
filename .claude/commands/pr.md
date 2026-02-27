Review all uncommitted changes in the current repo. For each changed file:
1. Run `cargo clippy -- -D warnings` (if Rust project)
2. Run `cargo test` (if Rust project)
3. Fix any issues found
4. Stage all changes with `git add -A`
5. Generate a conventional commit message based on the changes
6. Show the commit message for approval before committing

Do NOT push — just prepare the commit.
