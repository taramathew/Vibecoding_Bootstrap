---
description: Analyse and optimise all project documentation — CLAUDE.md, rules, ADRs, logs, architecture docs
argument-hint: "[scope] — 'all' (default), 'claude-md', 'rules', 'adrs', 'logs', 'architecture'"
---

You are a documentation optimiser. Your job is to analyse the project's living documents,
remove stale content, tighten wording, resolve contradictions, and ensure every document
earns its place.

## Step 1: Read all documentation

Read these files in order:

1. `CLAUDE.md` — project rules and constraints
2. `.claude/rules/plan-workflow.md` — process rules (18 sections)
3. `.docs/logs/DECISIONS.md` — architectural decision records
4. `.docs/logs/IMPLEMENTATION_LOG.md` — chronological change journal
5. All files in `.docs/architecture/` — subsystem specifications
6. `.docs/FEATURE_BACKLOG.md` — feature tracker

If `$ARGUMENTS` specifies a scope other than "all", only read and optimise the
targeted documents. Still read CLAUDE.md and DECISIONS.md for cross-reference context.

## Step 2: Staleness audit

For each document, check:

### CLAUDE.md
- Are all file paths in "Architecture Reference" still valid? (Glob for each path)
- Are all "Critical Rules" still relevant? (Search codebase — is the rule's subject still in use?)
- Are tech stack entries accurate? (Check package.json, imports)
- Are "Common Commands" still correct? (Verify scripts exist in package.json)
- Do quality gate commands match what CI actually runs?
- Are there duplicate rules saying the same thing in different words?

### .claude/rules/plan-workflow.md
- Do "Seeded Principles" reference phases/features that still exist?
- Are anti-patterns in §16/§17/§18 still relevant? (Is the technology/pattern still in use?)
- Does "Which Module to Read" (§6) list files that exist? (Glob for each path)
- Are there anti-patterns that duplicate CLAUDE.md rules? (Cross-reference)
- Is the `<!-- Last Updated -->` header current?

### DECISIONS.md
- Are any ADRs marked "Implemented" but the code has since been removed or replaced?
- Are any ADRs superseded by later ADRs but not marked "Superseded"?
- Are ADR numbers sequential with no gaps? (Gaps are OK — just note them)
- Do any ADRs reference files/components that no longer exist?

### IMPLEMENTATION_LOG.md
- Are there entries older than the last major phase boundary that should be archived?
- Are there entries with "Decisions" sections that were never promoted to ADRs?
  (Apply Knowledge Classification §1 — if the decision is still relevant, promote it now)
- Are there duplicate entries describing the same change?

### Architecture docs (.docs/architecture/*.md)
- Does each doc describe the system as it currently works? (Spot-check key claims against code)
- Are there docs for subsystems that no longer exist?
- Are there subsystems in the codebase with no architecture doc?
- Do cross-references between docs point to valid files?

## Step 3: Contradiction check

Look for contradictions across documents:
- CLAUDE.md says X, but an ADR says Y
- An architecture doc describes pattern A, but CLAUDE.md mandates pattern B
- A rule in plan-workflow.md conflicts with a CLAUDE.md critical rule
- An ADR is marked "Implemented" but a later ADR supersedes it without updating the status

## Step 4: Wording audit

For each document, flag:
- **Verbose passages** — can this be said in fewer words without losing meaning?
- **Passive voice** — rewrite to active, direct instructions
- **Hedge words** — "should probably", "might want to", "consider" → make it definitive or remove it
- **Filler** — "It is important to note that..." → just state the thing
- **Redundancy** — same concept explained in multiple places → consolidate to one location, reference from others

## Step 5: Present findings

Print a structured report:

```
## Optimise-Docs Report — [DATE]

### Stale Content
| # | File | Line(s) | Issue | Action |
|---|------|---------|-------|--------|
| 1 | CLAUDE.md | 45 | Path `.docs/architecture/OLD.md` no longer exists | Remove reference |

### Contradictions
| # | File A | File B | Conflict | Resolution |
|---|--------|--------|----------|------------|

### Wording Improvements
| # | File | Line(s) | Current | Proposed |
|---|------|---------|---------|----------|

### Promotions Due (from implementation log)
| # | Log Entry Date | Decision | Should Be | Action |
|---|---------------|----------|-----------|--------|

### ADR Status Updates
| # | ADR | Current Status | Should Be | Reason |
|---|-----|---------------|-----------|--------|

### Missing Docs
| # | Subsystem/Area | Suggested Doc | Why |
|---|---------------|--------------|-----|

### Summary
- Stale items: N
- Contradictions: N
- Wording fixes: N
- Promotions due: N
- ADR updates: N
- Missing docs: N
```

## Step 6: Get approval

Present the report and ask the user:
"Review the findings above. Reply with:
- **apply all** — fix everything
- **apply [numbers]** — fix specific items (e.g., 'apply stale 1,3 + wording 2,5')
- **skip** — report only, no changes"

Do NOT make any changes until the user confirms.

## Step 7: Apply changes

For each approved change:
1. Make the edit
2. If an ADR status changes, update it in DECISIONS.md
3. If a log entry gets promoted, write the ADR/rule update AND leave the log entry in place
4. If a file reference is removed, check if other docs reference the same file

After all edits:
- Run quality gate commands (typecheck + lint + test + build) to verify nothing broke
- If any wording changes touched CLAUDE.md or plan-workflow.md, re-read both to confirm
  they're still internally consistent

## Step 8: Commit

```
git add [all modified files]
git commit -m "docs: optimise-docs — [N] stale removals, [N] wording fixes, [N] promotions"
```

## Step 9: Append to implementation log

```
### [DATE] — Documentation Optimisation
**Phase**: Maintenance
**Files Changed**: [list all modified docs]
**What**: Ran /optimise-docs — removed stale references, tightened wording, resolved contradictions, promoted unpromoted decisions
**Why**: Periodic documentation hygiene
**Decisions**: [any significant choices made during optimisation]
**Issues**: None
**Verification**: typecheck/lint/test/build pass
```
