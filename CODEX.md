# Codex Initialization Notes

## 1. Preflight (Run Before Any Change)
- Confirm canonical repo path:
  - `git rev-parse --show-toplevel`
- Confirm remote:
  - `git remote -v`
- Confirm branch status:
  - `git status -sb`

If these do not match the expected repository, stop and switch to the correct clone before editing.

## 2. Clone Hygiene
- Keep one canonical working clone per repository.
- Avoid working across duplicate local clones of the same remote.
- If duplicates exist, either archive/remove extras or make one clearly read-only.

## 3. Documentation Loading
Before implementing changes, read all markdown docs under `nagz/Docs/` and `README.md`.

Required docs:
- `nagz/Docs/REQUIREMENTS.md`
- `nagz/Docs/PREFERENCES.md`
- `nagz/Docs/ARCHITECTURE.md`
- `README.md`

## 4. Xcode Text File Visibility Rule
- If the project uses a filesystem-synchronized group, keep docs/text files inside that synced folder (for this repo: `nagz/Docs/`) so they appear automatically in Xcode.
- If the project does not use filesystem synchronization, add docs explicitly in Xcode via `Add Files to ...`, preferably under a `Docs` group.
