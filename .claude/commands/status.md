Show comprehensive DIU OS project status:

1. **Git Status**: For each repo, show branch, uncommitted changes, ahead/behind
2. **Smart Contracts**: List all .rs files in contract dirs, compilation status
3. **Backend**: Check if backend exists, dependencies, compilation status
4. **Frontend**: Check physics-tutorial build status, dependencies
5. **Tests**: Run `cargo test --workspace` and `npm test`, show pass/fail counts
6. **Dependencies**: Check for outdated crates and npm packages
7. **TODO/FIXME**: Search all source files for TODO and FIXME comments

Present as a dashboard-style summary.
