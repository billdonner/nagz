Show ecosystem stats: working directory, lines of code, test counts, and git status.

## Instructions

1. **Run a single Bash command** that gathers all stats at once:

```bash
echo "CWD:$(pwd):$(git branch --show-current 2>/dev/null || echo 'no-git')" && echo "SRC_SERVER:$(find ~/nagzerver/src -name '*.py' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "TST_SERVER:$(find ~/nagzerver/tests -name '*.py' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "SRC_WEB:$(find ~/nagz-web/src \( -name '*.ts' -o -name '*.tsx' \) ! -path '*__tests__*' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "TST_WEB:$(find ~/nagz-web/src/__tests__ \( -name '*.ts' -o -name '*.tsx' \) | xargs wc -l | tail -1 | awk '{print $1}')" && echo "SRC_IOS:$(find ~/nagz-ios/Nagz -name '*.swift' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "TST_IOS:$(find ~/nagz-ios/NagzTests -name '*.swift' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "GIT_HUB:$(cd ~/nagz && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && echo "GIT_SERVER:$(cd ~/nagzerver && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && echo "GIT_WEB:$(cd ~/nagz-web && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && echo "GIT_IOS:$(cd ~/nagz-ios && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && for f in ~/nagz/.claude/commands/*.md; do echo "SKILL:$(basename "$f" .md):$(head -1 "$f")"; done
```

2. **Parse the output** and present as markdown. Format numbers with commas.

3. **Output format:**

**Working directory:** `/path/to/dir` (branch: `main`)

| Repo | Source LoC | Test LoC | Tests | Git Status |
|------|-----------|----------|-------|------------|
| nagzerver | X,XXX | X,XXX | NNN | clean/dirty |
| nagz-web | X,XXX | X,XXX | NNN | clean/dirty |
| nagz-ios | X,XXX | X,XXX | NNN | clean/dirty |
| **Total** | **XX,XXX** | **X,XXX** | **NNN** | |

| Skill | Description |
|-------|-------------|
| `/skill-name` | First line of skill file |

4. **Get test counts** from each repo's CLAUDE.md (use Grep, search for test count in parentheses — do this in parallel with step 1 if possible, or read from memory).
