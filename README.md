# Vibecoding Bootstrap

A battle-tested development process bootstrap package for AI-assisted (vibe coding) projects using [Claude Code](https://claude.ai/code).

Clone this repo, run the bootstrap prompt in a new project directory, and get a fully configured development workflow with plans, prompts, code review, and knowledge management — out of the box.

## What's Inside

```
Vibecoding_Bootstrap/
  README.md                              # This file
  prompts/
    BOOTSTRAP.md                         # Paste into Claude Code to set up a new project
    REVIEW.md                            # Paste to review/update this bootstrap package
  templates/
    CLAUDE.md.template                   # Skeleton CLAUDE.md for new projects
    DESIGN_DOC.template.md               # Sample design doc structure (fill before bootstrap)
    FEATURE_BACKLOG.md                   # Feature backlog table (seeded into project)
    logs/
      IMPLEMENTATION_LOG.md              # Empty log with entry format
      DECISIONS.md                       # Empty decisions log with ADR format
  process/
    rules/
      plan-workflow.md                   # 18-section process rules (plans, prompts, quality gates, anti-patterns)
    plugins/
      .claude-plugin/
        marketplace.json                 # Plugin manifest
      plan-workflow/
        commands/
          create-plan.md                 # /create-plan — structured implementation plans
          create-prompts.md              # /create-prompts — executable prompts from plans
          optimise-docs.md               # /optimise-docs — clean stale refs, tighten wording, promote decisions
      code-review/
        commands/
          code-review.md                 # /code-review — multi-agent PR review
```

## Quick Start

```bash
# 1. Clone the bootstrap repo
git clone https://github.com/taramathew/Vibecoding_Bootstrap.git

# 2. Go to your new project directory (empty or existing)
cd ~/projects/my-new-project

# 3. Open Claude Code
claude

# 4. Paste the contents of prompts/BOOTSTRAP.md
#    Claude will:
#    - Read your design doc (you provide the path)
#    - Copy process files into .claude/
#    - Seed templates into .docs/
#    - Initialize the project (git, package manager, etc.)
```

## What You Get

### Slash Commands
- `/create-plan [feature]` — Scans codebase, asks clarifying questions, produces a structured plan
- `/create-prompts [plan-name]` — Converts plan steps into self-contained prompts for fresh sessions
- `/code-review --comment` — Multi-agent PR review with confidence-based scoring (posts inline GitHub comments)
- `/optimise-docs [scope]` — Audit all docs for stale refs, contradictions, verbose wording, unpromoted decisions — then fix

### Development Workflow
```
Backlog → Plan → Prompt → Execute → Review → Learn
                                        ↓
                               Rules Feedback Loop
                          (anti-patterns, seeded principles)
```

### Knowledge Management
Every learning is classified and routed:
- **Universal constraints** → CLAUDE.md
- **Process patterns** → plan-workflow.md (self-improving rules)
- **Architectural decisions** → DECISIONS.md (ADR format)
- **Subsystem changes** → architecture docs
- **Everything else** → implementation log

### Quality Gates
- Dead code detection before merge
- Full quality pass (typecheck + lint + test + build) at every prompt boundary
- Phase Closure with knowledge promotion

## After Bootstrap

The bootstrap repo is **not a runtime dependency**. After setup, your project has its own copy of all process files in `.claude/`. You can:
- **Delete the clone** — the project is self-contained
- **Keep it** — to run `prompts/REVIEW.md` later or bootstrap another project

## Keeping Up to Date

Paste `prompts/REVIEW.md` into a Claude Code session (inside the bootstrap repo) periodically to:
- Review the bootstrap files against latest best practices
- Update anti-patterns from your project's learnings
- Improve process rules based on real implementation experience

## Philosophy

This package encodes hard-won lessons from real projects:
- **Sessions are stateless** — every cross-prompt dependency is explicitly wired
- **Plans bridge phases** — lessons learned, forward exports, deferred items are tracked
- **Rules self-improve** — each Phase Closure can refine the rules for future plans
- **High signal only** — code review reports only confidence >= 80% findings
- **Process, not prescription** — quality gate commands, merge strategy, and architecture docs are customizable per project
