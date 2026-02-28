# Workflow Discovery — Agent Instructions

You are mapping every user-facing workflow AND project admin/operational workflow in this codebase. Be exhaustive but only document workflows that actually exist in the code — do NOT invent or suggest workflows that are not implemented.

## What to Search For

Scan the entire codebase for:

1. **Page routes / URLs / Screen navigation**: Look in route configs, page directories (`app/`, `pages/`, `src/routes/`), router files, navigation components. For iOS: `UIViewController` subclasses, SwiftUI `NavigationStack`/`NavigationView`/`TabView`, storyboard scenes. For Android: Activities declared in `AndroidManifest.xml`, Fragments, Compose `NavHost`/`composable()` destinations, navigation graph XML. For React Native: screen components registered with navigators. For Flutter: route tables, `MaterialPageRoute`, `GoRouter` configurations.
2. **API endpoints**: Look in route handlers, controllers, Express/Fastify/Koa routers, API directories. Check for REST, GraphQL, and RPC patterns.
3. **CLI commands**: Look for commander, yargs, arg parsing, bin/ directories.
4. **UI flows**: Multi-step forms, wizards, modals triggered by user action, onboarding sequences, settings flows. (Mobile): bottom sheets, action sheets, tab-based flows, gesture-driven navigation, deep link entry points, widget interactions.
5. **Background jobs**: Cron jobs, queue processors, scheduled tasks, webhook handlers. For iOS: `BGTaskScheduler`, background `URLSession`, silent push handling, App Extensions (Today widgets, Share extensions, etc.). For Android: `WorkManager`, `JobScheduler`, foreground services, `BroadcastReceiver`, `ContentProvider`.
6. **Email/notification flows**: Transactional emails, push notifications, in-app notifications triggered by events. For iOS: APNs handling, `UNUserNotificationCenter`, notification service/content extensions. For Android: FCM, `FirebaseMessagingService`, notification channels.
7. **Auth flows**: Login, signup, password reset, MFA, OAuth callbacks, token refresh.
8. **Admin/management flows**: User management, settings, billing, reporting, dashboards.
9. **Analytics & monitoring**: Analytics integrations (e.g. Mixpanel, Amplitude, PostHog, Google Analytics), event tracking calls, analytics dashboards, error monitoring (e.g. Sentry, Bugsnag, Crashlytics), logging infrastructure, health check endpoints.
10. **Billing & payments**: Stripe/payment provider integrations, subscription management, invoicing, webhook handlers for payment events, pricing pages, plan management.
11. **Testing infrastructure**: Test suites, test configuration, E2E test flows, test utilities, test fixtures, mocking infrastructure.
12. **Error handling**: Global error boundaries, error reporting services, error logging, crash reporting, fallback UIs, retry mechanisms.
13. **Feedback funnels**: Contact forms, bug reporting flows, support email integrations, in-app feedback widgets, NPS/survey integrations, help center/FAQ pages.
14. **CI/CD & deployment**: GitHub Actions, deployment scripts, build configurations, environment configs, Dockerfiles, infrastructure-as-code, deployment pipelines.
15. **Staging & environment management**: Environment variable configs, staging vs production toggles, feature flags, environment-specific configurations.
16. **Deep link / URL scheme flows**: Universal Links (`apple-app-site-association`), custom URL schemes (`CFBundleURLTypes`), App Links (`assetlinks.json`), Android `intent-filter` with `VIEW` action, React Native deep link routing, Flutter deep link handling via `GoRouter`/`Navigator`.
17. **App lifecycle flows**: For iOS: `AppDelegate`/`SceneDelegate` methods, SwiftUI `scenePhase` handling, state restoration. For Android: `Activity` lifecycle callbacks, `ViewModel` saved state / `SavedStateHandle`, process death recovery patterns.
18. **Platform integration flows**: For iOS: Siri Shortcuts, App Clips, StoreKit/in-app purchases, Sign in with Apple. For Android: App Shortcuts, Instant Apps, Play Billing. Cross-platform: home screen widgets, platform share targets.

## How to Search

