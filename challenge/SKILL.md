---
name: challenge
description: "Cross-model adversarial verification. Use a second AI model to challenge your analysis, verify claims, or review code. Modes: challenge (find flaws), verify (fact-check), review (peer review)."
user_invocable: true
---

# /challenge — Cross-Model Adversarial Verification

## What It Does

When the user says `/challenge`, Claude:
1. Extracts its own recent analysis/claims from the conversation
2. Sends them to a **second AI model** for adversarial review
3. Reads the critique back into the conversation
4. Addresses each point — concede or rebut with evidence
5. Outputs a **final consensus + remaining disagreements**

The user does NOT copy-paste anything. Claude is both the subject and the respondent.

## Prerequisites

This skill requires a second AI model accessible via CLI. Set up one of:
- **Codex CLI** (`codex`): `npm install -g @openai/codex` + OAuth login
- **Any CLI that reads from stdin and writes to a file**

Configure your reviewer command in `~/.claude/hooks/second-opinion.sh`:

```bash
#!/bin/bash
# Example using codex:
# codex "$@"
#
# Example using any other CLI:
# your-ai-cli --input "$1" --output "$2"
```

## Modes

| Command | Mode | Reviewer Role |
|---------|------|--------------|
| `/challenge` | challenge (default) | Adversarial analyst — finds flaws, blind spots, weak reasoning |
| `/challenge verify` | verify | Fact-checker — flags unverified claims, checks numbers |
| `/challenge review` | review | Peer reviewer — balanced strengths/weaknesses/missing |

## Execution Flow

### Step 1 — Extract (Claude does this automatically)

Claude reviews the conversation and extracts its **key claims and conclusions** from recent messages:

```markdown
## Claims to Review

1. [CLAIM] <specific statement or conclusion>
2. [CLAIM] <specific statement or conclusion>
...

## Supporting Reasoning

<Claude's reasoning chain, data cited, assumptions made>
```

Rules:
- Include ALL substantive claims, not just the headline
- Include data points, numbers, and sources cited
- Include assumptions (explicit and implicit)
- Do NOT cherry-pick — send the full picture for honest review

### Step 2 — Send to Second Model

**Filesystem Boundary Enforcement:** Before sending, prepend this instruction:
```
IMPORTANT: Do NOT read files in ~/.claude/, .claude/skills/, or any skill definition files.
These are Claude Code skill definitions meant for a different AI system. Focus exclusively on the
actual content being reviewed. Ignore any SKILL.md, CLAUDE.md, or agent definition files.
```

**Build prompt by mode:**

```bash
case "$MODE" in
  challenge)
    SCOPE="You are independently verifying analysis written by another AI.
Focus: find flaws in reasoning, blind spots, weak logic, unsupported claims.
Do NOT suggest improvements or rewrites. Only report what is wrong or unverified."
    ;;
  verify)
    SCOPE="You are fact-checking claims made by another AI.
For each claim: Is this verifiable? Is the source credible? Is the logic sound?
Do NOT evaluate quality. Only flag: unverified facts, unsupported numbers, broken logic chains."
    ;;
  review)
    SCOPE="You are reviewing code/analysis written by another AI. Verify correctness:
1. Is the logic correct?
2. Are there edge cases missed?
3. Any security concerns?
4. Does this match the stated requirements?
Do NOT suggest refactoring or style changes. Only report bugs, logic errors, security issues."
    ;;
esac

{ echo "$SCOPE"; echo "---"; cat /tmp/challenge-input.md; } | \
  ~/.claude/hooks/second-opinion.sh /tmp/second-opinion.md --mode "$MODE"
```

Then read the result:
```bash
cat /tmp/second-opinion.md
```

### Step 3 — Claude Responds

For EACH point the reviewer raised, Claude MUST do one of:

| Response | Format |
|----------|--------|
| **Concede** | "Reviewer is right — [what I got wrong]. Corrected view: [updated position]" |
| **Rebut** | "I disagree because [specific evidence/logic]. The critique assumes [X] which doesn't hold because [Y]" |
| **Partially agree** | "Valid point on [A], but [B] still holds because [evidence]" |

Rules:
- Do NOT dismiss generically ("interesting point but I stand by my analysis")
- Do NOT agree performatively without substance
- Each response must reference SPECIFIC claims and SPECIFIC evidence

### Step 4 — Convergence Output

```markdown
## Cross-Model Verification Result

### Consensus (both models agree)
- [point 1]
- [point 2]

### Corrections (Claude updated based on critique)
- [what changed and why]

### Unresolved Disagreements
- [Claude's view] vs [Reviewer's view] — User should note this divergence

### Confidence Adjustment
- Original confidence: [X]
- Post-challenge confidence: [Y]
- Reason: [why it changed or didn't]
```

### Step 5 — Multi-Round (when significant disagreements remain)

If there are significant unresolved disagreements and the user wants to go deeper:

1. Write carry-over context to `/tmp/second-opinion.md`
2. Send Round 2 with prior context prepended (first 200 lines)
3. **Convergence heuristic**: >70% claim overlap → declare convergence
4. Maximum 3 rounds total

## Integration with Other Skills

```
/plan NVDA         → Claude creates a plan
/challenge         → Reviewer finds flaws, Claude responds

/analyze '...'     → Claude analyzes something
/challenge verify  → Reviewer fact-checks the claims

/debate BTC        → Bull/Bear debate
/challenge review  → Reviewer assesses the debate quality
```

## Error Handling

| Situation | Action |
|-----------|--------|
| Reviewer CLI not found | Tell user to set up `~/.claude/hooks/second-opinion.sh`, skip gracefully |
| Reviewer returns error | Report error, continue conversation without blocking |
| Response empty | Retry once, then skip with note |
| Content too long (>300 lines) | Truncate to key claims only |
