---
name: codex-code-review
description: Orchestrate an Opus-writes → Codex-audits → Opus-fixes → Codex-re-audits → push loop. Use when the user asks to "write and audit code", "codex review", "codex code review", "codex check", "code review with codex", "dual AI review", "codex audit loop", "let codex check", "let codex review", "write then push with review", "写完让 codex 检查", "用 codex 审", "codex 审查", "codex 检查", "让 codex 看看", "帮我写完代码然后审一下", or wants Codex-verified code before pushing. Requires local `codex` CLI (codex-cli).
---

# Codex Audit Loop

Opus writes code, Codex audits it for bugs/security/performance/style, Opus fixes findings, Codex confirms clean, then push with user approval.

## Workflow

```
Phase 1        Phase 2         Phase 3        Phase 4         Phase 5
Opus writes → Codex audits → Opus fixes → Codex re-audits → Push
  + commit      (review)      + commit       (confirm)       (ask first)
                  │                              │
                  └── issues? → back to 3 ───────┘ clean? → 5
                                (max 3 rounds)
```

## Phase 1: Confirm Codex Settings

Present default configuration and ask user to confirm once:

```
Codex audit config: model=gpt-5.4, reasoning=high
Proceed with these settings? (or specify changes)
```

If user disagrees, ask for preferred model and reasoning effort, then continue.

## Phase 2: Opus Writes Code

Implement the requested feature or fix. Commit before auditing:

```bash
git add <changed-files>
git commit -m "<type>: <description>"
```

## Phase 3: Codex Audits

Determine the base branch (`main`, `master`, or the branch the current branch was created from), then run:

```bash
codex review --base <base-branch> -c model="<model>" -c model_reasoning_effort="<effort>" 2>/dev/null
```

**Parse output by severity (priority order):**

1. **Bugs / Logic errors** — must fix
2. **Security issues** — must fix
3. **Performance problems** — fix if reasonable
4. **Style / conventions** — fix at discretion

If output is clean (no bug/security/performance findings), skip to Phase 5.

## Phase 4: Opus Fixes

For each finding from Codex:

1. Read the relevant file
2. Apply the fix (immutable patterns)
3. Verify the fix addresses the specific Codex concern

Commit all fixes:

```bash
git add <fixed-files>
git commit -m "fix: address codex review findings"
```

## Phase 5: Codex Re-audits

Run the same `codex review` command again to confirm fixes are clean.

- **Still has issues** → repeat Phase 4 → Phase 5 (max 3 total audit rounds)
- **After 3 rounds with remaining issues** → present issues to user, ask how to proceed
- **Clean** → announce "Codex audit passed" and proceed to Phase 6

## Phase 6: Push

Ask user for confirmation before pushing:

```
Codex audit passed (N round(s)). Push to origin/<branch>?
```

On confirmation:

```bash
git push -u origin <branch>
```

## Resuming Codex for Discussion

When a Codex review raises a complex or ambiguous finding, resume the session to discuss rather than guessing intent:

```bash
echo "<follow-up question or context>" | codex exec resume --last 2>/dev/null
```

Use resume when:
- A finding is unclear and needs Codex to elaborate
- You want Codex to evaluate your proposed fix before applying it
- The finding involves a trade-off that benefits from discussion

Do not pass `-c model=` or `-c model_reasoning_effort=` on resume unless the user explicitly requests it — resume inherits the original session config.

## Critical Evaluation of Codex Output

Treat Codex as a knowledgeable colleague, not an authority:

- **Trust your own knowledge** — if Codex flags something that is actually correct, do not blindly "fix" it
- **Codex has knowledge cutoffs** — it may be wrong about latest API signatures, library versions, or model names
- **Research disagreements** — when Opus and Codex disagree, check the actual source (docs, types, tests) before deciding
- **State disagreements clearly** — tell the user: "Codex flagged X, but I believe Y because Z. Your call."
- **Never defer blindly** — changing correct code to satisfy an incorrect review finding introduces bugs

## Rules

- Always commit before auditing — Codex reviews diffs, uncommitted changes produce noisy reviews
- Max 3 audit rounds — escalate to user if issues persist
- Never skip re-audit — Phase 5 is mandatory after fixes
- Always ask before push — never auto-push
- Suppress stderr by default (`2>/dev/null`) — remove only if user asks for full output

## Error Handling

- **Codex not found**: tell user to install with `npm i -g @openai/codex`
- **Non-zero exit**: stop, show error, ask user how to proceed
- **No git repo**: inform user this workflow requires a git repository
