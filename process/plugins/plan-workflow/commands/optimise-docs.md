---
description: Analyse and optimise all project documentation — CLAUDE.md, rules, ADRs, logs, architecture docs
argument-hint: "[scope] — 'all' (default), 'claude-md', 'rules', 'adrs', 'logs', 'architecture', 'plans'"
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
5. All files in `.docs/logs/archive/` — archived implementation logs (read in chronological
   order to trace decision evolution — an early decision may have been superseded later)
6. All files in `.docs/architecture/` — subsystem specifications
7. `.docs/FEATURE_BACKLOG.md` — feature tracker
8. All active plans in `.docs/plans/` — plan status and forward exports

If `$ARGUMENTS` specifies a scope other than "all", only read and optimise the
targeted documents. Still read CLAUDE.md and DECISIONS.md for cross-reference context.

For large projects (10,000+ lines of documentation), use parallel agents to read
different document categories simultaneously.

## Step 2: Staleness audit

For each document, check:

### CLAUDE.md
- Are all file paths in "Architecture Reference" still valid? (Glob for each path)
- Are all "Critical Rules" still relevant? (Search codebase — is the rule's subject still in use?)
- Are tech stack entries accurate? (Check package.json, imports)
- Are "Common Commands" still correct? (Verify scripts exist in package.json)
- Do quality gate commands match what CI actually runs?
- Are there duplicate rules saying the same thing in different words?
- Are detailed rules that belong in architecture docs cluttering CLAUDE.md?
  (General principles stay in CLAUDE.md; detailed implementation rules should point
  to the relevant architecture doc with "See [doc] for details")

### .claude/rules/plan-workflow.md
- Do "Seeded Principles" reference phases/features that still exist?
- Are anti-patterns in §16/§17/§18 still relevant? (Is the technology/pattern still in use?)
- Does "Which Module to Read" (§6) list files that exist? (Glob for each path)
- Are there anti-patterns that duplicate CLAUDE.md rules? (Cross-reference)
- Is the `<!-- Last Updated -->` header current?

### DECISIONS.md
- Are any ADRs marked "Implemented" but the code has since been removed or replaced?
- Are any ADRs superseded by later ADRs but not marked "Superseded"?
- **Decision evolution**: trace each ADR through the full log timeline (including archives).
  Was the approach later changed? If a log entry says "we tried X from ADR-NNN but
  switched to Y", the ADR should be marked "Superseded" and a new ADR written.
- Supersession chains: if ADR-B says "supersedes ADR-A", verify ADR-A status is updated
- ADRs that contradict each other (same topic, different decisions, both "Implemented")
- ADR numbering gaps (note, not error)
- Do any ADRs reference files/components that no longer exist?
- ADRs missing required fields (Context, Decision, Rationale, Status)
- **Unpromoted decisions**: log entries with "Decisions" sections that were never promoted
  to ADRs. Apply Knowledge Classification §1 — if the decision is still relevant, flag it.

### IMPLEMENTATION_LOG.md (current + archived)
- Are there entries older than the last major phase boundary that should be archived?
- Are there entries with "Decisions" sections that were never promoted to ADRs?
  (Apply Knowledge Classification §1 — if the decision is still relevant, promote it now)
- Are there duplicate entries describing the same change?

### Architecture docs (.docs/architecture/*.md)
- Does each doc describe the system as it currently works?
  **Verify thoroughly against the codebase** — not just spot-checks:
  - Schema docs: verify tables/columns match actual schema files
  - API docs: verify endpoints match actual function/route files
  - Component docs: verify structures match actual component tree
  - Auth docs: verify flows match actual middleware/auth code
  - Flag significant codebase patterns NOT documented in any architecture doc
- Are there docs for subsystems that no longer exist?
- Are there subsystems in the codebase with no architecture doc?
- Do cross-references between docs point to valid files?
- Recent ADRs (last 3 months): is the relevant architecture module updated to reflect them?

### Plans + Backlog
- Plan Status field vs FEATURE_BACKLOG.md row status — flag mismatches
- Every `[plan](...)` link in backlog — verify file exists
- Plans marked "Done" — should they be archived to `_archive/`?
- Plans marked "In Progress" — verify a branch exists (or recently merged)
- Plans with unresolved "Open Questions" but status is "Approved"/"In Progress"
- Forward export `@public` tags: Grep for `@public Forward export`. If consuming
  phase is "Done", tag is stale.

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
- **Hedge words** — "should probably", "might want to", "consider" — make it definitive or remove it
- **Filler** — "It is important to note that..." — just state the thing
- **Redundancy** — same concept explained in multiple places — consolidate to one location, reference from others

## Step 5: Validate findings

**CRITICAL — before presenting findings, verify every claim:**

For each finding that asserts "file X doesn't exist" or "code Y was removed" or
"ADR-N contradicts ADR-M":
- **Actually check** — Glob/Grep for the file. Read the ADR text. Verify the claim.
- Suppress false positives. Flag ambiguous items as "needs manual review".

For each proposed change (especially architecture doc updates, CLAUDE.md rule removals,
ADR status changes):
- Would this change remove information that's still needed?
- Would moving a CLAUDE.md rule to an architecture doc cause it to be missed?
  (Architecture docs aren't auto-loaded in every session)
- Is the proposed wording actually better, or just different?

Mark risky proposals with a warning flag in the report.

## Step 6: Present findings

Print a structured report:

```
## Optimise-Docs Report — [DATE]

### Stale Content
| # | File | Line(s) | Issue | Action | Status |
|---|------|---------|-------|--------|--------|
| 1 | CLAUDE.md | 45 | Path `.docs/architecture/OLD.md` no longer exists | Remove reference | confirmed |

### Contradictions
| # | File A | File B | Conflict | Resolution | Status |
|---|--------|--------|----------|------------|--------|

### Wording Improvements
| # | File | Line(s) | Current | Proposed | Status |
|---|------|---------|---------|----------|--------|

### Promotions Due (from implementation log)
| # | Log Entry Date | Decision | Should Be | Action | Status |
|---|---------------|----------|-----------|--------|--------|

### ADR Status Updates
| # | ADR | Current Status | Should Be | Reason | Status |
|---|-----|---------------|-----------|--------|--------|

### Plan/Backlog Fixes
| # | File | Issue | Proposed Fix | Status |
|---|------|-------|-------------|--------|

### Missing Docs
| # | Subsystem/Area | Suggested Doc | Why |
|---|---------------|--------------|-----|

### Needs Manual Review
| # | Source | Finding | Why Manual Review Needed |
|---|--------|---------|------------------------|

### Summary
- Stale items: N
- Contradictions: N
- Wording fixes: N
- Promotions due: N
- ADR updates: N
- Plan/backlog fixes: N
- Missing docs: N
- Needs manual review: N
```

## Step 7: Get approval

Present the report and ask the user:
"Review the findings above. Reply with:
- **apply all** — fix everything
- **apply [numbers]** — fix specific items (e.g., 'apply stale 1,3 + wording 2,5')
- **skip** — report only, no changes"

Do NOT make any changes until the user confirms.

## Step 8: Apply changes

For each approved change:
1. Make the edit
2. If an ADR status changes, update it in DECISIONS.md
3. If a log entry gets promoted, write the ADR/rule update AND leave the log entry in place
4. If a file reference is removed, check if other docs reference the same file

After all edits:
- Run quality gate commands (typecheck + lint + test + build) to verify nothing broke
- If any wording changes touched CLAUDE.md or plan-workflow.md, re-read both to confirm
  they're still internally consistent

## Step 9: Commit

```
git add [all modified files]
git commit -m "docs: optimise-docs — [N] stale removals, [N] wording fixes, [N] promotions"
```

## Step 10: Append to implementation log

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
