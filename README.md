# business-in-a-box

Reusable [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash commands for taking a codebase from prototype to production.

## Commands

### `/project:production-audit`

Automated production-readiness audit loop. Drop into any repo, run the command, and it will:

1. **Discover** all user workflows in the codebase
2. **Assess** every workflow in parallel across 3 lenses — UX, Security, Performance
3. **Split** findings into conflict-free work packages (no shared files between splits)
4. **Fix** all Critical/High/Medium findings via parallel agents in isolated git worktrees
5. **Loop** — re-assess, fix new issues, repeat until clean (max 3 iterations)

All progress is checkpointed to `.claude/audit/state.json`, so the command survives context flushes, token limits, and session restarts.

#### Design principles

- **Repository-agnostic** — discovers project structure, test runners, and build systems dynamically
- **Context-resilient** — state machine on disk, phase-specific instruction loading, one-line agent returns
- **Token-optimized** — slim 45-line dispatcher, JIT-loaded phase instructions, findings never enter orchestrator context
- **Parallel-safe** — file-disjoint splits, worktree isolation, port/DB isolation per agent, post-merge integration verification

#### File layout

```
.claude/commands/
  production-audit.md                              # slim dispatcher + state machine
  audit-templates/
    orchestrator-discover-and-assess.md            # phases 1-3 instructions (JIT-loaded)
    orchestrator-execute-and-merge.md              # phases 4-7 instructions (JIT-loaded)
    discover-workflows.md                          # agent: workflow mapping
    assess-ux.md                                   # agent: UX audit checklist
    assess-security.md                             # agent: security audit (OWASP Top 10)
    assess-performance.md                          # agent: performance audit
    execute-split.md                               # agent: implement fixes in worktree
```

#### Runtime output

```
.claude/audit/
  state.json                  # durable state — resume from any phase
  workflows.md                # discovered user workflows
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
/project:production-audit
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Git (for worktree-based parallel execution)

## License

MIT
