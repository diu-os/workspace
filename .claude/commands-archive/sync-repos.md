Synchronize all DIU OS repositories from GitHub.

For each repo in the diu-os organization:
1. If already cloned locally, run `git pull origin main`
2. If not cloned, run `git clone git@github.com:diu-os/REPO_NAME.git`
3. Show current branch and status for each repo

Known repos to sync:
- physics-tutorial
- diu-os.github.io
- developer-portal
- manifesto
- contracts

After sync, show a summary table:
| Repo | Branch | Status | Last Commit | Behind/Ahead |
