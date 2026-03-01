# Go-Live Readiness Assessment

Assess whether the current project is ready to launch as a production business. Identify gaps, work through major decisions with the user, and produce a comprehensive go-live checklist.

## Initialize

Run: `mkdir -p .claude/go-live/assessments`

If `.claude/go-live/state.json` exists, read it and jump to the current `phase`.
Otherwise, write: `{"phase":"assessment","assessments":{"auth-security":"pending","testing-errors":"pending","billing-analytics":"pending","infrastructure-cicd":"pending","feedback":"pending"},"errors":[]}`

## Phase 1: Parallel Assessment

Launch agents in parallel — each writes to its OWN file (no overwrites). Only launch agents for areas where `state.assessments.{area}` is `"pending"`.

For each pending area, launch Agent (`subagent_type: "general-purpose"`, `model: "haiku"`) with the prompt below, substituting `{AREA}` and `{DESCRIPTION}`:

### Agent prompts

**auth-security** →
> Assess go-live readiness for AUTHENTICATION, AUTHORIZATION & SECURITY. Scan the codebase for: auth provider (Clerk, Auth0, NextAuth, custom), login/signup/password-reset flows, MFA, OAuth, role-based access control, session management, token handling, HTTPS enforcement, CORS config, CSRF protection, input validation/sanitization, secrets management (.env, vault), dependency vulnerabilities (check lock files), rate limiting, security headers, data encryption. For each item found, note its status (implemented/partial/missing). For items NOT found, note what would typically be needed. Write findings + implementation suggestions to `.claude/go-live/assessments/auth-security.md`. Return ONE LINE: "Auth & Security assessment complete."

**testing-errors** →
> Assess go-live readiness for TESTING & ERROR HANDLING. Scan the codebase for: unit test suites + coverage, integration tests, E2E tests, test configs, CI test runs, test utilities, fixtures, mocking, coverage gaps, critical untested paths, global error boundaries, crash reporting (Sentry, Bugsnag, Crashlytics), error logging, fallback UIs, retry mechanisms, graceful degradation, unhandled rejection/exception handlers. For each item found, note its status (implemented/partial/missing). For items NOT found, note what would typically be needed. Write findings + implementation suggestions to `.claude/go-live/assessments/testing-errors.md`. Return ONE LINE: "Testing & Error Handling assessment complete."

**billing-analytics** →
> Assess go-live readiness for BILLING, PAYMENTS, ANALYTICS & MONITORING. Scan the codebase for: payment provider integrations (Stripe, etc.), subscription/plan management, invoicing, pricing pages, webhook handlers for payment events, trial/freemium logic, analytics integrations (Mixpanel, Amplitude, PostHog, GA), event tracking, dashboards, health check endpoints, uptime monitoring, log aggregation, APM. For each item found, note its status (implemented/partial/missing). For items NOT found, note what would typically be needed. Write findings + implementation suggestions to `.claude/go-live/assessments/billing-analytics.md`. Return ONE LINE: "Billing & Analytics assessment complete."

**infrastructure-cicd** →
> Assess go-live readiness for INFRASTRUCTURE, ENVIRONMENTS & CI/CD. Scan the codebase for: hosting/deployment config (Vercel, Railway, AWS, etc.), staging vs production environment setup, environment variable management, database hosting, CDN, domain/DNS setup, SSL certificates, Docker configs, scaling config, GitHub Actions or other CI pipelines, automated testing in CI, linting in CI, build steps, deployment automation, preview deployments, release process, version management. For each item found, note its status (implemented/partial/missing). For items NOT found, note what would typically be needed. Write findings + implementation suggestions to `.claude/go-live/assessments/infrastructure-cicd.md`. Return ONE LINE: "Infrastructure & CI/CD assessment complete."

**feedback** →
> Assess go-live readiness for FEEDBACK FUNNELS. Scan the codebase for: contact forms, bug reporting, support email setup, in-app feedback widgets, NPS/survey tools, help center/FAQ, user communication channels. For each item found, note its status (implemented/partial/missing). For items NOT found, note what would typically be needed. Write findings + implementation suggestions to `.claude/go-live/assessments/feedback.md`. Return ONE LINE: "Feedback assessment complete."

### After each agent completes
Set `state.assessments.{area} = "complete"` immediately (incremental checkpoint).

### On failure
Set `state.assessments.{area} = "failed"`, record in `state.errors`, retry once.

### When all done
Set `state.phase = "recommendations"`.

---

## Phase 2: Compile Recommendations & User Decisions

Launch one Agent (`subagent_type: "general-purpose"`, `model: "haiku"`) to compile:

> Read ALL assessment files from `.claude/go-live/assessments/`. Compile into a structured recommendations file at `.claude/go-live/recommendations.md` with these sections:
>
> ## External Services Needed
> List anything requiring new external service setup (even free-tier): e.g., Clerk, Stripe, Sentry, Vercel, Railway, Resend, PostHog. For each: name, purpose, free-tier availability, implementation complexity (small/medium/large).
>
> ## Major Business Decisions
> List significant business/architecture choices that need user input. Number each one. Examples: auth provider choice, payment provider, hosting platform, monitoring stack.
>
> ## Medium-Minor Items
> Briefly summarize remaining items that are important but don't require major decisions. Keep this section SHORT — just a bulleted list with one-line descriptions.
>
> Return ONE LINE: "Recommendations compiled."

After the agent completes, read `.claude/go-live/recommendations.md`.

### Work through decisions with the user:
1. Present "External Services Needed" — ask the user to confirm/modify each service choice one by one.
2. Present "Major Business Decisions" — work through each numbered item one by one, asking for the user's preference.
3. Present "Medium-Minor Items" as a summary — ask if the user wants to discuss any of them.

Save user decisions to `.claude/go-live/decisions.md` as you go.

Set `state.phase = "checklist"`.

---

## Phase 3: Generate Go-Live Checklist

Launch one Agent (`subagent_type: "general-purpose"`, `model: "haiku"`):

> Read `.claude/go-live/recommendations.md` and `.claude/go-live/decisions.md`. Generate a comprehensive go-live checklist at `.claude/go-live/GO-LIVE-CHECKLIST.md` using this format:
>
> ```markdown
> # Go-Live Checklist
> _Generated: {date}_
>
> ## Critical — Must Complete Before Launch
> - [ ] {item} — {brief description}
>
> ## High Priority — Should Complete Before Launch
> - [ ] {item} — {brief description}
>
> ## Nice to Have — Can Complete After Launch
> - [ ] {item} — {brief description}
>
> ## Already Done
> - [x] {item} — {brief description}
>
> ## User Decisions Log
> {decisions from decisions.md}
> ```
>
> Organize items by priority. Include concrete next steps for each item. Return ONE LINE: "Checklist generated."

After the agent completes, set `state.phase = "complete"`.

Tell the user: "Go-live checklist saved to `.claude/go-live/GO-LIVE-CHECKLIST.md`. Run `/project:production-audit` next — it will pick up this checklist and implement the items iteratively."

---

## Invariants

- Checkpoint `state.json` after every phase transition.
- Each agent writes to its own file — no shared writes.
- All agents return ONE-LINE summaries — details are on disk.
- Do NOT read assessment file contents into orchestrator context during Phase 1 — only the agent in Phase 2 reads them.
