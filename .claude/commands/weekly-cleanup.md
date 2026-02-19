# Weekly Project Cleanup

Run the following steps to optimize the DIU OS workspace.

## 1. Check target/ size before cleanup

```bash
du -sh ~/projects/diu-os/diu-contracts/target/ 2>/dev/null || echo "target/ not found"
```

## 2. Clean build artifacts

```bash
cargo clean --manifest-path ~/projects/diu-os/diu-contracts/Cargo.toml
```

## 3. Purge old Claude logs (>14 days)

```bash
BEFORE=$(du -sh ~/.claude/logs/ 2>/dev/null | cut -f1)
find ~/.claude/logs -name "*.log" -mtime +14 -delete 2>/dev/null
AFTER=$(du -sh ~/.claude/logs/ 2>/dev/null | cut -f1)
echo "Logs: $BEFORE → $AFTER"
```

## 4. Run full test suite

Verify all 139 tests pass after cleaning:

```bash
cargo test --manifest-path ~/projects/diu-os/diu-contracts/Cargo.toml 2>&1 | tail -20
```

## 5. Git status — all repos

```bash
for repo in diu-contracts physics-tutorial diu-docs diu-os.github.io developer-portal manifesto; do
  path=~/projects/diu-os/$repo
  if [ -d "$path/.git" ]; then
    STATUS=$(git -C "$path" status --short)
    BRANCH=$(git -C "$path" branch --show-current)
    if [ -z "$STATUS" ]; then
      echo "✓ $repo ($BRANCH) — clean"
    else
      echo "! $repo ($BRANCH) — uncommitted changes:"
      echo "$STATUS" | sed 's/^/    /'
    fi
  fi
done
```

## 6. Usage reminder

Check `/usage` in Claude Code to review token consumption for the week.
Compare with previous week to spot unusually large sessions.

## 7. Summary

Report the following:
- Space freed in `target/` (before vs after `cargo clean`)
- Test result: pass/fail count
- Repos with uncommitted changes (if any)
- Recommendation: redeploy needed? (if contracts changed since last deployment on 15 Feb 2026)

Deployed addresses for reference — if contracts unchanged, no redeploy needed:
- DIURegistry: `0x3873828826a5e7768d2ad934b8466f817d5d5d07`
- DIUReputation: `0x10696cc645b4a9adbdede4a1e3d515621140a83c`
- DIUAchievements: `0x72bbe907b62ac1d964610f7019aa19b986986535`
- DIUToken: `0x35ec6ca36f7e9b1d35ffe8e74e78a5882f899ce2`
