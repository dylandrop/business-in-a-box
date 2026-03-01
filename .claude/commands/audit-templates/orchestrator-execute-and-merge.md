# Orchestrator: Execution, Merging, Loop Check, Report

Detailed phase instructions. Read ONLY when state.phase is "execution", "merging", "loop-check", or "report".

---

## Execution

Read `state.iteration` → `I` and `state.execution`.

**Resume logic**: Only launch agents for splits NOT in `execution.completed`.

For each pending split N, launch Agent (`subagent_type: "general-purpose"`, `isolation: "worktree"`) in parallel:

> Software engineer. Iteration: {I}, Split: {N}. Read instructions at `{state.templatesPath}/execute-split.md`. Assignment at `.claude/audit/splits/iter-{I}/split-{N}.md`. Write result to `.claude/audit/splits/iter-{I}/split-{N}-result.md`.

As each completes:
1. Read `split-{N}-result.md` for branch name.
2. Update state: add N to `execution.completed`, record `execution.branches[N] = {branch, worktreePath}`.

On failure: record in `state.errors`, retry once. Skip on second failure.
After all done: set `state.phase = "merging"`.

---

## Merging

Read `state.execution.branches` and `state.execution.merged`.

### Recovery (if branch info lost after context flush)
If `branches` is empty but `completed` is not: scan `split-*-result.md` files for branch names. Fallback: `git worktree list` and `git branch --list 'audit-*'`.

### Sequential merge
For each unmerged branch:
1. Check already merged: `git merge-base --is-ancestor {branch} HEAD` → skip if yes.
2. Merge: `git merge {branch} --no-edit`
3. On conflict: record in `state.errors`, ask user.
4. Cleanup: `git worktree remove {path} 2>/dev/null; git branch -d {branch} 2>/dev/null`
5. Checkpoint: add to `state.execution.merged` immediately.

### Post-merge verification
Run full build + full test suite to catch cross-split interface breakage. Fix and commit if needed: `audit: iteration {I} - post-merge integration fixes`

### Deferred dependencies
Check `split-*-result.md` files for deferred packages. If any: install all at once, rebuild, retest, commit: `audit: iteration {I} - add deferred dependencies`

### Commit state
`git add .claude/audit/ && git commit -m "audit: complete iteration {I}"`
Set `state.phase = "loop-check"`.

---

## Loop Check

Read `state.iteration` → `I`.

1. If `I >= state.maxIterations`: set `state.phase = "report"`. Go to §Report.
2. Else: set `state.iteration = I + 1`, reset `state.assessment` to all `"pending"`, clear `state.execution` and `state.openFindings`, set `state.phase = "assessment"`.
3. Go back to the assessment phase (read `orchestrator-discover-and-assess.md` §Assessment).

---

## Report

Read all findings files across all iterations AND scan the codebase for last-mile setup items that require human action. Write `.claude/audit/REPORT.md`:

```markdown
# Production Readiness Audit Report
_Completed: {date} | Iterations: {N}_

## Executive Summary
{2-3 sentences}

## Findings Summary
| Category    | Critical | High | Medium | Low | Total |
|-------------|----------|------|--------|-----|-------|
| UX          |          |      |        |     |       |
| Security    |          |      |        |     |       |
| Performance |          |      |        |     |       |

## Resolved Findings
{ID, title, iteration fixed}

## Remaining Open Findings
{ID, title, why not addressed}

## Manual Steps Required

Things Claude cannot do — the human must complete these before go-live. Compile this section by scanning the codebase for references to external services, placeholder secrets, and unfinished configuration. Organize into the subsections below. Only include subsections that are relevant to this project.

### Secrets & Environment Variables
For each `.env.example`, `.env.local.example`, or environment variable reference found in the codebase:
- Variable name and which service it belongs to
- Where to obtain the value (e.g., "Stripe Dashboard → Developers → API Keys")
- Whether it's needed for development, staging, production, or all
- Flag any secrets that appear to be hardcoded, use placeholder values (e.g., `sk_test_xxx`, `your-api-key-here`, `CHANGEME`), or are committed to the repo

### External Service Signup & Configuration
For each third-party service the codebase integrates with (auth providers, payment processors, email services, analytics, error tracking, cloud hosting, CDNs, etc.):
- Service name and what it's used for in the app
- Account setup steps (sign up, create project/app, configure settings)
- Specific configuration required (webhook URLs to register, callback/redirect URIs to whitelist, domain verification, DNS records)
- API keys or credentials to generate and where to put them
- Webhook secrets and endpoints to configure (list the exact route paths from the codebase)

### OAuth & Authentication Provider Setup
- OAuth app registration (Google, GitHub, Apple, etc.) — redirect URIs to configure, scopes to request
- Auth provider dashboard settings (Clerk, Auth0, Firebase Auth, etc.) — allowed origins, sign-in methods to enable, email templates to customize
- Social login button configuration that exists in the UI but requires provider-side setup

### Payment & Billing Setup
- Payment provider onboarding (Stripe account activation, business verification)
- Product/price/plan creation in the provider dashboard matching what the codebase expects (list specific product IDs, price IDs, or lookup keys referenced in the code)
- Webhook endpoint registration with the provider (list exact endpoint paths)
- Tax configuration, invoicing settings, customer portal setup

### Domain, DNS & SSL
- Domain registration and DNS records to create (A, CNAME, TXT, MX records)
- SSL certificate provisioning (or confirmation that hosting provider handles it)
- Email sending domain verification (SPF, DKIM, DMARC records for services like Resend, SendGrid, SES)

### App Store & Play Store Submission
If the project targets iOS or Android:
- Apple Developer Program enrollment ($99/year) and App Store Connect app creation
- Google Play Developer account enrollment ($25 one-time) and Play Console app creation
- Code signing setup (iOS certificates & provisioning profiles, Android upload key or Play App Signing enrollment)
- Store listing content to prepare (descriptions, screenshots, privacy policy URL, support URL)
- Review submission and expected timeline

### CI/CD & Infrastructure
- CI/CD pipeline secrets to configure (in GitHub Actions, GitLab CI, etc.) — list each secret name and where the value comes from
- Hosting provider account setup and project linking
- Database provisioning (hosted instance, connection string, initial migrations)
- File storage / CDN bucket creation and configuration
- Monitoring / alerting service setup and notification channels

### Post-Launch Verification
- Smoke test checklist: critical user flows to manually verify in production after deploy
- Monitoring dashboards to set up (error rates, response times, key business metrics)
- Alerting rules to configure (error spike thresholds, downtime, billing failures)
- Backup and disaster recovery verification

## Errors Encountered
{from state.errors}

## Files Modified
{from git log}
```

Set `state.phase = "complete"`. Present the report to the user.
