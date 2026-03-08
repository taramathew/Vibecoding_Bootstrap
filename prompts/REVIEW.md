# Bootstrap Review Prompt

Paste this into a Claude Code session (inside the Vibecoding_Bootstrap repo directory) to critically review and update the bootstrap package.

Run this periodically — after completing a major project phase, after discovering new anti-patterns, or when adopting new tools/practices.

---

## The Prompt

```
You are reviewing and updating a Vibecoding Bootstrap package — a reusable development process toolkit for AI-assisted (vibe coding) projects using Claude Code.

Read ALL files in this repository thoroughly:
1. README.md — what this package is
2. prompts/BOOTSTRAP.md — the setup prompt for new projects
3. prompts/REVIEW.md — this review prompt (meta-review it too)
4. templates/ — CLAUDE.md template, log templates, design doc template, backlog template
5. process/rules/plan-workflow.md — the core process rules (18 sections)
6. process/plugins/plan-workflow/commands/create-plan.md
7. process/plugins/plan-workflow/commands/create-prompts.md
8. process/plugins/plan-workflow/commands/optimise-docs.md
9. process/plugins/code-review/commands/code-review.md
10. process/plugins/.claude-plugin/marketplace.json

YOUR ROLE: You are an independent process architect and developer experience expert. Your job is to find gaps, outdated patterns, missing best practices, and opportunities to improve. Do NOT rubber-stamp the current state.

REVIEW EACH FILE AGAINST THESE CRITERIA:

## 1. Currency & Best Practices
- Are the anti-patterns (§16, §17, §18) still relevant? Are any outdated?
- Are there NEW common anti-patterns from recent Claude Code / AI coding experience that should be added?
- Do the quality gates reflect current best practices (dead code detection, type checking, linting, testing, building)?
- Is the error handling philosophy still sound?
- Are there new Claude Code features (tools, hooks, MCP servers, slash commands, agent SDK patterns) that the process should leverage?

## 2. Vibecoding Workflow
- Is the Backlog → Plan → Prompt → Execute → Review → Learn cycle optimal?
- Are there missing steps or unnecessary steps?
- Does the cross-prompt context system (§10) adequately handle stateless sessions?
- Is the Phase Closure process (§9) efficient or over-engineered?
- Is the Knowledge Classification system (§1) routing learnings to the right destinations?

## 3. Plugin Quality
- Does /create-plan ask the right research questions? Missing any?
- Does /create-prompts produce truly self-contained prompts?
- Does /code-review have the right balance of thoroughness vs false positives?
- Does /optimise-docs cover the right audit areas? Missing any?
- Are the agent model assignments (haiku/sonnet/opus) still optimal given current model capabilities?
- Is the confidence threshold (80%) right?

## 4. Template Quality
- Is the CLAUDE.md template comprehensive enough without being bloated?
- Does it cover: principles, architecture references, critical rules, documentation rules, development workflow, commands?
- Are the log templates (IMPLEMENTATION_LOG.md, DECISIONS.md) the right format?
- Is the design doc template useful as a starting point?
- Is the feature backlog template clear?

## 5. Bootstrap Prompt
- Does prompts/BOOTSTRAP.md make sense for a brand-new project?
- Does it handle both greenfield projects AND adding the process to existing projects?
- Are the customization instructions clear?
- Does it properly reference all process files without asking Claude to regenerate them?
- Are the template paths correct after restructuring?

## 6. Completeness
- Is anything missing that a new project would need on day 1?
- Are there common project types (monorepo, single-package, full-stack, backend-only, frontend-only) that the bootstrap doesn't handle well?
- Should there be a codebase-audit command? (The original had one but it was heavily project-specific)

## 7. Wording & Clarity
- Is every instruction actionable and unambiguous?
- Are there redundancies across files?
- Is the tone consistent (direct, no filler)?
- Can a developer unfamiliar with this system understand each file standalone?

FOR EACH FILE, PROVIDE:
- **Issues Found**: Specific problems with line references
- **Missing Content**: What should be added
- **Outdated Content**: What should be removed or updated
- **Suggested Changes**: Concrete edits (not vague recommendations)

THEN:
- Apply all changes directly to the files
- Update the README.md if the package structure changed
- Update the <!-- Last Updated --> comment in plan-workflow.md
- Commit with message: "chore: bootstrap review — [summary of changes]"

Be direct. Propose concrete edits. If something works well, say so briefly and move on.
```
