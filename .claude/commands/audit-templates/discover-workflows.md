# Workflow Discovery — Agent Instructions

You are mapping every user-facing workflow in this codebase. Be exhaustive.

## What to Search For

Scan the entire codebase for:

1. **Page routes / URLs**: Look in route configs, page directories (`app/`, `pages/`, `src/routes/`), router files, navigation components.
2. **API endpoints**: Look in route handlers, controllers, Express/Fastify/Koa routers, API directories. Check for REST, GraphQL, and RPC patterns.
3. **CLI commands**: Look for commander, yargs, arg parsing, bin/ directories.
4. **UI flows**: Multi-step forms, wizards, modals triggered by user action, onboarding sequences, settings flows.
5. **Background jobs**: Cron jobs, queue processors, scheduled tasks, webhook handlers.
6. **Email/notification flows**: Transactional emails, push notifications, in-app notifications triggered by events.
7. **Auth flows**: Login, signup, password reset, MFA, OAuth callbacks, token refresh.
8. **Admin/management flows**: User management, settings, billing, reporting.

## How to Search

- Use Glob to find route files: `**/route*.{ts,tsx,js,jsx}`, `**/controller*.{ts,js}`, `**/handler*.{ts,js}`
- Use Glob for page directories: `**/app/**/page.{ts,tsx,js,jsx}`, `**/pages/**/*.{ts,tsx,js,jsx}`
- Use Grep for route patterns: `router.get`, `router.post`, `app.get`, `app.post`, `@Get`, `@Post`
- Use Grep for common framework patterns: `export default function`, `getServerSideProps`, `loader`, `action`
- Check `package.json` scripts for CLI entry points
- Check for cron/scheduler patterns: `cron`, `schedule`, `setInterval`

## Output Format

Write to the file path specified by the orchestrator. Use this EXACT format:

```markdown
# User Workflows

_Generated: {date} | Total: {N} workflows_

## 1. {Workflow Name}
- **Type**: UI flow | API endpoint | CLI command | Background job | Auth flow
- **Entry Point**: {URL, route, command, or trigger}
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
