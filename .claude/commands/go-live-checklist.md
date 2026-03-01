# Go-Live Readiness Assessment

Assess whether the current project is ready to launch as a production business. Identify gaps, work through major decisions with the user, and produce a comprehensive go-live checklist.

## Initialize

Run: `mkdir -p .claude/go-live/assessments`

If `.claude/go-live/state.json` exists, read it and jump to the current `phase`.
Otherwise, write: `{"phase":"assessment","assessments":{"auth-security":"pending","testing-errors":"pending","billing-analytics":"pending","infrastructure-cicd":"pending","feedback":"pending","app-store-readiness":"pending"},"errors":[]}`

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

**app-store-readiness** →
> Assess go-live readiness for APP STORE (iOS) and GOOGLE PLAY STORE (Android) submission. First determine which platforms the project targets (iOS, Android, or both) by scanning for Xcode project files, Android manifests, React Native/Flutter/Capacitor configs, etc. Skip any platform that is not present. For each applicable platform, assess the following:
>
> **iOS App Store — Metadata & Assets**: App Store Connect configuration or fastlane metadata, bundle identifier, app name/subtitle/description/keywords/category, app icon (all required sizes including 1024x1024), launch screen / splash screen, screenshots for required device sizes (6.7", 6.5", 5.5" iPhones; iPad Pro), preview videos (optional), privacy policy URL, marketing URL, support URL.
>
> **iOS App Store — Compliance & Review**: App Review Guidelines compliance (no placeholder content, all features functional, no hidden features, no misleading descriptions), age rating / content rating configured, App Tracking Transparency (ATT) prompt if tracking (with `NSUserTrackingUsageDescription`), App Privacy Details / privacy nutrition labels completed, export compliance (encryption usage declaration — `ITSAppUsesNonExemptEncryption` in Info.plist), all `NS*UsageDescription` keys set for every permission requested (Camera, Location, Microphone, Photo Library, Contacts, etc.), minimum deployment target meets current App Store requirements, Sign in with Apple implemented if other third-party sign-in is offered, in-app purchase / subscription configuration if applicable (products, pricing, StoreKit integration, receipt validation, restore purchases), Universal Links configured if deep linking is used, push notification entitlements and APNs configuration.
>
> **iOS App Store — Build & Release**: Xcode release build configuration (code signing, provisioning profiles, team ID), archive and upload workflow (manual, Xcode Cloud, fastlane, or CI), TestFlight beta testing configured, bitcode / app thinning settings, app size within limits (initial install < 200MB over cellular), crash-free rate / stability (no known crash-on-launch), version and build number management.
>
> **Google Play Store — Metadata & Assets**: Google Play Console configuration or fastlane supply metadata, application ID (package name), app name/short description/full description/category, app icon (512x512), feature graphic (1024x500), screenshots for phone, 7" tablet, 10" tablet (minimum 2 per type), preview video (optional YouTube link), privacy policy URL, contact email/website/phone.
>
> **Google Play Store — Compliance & Review**: Content rating questionnaire completed, Data Safety section completed (data collection, sharing, security practices), target audience and content declarations (especially if children are a target), ads declaration if serving ads, permissions declared in `AndroidManifest.xml` are minimal and justified, `targetSdkVersion` meets current Google Play requirements, 64-bit native library support (`arm64-v8a`, `x86_64`), large screen / tablet support declared and tested, app complies with Developer Program Policies (no deceptive behavior, proper disclosure, billing policy compliance for digital goods), Google Play Billing for in-app digital purchases if applicable (products, subscriptions, billing client integration), Android App Links configured if deep linking is used.
>
> **Google Play Store — Build & Release**: Android App Bundle (AAB) format used (not APK for Play Store), ProGuard/R8 code shrinking and obfuscation enabled for release, Play App Signing enrolled, release tracks configured (internal → closed → open → production), pre-launch report reviewed (Firebase Test Lab automated testing), app size optimized (download size, install size), version code and version name management, release notes for each version.
>
> **Cross-platform (if applicable)**: Are platform-specific assets (icons, splash screens, screenshots) maintained for both platforms? Are platform-specific metadata files (Info.plist, AndroidManifest.xml) fully configured? Are there any web-only placeholder screens or features that are non-functional on mobile? If using a cross-platform framework (React Native, Flutter, Capacitor, etc.), are native modules properly linked and built for both platforms?
>
> For each item found, note its status (implemented/partial/missing). For items NOT found, note what would typically be needed. Write findings + implementation suggestions to `.claude/go-live/assessments/app-store-readiness.md`. Return ONE LINE: "App Store Readiness assessment complete."

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
