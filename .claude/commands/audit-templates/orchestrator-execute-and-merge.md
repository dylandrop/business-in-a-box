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

Read all findings files across all iterations AND scan the codebase for last-mile setup items that require human action. Also check for user decisions and environment context: read `.claude/go-live/decisions.md` and `.claude/go-live/GO-LIVE-CHECKLIST.md` if they exist — these contain user choices about hosting providers, services, environment strategy (staging/test/production), and other infrastructure decisions. The Manual Steps section must respect these decisions: only reference environments the user has chosen to use, only recommend services the user selected, and tailor setup instructions to match their chosen providers and architecture. Write `.claude/audit/REPORT.md`:

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

Things Claude cannot do — the human must complete these before go-live. Compile this section by scanning the codebase for references to external services, placeholder secrets, and unfinished configuration. If `.claude/go-live/decisions.md` exists, use it to tailor this section: only reference environments the user chose (e.g., if they decided on production-only with no staging, don't list staging setup steps; if they chose a specific hosting provider, give instructions for that provider, not generic ones). Organize into the subsections below. Only include subsections that are relevant to this project.

### Secrets & Environment Variables
For each `.env.example`, `.env.local.example`, or environment variable reference found in the codebase:
- Variable name and which service it belongs to
- Where to obtain the value (e.g., "Stripe Dashboard → Developers → API Keys")
- Which environment(s) it's needed for — only list environments the user has chosen to set up (check decisions.md). If the user chose production-only, don't list staging/dev values. If they have staging + production, list both with notes on which values differ per environment (e.g., test vs live API keys)
- Flag any secrets that appear to be hardcoded, use placeholder values (e.g., `sk_test_xxx`, `your-api-key-here`, `CHANGEME`), or are committed to the repo

### External Service Signup & Configuration
For each third-party service the codebase integrates with (auth providers, payment processors, email services, analytics, error tracking, cloud hosting, CDNs, etc.):
- Service name and what it's used for in the app
- Account setup steps (sign up, create project/app, configure settings)
- Specific configuration required (webhook URLs to register, callback/redirect URIs to whitelist, domain verification, DNS records) — if the user has multiple environments, note which webhook URLs / redirect URIs need to be registered per environment
- API keys or credentials to generate and where to put them
- Webhook secrets and endpoints to configure (list the exact route paths from the codebase)
- If the user chose a specific provider in decisions.md, give instructions for that provider — do not list alternatives they rejected

### OAuth & Authentication Provider Setup
- OAuth app registration (Google, GitHub, Apple, etc.) — redirect URIs to configure per environment (e.g., `localhost` for dev, staging domain, production domain), scopes to request
- Auth provider dashboard settings (Clerk, Auth0, Firebase Auth, etc.) — allowed origins per environment, sign-in methods to enable, email templates to customize
- Social login button configuration that exists in the UI but requires provider-side setup

### Payment & Billing Setup
- Payment provider onboarding (Stripe account activation, business verification)
- Product/price/plan creation in the provider dashboard matching what the codebase expects (list specific product IDs, price IDs, or lookup keys referenced in the code) — note if test-mode products are needed for staging/dev environments
- Webhook endpoint registration with the provider per environment (list exact endpoint paths and the environment-specific base URLs)
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
- If the user has a staging/beta track: TestFlight setup (iOS) and internal/closed testing track (Android) with tester groups
- Review submission and expected timeline

### CI/CD & Infrastructure
- CI/CD pipeline secrets to configure (in GitHub Actions, GitLab CI, etc.) — list each secret name, where the value comes from, and which environment(s) it applies to. If the user has separate staging/production pipelines, organize secrets per environment
- Hosting provider account setup and project linking — per environment (e.g., separate Vercel projects for staging and production, or separate Railway environments). Use the provider the user chose in decisions.md
- Database provisioning per environment (hosted instance, connection string, initial migrations) — note if staging and production need separate database instances
- File storage / CDN bucket creation and configuration — per environment if applicable
- Monitoring / alerting service setup and notification channels

### Post-Launch Verification
- Smoke test checklist: critical user flows to manually verify in each environment after deploy (staging first if applicable, then production)
- Monitoring dashboards to set up (error rates, response times, key business metrics)
- Alerting rules to configure (error spike thresholds, downtime, billing failures) — different thresholds may be appropriate for staging vs production
- Backup and disaster recovery verification

### Setup Automation Script

Generate a shell script at `.claude/audit/setup-automation.sh` that automates as much of the manual configuration above as possible. The script should:

- Be idempotent (safe to run multiple times without breaking things)
- Check for required CLI tools before running (e.g., `stripe`, `gh`, `vercel`, `fly`, `railway`, `netlify`, `aws`, `gcloud`, `firebase`, etc.) and print install instructions for any that are missing
- Use the CLIs for external services where available to automate setup:
  - **Stripe**: create products/prices matching code references via `stripe products create` / `stripe prices create`, register webhook endpoints via `stripe webhook_endpoints create`, output the webhook signing secret
  - **Vercel/Netlify/Railway/Fly/etc.**: link project, set environment variables via CLI (e.g., `vercel env add`, `railway variables set`)
  - **GitHub Actions**: set repository secrets via `gh secret set`
  - **Clerk/Auth0/Firebase Auth**: use their CLI or API to configure allowed origins, redirect URIs where possible
  - **Resend/SendGrid/SES**: use CLI or API to verify sending domains, create API keys
  - **Database**: run migrations, seed initial data if seed scripts exist
  - **DNS**: print the exact records needed (these usually can't be automated, but the script should output them clearly)
- Prompt the user interactively for secrets and API keys it cannot generate itself, then write them to `.env` files and hosting provider environment configs
- Separate setup steps by environment (production, staging, etc.) based on the user's decisions in `decisions.md`
- Log every action it takes and skip steps that are already completed (detect existing config)
- End with a summary of what was configured, what was skipped, and what still needs manual action

Include the script in the report as a collapsible code block and also write it to disk:

<details>
<summary>Setup Automation Script (.claude/audit/setup-automation.sh)</summary>

```bash
{generated script content}
```
</details>

> Run with: `chmod +x .claude/audit/setup-automation.sh && .claude/audit/setup-automation.sh`

### Claude Automation Prompt

Also include a ready-to-use Claude prompt that the user can paste into a new Claude Code session to have Claude interactively walk through and automate the remaining manual setup. The prompt should be tailored to this specific project — reference the actual services, environment variables, providers, and environments discovered during the audit. Write it as a fenced block the user can copy:

> **Copy the prompt below into a new Claude Code session to get interactive help completing the remaining setup:**

~~~
You are helping me complete the production setup for my application. Here is what needs to be configured:

{List each manual step category that applies, with specifics from the audit:}

**Environment Variables Needed:**
{List each variable name, which service it's for, and where to get the value}

**External Services to Configure:**
{List each service, what needs to be set up, and the specific webhook URLs / redirect URIs / config values from the codebase}

**Infrastructure:**
{Hosting provider, database, CI/CD — per environment}

Please walk me through each step interactively:
1. Ask me for credentials/API keys one service at a time
2. Use CLI tools where available to automate the configuration (e.g., stripe CLI, vercel CLI, gh CLI)
3. Write values to the appropriate .env files and set them on hosting providers
4. Verify each step works before moving to the next
5. At the end, run a verification check to confirm everything is connected

Start with the most critical items first (secrets and environment variables), then move to external service configuration, then infrastructure.
~~~

## Errors Encountered
{from state.errors}

## Files Modified
{from git log}
```

Set `state.phase = "complete"`. Present the report to the user.
