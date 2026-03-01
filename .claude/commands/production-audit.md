# Production Readiness Audit

Automated audit loop: discover workflows → assess (UX/security/performance) → fix all Critical/High/Medium findings via parallel worktree agents → repeat until clean.

Resilient to context flushes via `.claude/audit/state.json`. Agent templates at `~/.claude/commands/audit-templates/` (global) or `.claude/commands/audit-templates/` (project-local). Check project-local first, fall back to global.

## Parameters

- `$ARGUMENTS` — optional: max number of audit iterations (default: 3). Example: `/project:production-audit 5`

## Initialize

Run: `mkdir -p .claude/audit/findings .claude/audit/splits`

If `.claude/audit/state.json` exists, read it and jump to the current `phase`.
Otherwise: parse `$ARGUMENTS` as an integer for `maxIterations` (default `3` if empty or non-numeric). Resolve `templatesPath`: check if `.claude/commands/audit-templates/` exists in the project directory — if yes, use `.claude/commands/audit-templates`; otherwise use `~/.claude/commands/audit-templates`. Write: `{"iteration":1,"phase":"discovery","maxIterations":<parsed>,"templatesPath":"<resolved>","workflowCount":0,"assessment":{"ux":"pending","security":"pending","performance":"pending"},"openFindings":{"critical":0,"high":0,"medium":0},"execution":{"totalSplits":0,"launched":[],"completed":[],"branches":{},"merged":[]},"errors":[]}`

## Phase Routing

Read `state.phase`. Load the instructions file for the current phase — do NOT read files for other phases.

| Phase | Read This File | Section to Follow |
|-------|---------------|-------------------|
| `discovery` | `{TEMPLATES}/orchestrator-discover-and-assess.md` | §Discovery |
| `assessment` | `{TEMPLATES}/orchestrator-discover-and-assess.md` | §Assessment |
| `splitting` | `{TEMPLATES}/orchestrator-discover-and-assess.md` | §Splitting |
| `execution` | `{TEMPLATES}/orchestrator-execute-and-merge.md` | §Execution |
| `merging` | `{TEMPLATES}/orchestrator-execute-and-merge.md` | §Merging |
| `loop-check` | `{TEMPLATES}/orchestrator-execute-and-merge.md` | §Loop Check |
| `report` | `{TEMPLATES}/orchestrator-execute-and-merge.md` | §Report |

**Resolve `{TEMPLATES}`**: If `.claude/commands/audit-templates/` exists in the project, use that. Otherwise use `~/.claude/commands/audit-templates/`. Store the resolved path in `state.templatesPath` on first run so agents use a consistent location.

Template paths: check `.claude/commands/` first (project-local), then `~/.claude/commands/` (global). Use whichever exists.

## Phase Flow

```
discovery → assessment → splitting → execution → merging → loop-check
                ↑                                              |
                └──────────── if C/H/M remain ─────────────────┘
                                    ↓ (none remain or iteration >= max)
                                  report → complete
```

## Invariants

- Checkpoint `state.json` after every phase transition.
- Findings versioned: `.claude/audit/findings/iter-{N}/`
- Splits namespaced: `.claude/audit/splits/iter-{N}/`
- No two splits share files (parallel-safe).
- Max iterations controlled by `state.maxIterations` (default 3, overridable via parameter).
- All agents return ONE-LINE summaries — details are on disk.
