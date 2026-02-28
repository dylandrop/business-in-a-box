# business-in-a-box

Reusable [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash commands for taking a codebase from prototype to production.

## Getting Started

1. Drop a business description into your repo (e.g. in `CLAUDE.md` or any file Claude can read) describing what your product does and who it's for
2. Run `/project:go-live-checklist` — it identifies what's missing and works through decisions with you
3. Run `/project:production-audit` — it picks up the checklist and iteratively implements, assesses, and refines until production-ready

That's it. The two commands form a pipeline:

```
go-live-checklist → production-audit
```

You can also run either command standalone.

## Commands

### `/project:go-live-checklist`

Go-live readiness assessment. Evaluates whether your project is ready to launch as a production business:

1. **Assess** — 9 parallel agents scan the codebase for: billing, analytics, testing, error handling, feedback funnels, auth, security, infrastructure, and CI/CD
2. **Recommend** — compiles findings into external service needs, major business decisions, and minor items, then works through each decision with you interactively
3. **Checklist** — generates a prioritized go-live checklist at `.claude/go-live/GO-LIVE-CHECKLIST.md`

Progress checkpointed to `.claude/go-live/state.json`.

#### Runtime output

```
.claude/go-live/
  state.json                  # durable state — resume from any phase
  assessments/                # per-area assessment files (one per agent)
  recommendations.md          # compiled recommendations
  decisions.md                # user decisions log
  GO-LIVE-CHECKLIST.md        # final prioritized checklist (consumed by production-audit)
```

### `/project:production-audit`

Automated production-readiness audit loop. Drop into any repo, run the command, and it will:

1. **Discover** all user-facing and admin/operational workflows in the codebase — if a go-live checklist exists, its items are incorporated as workflows to implement
2. **Assess** every workflow in parallel across 3 lenses — UX, Security, Performance — with platform-specific criteria for web, iOS, and Android
3. **Split** findings into conflict-free work packages (no shared files between splits)
4. **Fix** all Critical/High/Medium findings via parallel agents in isolated git worktrees
5. **Loop** — re-assess, fix new issues, repeat until clean (default 3 iterations, configurable)

Pass an optional iteration count: `/project:production-audit 5`

All progress is checkpointed to `.claude/audit/state.json`, so the command survives context flushes, token limits, and session restarts.

#### Design principles

- **Cross-platform** — supports web, iOS (Swift/SwiftUI/UIKit), Android (Kotlin/Compose/XML), React Native, and Flutter
- **Repository-agnostic** — discovers project structure, test runners, and build systems dynamically
- **Context-resilient** — state machine on disk, phase-specific instruction loading, one-line agent returns
- **Token-optimized** — slim 45-line dispatcher, JIT-loaded phase instructions, findings never enter orchestrator context
- **Parallel-safe** — file-disjoint splits, worktree isolation, port/DB isolation per agent, post-merge integration verification

#### File layout

```
.claude/commands/
  production-audit.md                              # slim dispatcher + state machine
  go-live-checklist.md                             # go-live readiness assessment
  audit-templates/
    orchestrator-discover-and-assess.md            # phases 1-3 instructions (JIT-loaded)
    orchestrator-execute-and-merge.md              # phases 4-7 instructions (JIT-loaded)
    discover-workflows.md                          # agent: workflow mapping (user + admin)
    assess-ux.md                                   # agent: UX audit checklist (web + mobile)
    assess-security.md                             # agent: security audit (OWASP Top 10 + Mobile Top 10)
    assess-performance.md                          # agent: performance audit (web + mobile)
    execute-split.md                               # agent: implement fixes in worktree
```

#### Runtime output

```
.claude/audit/
  state.json                  # durable state — resume from any phase
  workflows.md                # discovered workflows (user-facing + admin/operational)
  findings/iter-{N}/          # assessment findings, versioned per iteration
  splits/iter-{N}/            # work splits + execution results
  REPORT.md                   # final audit report
```

## Installation

Copy the `.claude/commands/` directory into any repo:

```bash
# From inside your target repo
git clone https://github.com/dylandrop/business-in-a-box.git /tmp/biab
cp -r /tmp/biab/.claude/commands/ .claude/commands/
rm -rf /tmp/biab
```

Or add as a git subtree:

```bash
git subtree add --prefix=.claude/commands https://github.com/dylandrop/business-in-a-box.git main --squash
```

Then run in Claude Code:

```
/project:go-live-checklist
/project:production-audit
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Git (for worktree-based parallel execution)

## License

MIT
