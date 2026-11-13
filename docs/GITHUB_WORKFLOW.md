# GitHub workflow — BugTwin

The notebook is the only application source. Documentation, CI configuration and dependency records may be tracked separately.

## First publication

Create an **empty** repository named `bugtwin` in GitHub (no README or license generated). Extract the delivery ZIP and open PowerShell in the `bugtwin` directory.

```powershell
git init
git branch -M main
git add .
git commit -m "feat: initialize BugTwin data and lexical retrieval harness"
# Substitute your own GitHub owner; do not paste a credential into the URL.
git remote add origin https://github.com/YOUR_USERNAME/bugtwin.git
git push -u origin main
```

If Git reports missing identity, configure `git config --global user.name "YOUR NAME"` and `git config --global user.email "YOUR VERIFIED EMAIL"`, then retry. GitHub authentication uses a credential manager, SSH key or GitHub CLI, not an account password pasted into the command.

## Subsequent checkpoint: one feature branch per milestone

```powershell
git switch main
git pull --ff-only origin main
git switch -c feature/m3-semantic-retrieval
# Make changes and run the notebook fixture tests.
git status
git add BugTwin.ipynb README.md docs .github
# Stage only reviewed files; do not stage raw data, caches or credentials.
git commit -m "feat: add semantic duplicate retrieval"
git push -u origin feature/m3-semantic-retrieval
```

Open a pull request from `feature/m3-semantic-retrieval` into `main`. Wait for `Notebook CI` to pass, review the diff, and merge via GitHub's **Squash and merge** option. Alternatively, with GitHub CLI (`gh` installed and authenticated):

```powershell
gh pr create --base main --head feature/m3-semantic-retrieval --fill
gh pr checks --watch
gh pr merge --squash --delete-branch
git switch main
git pull --ff-only origin main
```

GitHub's main-branch rulesets or protection can require a pull request and passing checks. Enable these in **Settings → Rules → Rulesets** (or the repository's Branches settings where available), targeting `main`.

## If the repo already exists

**Do not initialize over an existing working tree.** Clone it, create a feature branch, and copy/merge the supplied notebook and docs into that clone. Keep the current repository history. Run tests and open a PR; do not force-push `main`.

## Before merging any feature

1. Notebook executes end-to-end in fixture mode with all tests passing.
2. Colab has been exercised with the actual source data when claiming real source readiness.
3. New notebook code and data contracts have focused regression tests.
4. No `bugtwin_runtime`, datasets, model weights, API keys or notebook outputs are staged.
5. README and design records reflect the actual implemented features.
6. Pull request review and CI checks pass.

## Troubleshooting

- `fatal: not a git repository`: `cd` to the extracted `bugtwin` directory.
- `remote origin already exists`: inspect `git remote -v` before changing it.
- `git push` authentication fails: use `gh auth login` or configure Git Credential Manager.
- CI fails: reproduce locally with `BUGTWIN_M1_SOURCE=fixture` and inspect the failing cell.
- Dirty notebook diffs: clear outputs before committing. Downloaded data must remain outside Git.
