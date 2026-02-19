Show ecosystem stats: working directory, lines of code, test counts, and git status.

## Instructions

1. **Print current working directory** and its purpose:
   - Run `pwd` to get the current directory
   - Show: `**Working directory:** /path/to/dir`
   - Show the current git branch if in a git repo

2. **Count lines of code** in parallel across all 3 repos:
   - nagzerver source: `find ~/nagzerver/src -name '*.py' | xargs wc -l | tail -1`
   - nagzerver tests: `find ~/nagzerver/tests -name '*.py' | xargs wc -l | tail -1`
   - nagz-web source (non-test): `find ~/nagz-web/src -name '*.ts' -o -name '*.tsx' | grep -v __tests__ | xargs wc -l | tail -1`
   - nagz-web tests: `find ~/nagz-web/src/__tests__ -name '*.ts' -o -name '*.tsx' | xargs wc -l | tail -1`
   - nagz-ios source: `find ~/nagz-ios/Nagz -name '*.swift' | xargs wc -l | tail -1`
   - nagz-ios tests: `find ~/nagz-ios/NagzTests -name '*.swift' | xargs wc -l | tail -1`

3. **Get test counts** from each repo's CLAUDE.md (look for the test count in parentheses).

4. **Check git status** of all 4 repos (nagz, nagzerver, nagz-web, nagz-ios) — report clean or dirty.

5. **List available skills** from `~/nagz/.claude/commands/`:
   - Read each `.md` file's first line (the description)
   - Present as a skills table

6. **Present results** — first the working directory info, then the stats table, then the skills table:

| Repo | Source LoC | Test LoC | Tests | Git Status |
|------|-----------|----------|-------|------------|
| nagzerver | X,XXX | X,XXX | NNN | clean/dirty |
| nagz-web | X,XXX | X,XXX | NNN | clean/dirty |
| nagz-ios | X,XXX | X,XXX | NNN | clean/dirty |
| **Total** | **XX,XXX** | **X,XXX** | **NNN** | |

| Skill | Description |
|-------|-------------|
| `/skill-name` | First line of skill file |

7. Format numbers with commas for readability.