- Use Glob to find route files: `**/route*.{ts,tsx,js,jsx}`, `**/controller*.{ts,js}`, `**/handler*.{ts,js}`
- Use Glob for page directories: `**/app/**/page.{ts,tsx,js,jsx}`, `**/pages/**/*.{ts,tsx,js,jsx}`
- Use Grep for route patterns: `router.get`, `router.post`, `app.get`, `app.post`, `@Get`, `@Post`
- Use Grep for common framework patterns: `export default function`, `getServerSideProps`, `loader`, `action`
- Check `package.json` scripts for CLI entry points
- Check for cron/scheduler patterns: `cron`, `schedule`, `setInterval`
- Use Grep for analytics patterns: `analytics`, `track`, `identify`, `mixpanel`, `amplitude`, `posthog`, `gtag`, `GA4`
- Use Grep for error monitoring: `Sentry`, `Bugsnag`, `Crashlytics`, `captureException`, `captureMessage`
- Use Grep for billing/payments: `stripe`, `checkout`, `subscription`, `invoice`, `payment`, `billing`
- Use Glob for CI/CD configs: `.github/workflows/*.yml`, `Dockerfile*`, `docker-compose*`, `.env*`, `vercel.json`, `railway.json`, `fly.toml`, `Procfile`
- Use Glob for test configs: `jest.config.*`, `vitest.config.*`, `playwright.config.*`, `cypress.config.*`, `*.test.*`, `*.spec.*`
- Check for feature flags: `feature`, `flag`, `toggle`, `LaunchDarkly`, `unleash`, `flipper`
- Use Glob for mobile source files: `**/*.swift`, `**/*.storyboard`, `**/*.xib`, `**/Info.plist`, `**/*.kt`, `**/*.java`, `**/AndroidManifest.xml`, `**/res/navigation/*.xml`, `**/*.dart`
- Use Grep for mobile patterns: `UIViewController`, `NavigationStack`, `BGTaskScheduler`, `class.*Activity`, `@Composable`, `NavHost`, `<activity`, `<service`, `intent-filter`
- Check mobile build/config files: `Podfile`, `Package.swift`, `build.gradle`, `build.gradle.kts`, `AndroidManifest.xml`, `Info.plist`, `pubspec.yaml`

## Output Format

Write to the file path specified by the orchestrator. Use this EXACT format:

```markdown
# Workflows

_Generated: {date} | Total: {N} workflows_

## 1. {Workflow Name}
- **Type**: UI flow | API endpoint | CLI command | Background job | Auth flow | Screen navigation | Deep link handler | App extension | Platform integration | Admin/management | Analytics | Billing | Testing | Error handling | Feedback funnel | CI/CD | Environment config
- **Entry Point**: {URL, route, command, trigger, screen class, or deep link}
- **Steps**:
  1. {step description}
  2. {step description}
  ...
- **Key Files**:
  - `path/to/file1.ts`
  - `path/to/file2.tsx`
- **Dependencies**: {other workflow names, or "None"}
```

## Return to Orchestrator

Your final response message must be ONE LINE only:
`Discovered {N} workflows. Written to .claude/audit/workflows.md.`
Do NOT list workflows in your response — they are saved to disk.

## Quality Checklist

Before finishing, verify:
- [ ] Checked every top-level directory
- [ ] Found all API route files
- [ ] Found all page/view files
- [ ] Checked for background jobs and cron tasks
- [ ] Checked for email/notification triggers
- [ ] Each workflow lists ALL implementing files (not just the entry point)
- [ ] No duplicate workflows
- [ ] Checked for iOS view controllers, SwiftUI views, and storyboard scenes
- [ ] Checked for Android activities, fragments, and Compose screens
- [ ] Checked `AndroidManifest.xml` for declared components (activities, services, receivers, providers)
- [ ] Checked for deep link handlers (Universal Links, App Links, URL schemes, intent-filters)
- [ ] Checked for background task registrations (BGTaskScheduler, WorkManager, services)
- [ ] Checked for app extensions and platform integrations
- [ ] Checked for analytics/monitoring integrations and event tracking
- [ ] Checked for billing/payment integrations and subscription flows
- [ ] Checked for error handling infrastructure (error boundaries, crash reporting)
- [ ] Checked for feedback/support flows (contact forms, bug reporting)
- [ ] Checked for CI/CD configurations and deployment pipelines
- [ ] Checked for environment/staging configurations and feature flags
- [ ] Checked for testing infrastructure and test suites
- [ ] Did NOT invent or suggest workflows that are not actually implemented in the code
