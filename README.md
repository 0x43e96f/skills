# Claude Code Skills

A collection of Claude Code skills for structured development workflows.

## Skills

| Skill | Description | Source |
|-------|-------------|--------|
| [kickoff](./kickoff/SKILL.md) | "Interview Me" technique — Claude asks you questions before starting | Original |
| [pua](./pua/SKILL.md) | Full dev loop engine: Plan → Execute → Review → Security → Verify → Commit | Adapted from [tanweai/pua](https://github.com/tanweai/pua) |
| [office-hours](./office-hours/SKILL.md) | Product thinking partner — reframe the problem before writing code | Adapted from [garrytan/gstack](https://github.com/garrytan/gstack/blob/main/office-hours/SKILL.md) |
| [brainstorming](./brainstorming/SKILL.md) | Turn ideas into designs before writing any code | Adapted from [obra/superpowers](https://github.com/obra/superpowers) |
| [challenge](./challenge/SKILL.md) | Cross-model adversarial verification — use a second AI to challenge your analysis | Original |

### Also worth checking out

These skills are taken directly (or near-directly) from [obra/superpowers](https://github.com/obra/superpowers) — install from there:

- `systematic-debugging` — 4-phase root cause analysis before any fix
- `verification-before-completion` — verify with real commands before claiming done
- `finishing-a-development-branch` — structured options for merging/PR/cleanup
- `using-git-worktrees` — isolated worktrees for risky changes

## How to Install

Copy a skill folder into your `~/.claude/skills/` directory:

```bash
# Clone this repo
git clone https://github.com/0x43e96f/skills ~/skills-repo

# Copy a skill
cp -r ~/skills-repo/kickoff ~/.claude/skills/

# Or install all at once
cp -r ~/skills-repo/kickoff ~/skills-repo/pua ~/skills-repo/office-hours \
      ~/skills-repo/brainstorming ~/skills-repo/challenge \
      ~/.claude/skills/
```

Then invoke with `/kickoff`, `/pua`, `/office-hours`, etc. in Claude Code.

## Recommended Workflow

```
Unclear requirements  →  /kickoff    (let Claude ask you questions first)
Product sanity check  →  /office-hours  (optional)
Implementation plan   →  /plan
Validate the plan     →  /challenge  (optional — second opinion)
End-to-end execution  →  /pua
```

Not every task needs all steps. Small fixes → just `/pua`. Clear requirements → skip `/kickoff`.

## Credits

- [tanweai/pua](https://github.com/tanweai/pua) — original PUA pressure escalation concept
- [garrytan/gstack](https://github.com/garrytan/gstack) — office-hours YC-style forcing questions
- [obra/superpowers](https://github.com/obra/superpowers) — brainstorming, systematic-debugging, and other core skills
- [yeachan-heo/oh-my-claudecode](https://github.com/yeachan-heo/oh-my-claudecode) — multi-agent orchestration inspiration

## License

MIT
