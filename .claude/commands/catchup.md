Read all files changed in the current git branch compared to main. For each changed file, summarize what changed and note any issues. If there are no git changes, read recent files modified in the last session.

Steps:
1. Run `git diff --name-only main...HEAD` to find changed files
2. If no branch changes, run `git diff --name-only HEAD` for uncommitted changes
3. If still nothing, run `find . -name '*.rs' -newer CLAUDE.md -not -path '*/target/*'` for recently modified files
4. Read each changed file and give a brief status summary
5. End with: "Ready to continue. What's next?"
