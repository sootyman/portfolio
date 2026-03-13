# CLAUDE.md

<!-- IMPORTANT: Keep this file concise. Every line is loaded into every agent session
     (local and headless). Bloated instructions get ignored. For each line, ask:
     "Would removing this cause an agent to make a mistake?" If not, cut it.
     Move verbose reference material to .claude/skills/ (loaded on demand). -->

## Project Overview

Self-hosted investment portfolio dashboard (Next.js 15, TypeScript, SQLite/Prisma, Recharts). Deployed via Docker Compose on a local home server (miniPC). Two pages: portfolio holdings management and performance dashboard. See [docs/architecture.md](docs/architecture.md) for full tech stack rationale.

## Critical Policies

- **Never commit secrets.** API keys, credentials, and `.env` files must not be committed. Use environment variables.
- **No external data persistence.** All data must stay in the local SQLite database. No cloud sync, no external storage.
- **Single-user only.** No multi-tenancy. Do not add user/tenant scoping — it is not needed and adds complexity.

## Architecture

```
app/                 # Next.js App Router
  api/               # API routes (REST)
    holdings/        # Portfolio holdings CRUD
    prices/          # Market price fetch/cache
  dashboard/         # Dashboard page
  portfolio/         # Holdings management page
  layout.tsx         # Root layout
components/          # Shared React components
lib/                 # Utilities (Prisma client, validation, helpers)
prisma/
  schema.prisma      # Database schema
public/              # Static assets
docker-compose.yml   # Local deployment
Dockerfile           # App container
```

- Import alias: `@/` maps to the project root
- Database access: use the Prisma client from `@/lib/prisma`
- Validation: use Zod schemas; define them alongside the code that uses them
- Charts: Recharts components only — do not introduce a second charting library

## Commands

```bash
npm run dev               # Dev server (http://localhost:3000)
npm run build             # Production build
npm run lint              # ESLint
npm test                  # Full test suite
npx tsc --noEmit          # Type check
npx prisma migrate dev    # Apply DB migrations
npx prisma studio         # DB browser UI
docker compose up -d      # Start production stack
```

## Testing

- Framework: Jest (unit/integration) — to be configured when the first features land
- New API routes require at least a happy-path and an error-path test
- Run `npm test` before every PR

## Codebase Discovery (Required Before Writing Code)

Before implementing ANY feature or fix, agents MUST explore the existing codebase:

1. **Understand the project**: Read `package.json` to identify the tech stack, framework, and key dependencies. Read the top-level directory structure to understand how the project is organized.
2. **Study the target area**: Read 2-3 existing files in the directories you will be modifying. Note naming conventions (camelCase vs kebab-case, file naming patterns), import style, error handling patterns, and how similar features are already implemented.
3. **Find reusable code**: Before creating any new utility, helper, component, or abstraction, search the codebase for existing ones. Use Grep and Glob to look for similar functionality. Reuse what exists rather than duplicating.
4. **Match the project's patterns**: Your code must look like it was written by the same team. Match the existing code style, naming conventions, module structure, and architectural patterns. If the project uses a specific pattern (e.g., service layers, repository pattern, API response helpers), follow it.

This applies to all agents: dev, review, and fix. The review agent should verify that PRs follow these principles.

## Code Quality and Tech Debt

- **Boy scout rule**: Leave code better than you found it. If you touch a file and see a small, safe improvement (dead import, obvious simplification, missing type), fix it in the same PR.
- **Do not create new tech debt**: No TODO/FIXME/HACK comments unless the issue explicitly calls for a partial implementation. No copy-pasted code when a shared utility exists or should be created. No quick workarounds that bypass the project's established patterns.
- **Prefer existing abstractions**: If the project has a utility for something (API responses, validation, logging, error handling), use it. Do not create parallel implementations.
- **Keep changes focused**: Address tech debt only in files you are already modifying for the current task. Do not refactor unrelated code.

## Headless Agent Guidelines

<!-- These rules apply when agents run autonomously in GitHub Actions
     via anthropics/claude-code-action@v1. They have no human to ask. -->

### Verification Before PR

Every agent MUST verify before creating a pull request:
1. Tests pass: `npm test`
2. Types clean: `npx tsc --noEmit`
3. Lint clean: `npm run lint`
4. Build succeeds: `npm run build`

If any check fails, fix the issue. If it cannot be fixed after 2 attempts, stop and report the failure in a PR comment. Do NOT create a PR with failing checks.

### Autonomy Boundaries

Agents operate autonomously within these boundaries:

**Proceed without asking:**
- Implement features described in the GitHub Issue
- Write and run tests
- Fix lint errors and type errors
- Create feature branches and open PRs
- Update issue checkboxes to track progress
- Fix small tech debt in files you are already modifying

**Stop and escalate (comment on the issue/PR):**
- Ambiguous or contradictory requirements
- Changes that affect authentication, authorization, or encryption
- Database schema migrations or destructive data changes
- Architectural decisions not covered by this file
- Tests that fail for reasons unrelated to the current change

### Progress Tracking

**GitHub Issue checkboxes**: Dev agents MUST update checkboxes as they work. Update 3-5 times during implementation, not in a single batch at the end.

**PR task lists**: After creating a PR, include a checklist of completed and remaining items in the PR body. Update before pushing final changes.

### Commit Conventions

- One logical change per commit
- Commit messages: imperative mood, concise (`Add user validation`, not `Added some validation stuff`)
- PR title matches the issue title
- PR body includes `Closes #N` to auto-link the GitHub Issue

### Context Management

- Read CLAUDE.md at session start (this happens automatically)
- Stay focused on the assigned issue. Do not refactor unrelated code.
- If context is running low, prioritize: (1) completing the current task, (2) running tests, (3) opening the PR

## Working Style

### Task Management
1. Write plan to `tasks/todo.md` with checkable items
2. Check in before starting implementation
3. Mark items complete as you go; add review section when done

### Non-Trivial Changes
For API integrations, schema changes, auth/security, core workflows:
1. **Impact & Risk**: Identify affected areas/dependencies. Call out security, data integrity, performance risks. If uncertain, STOP and ask.
2. **Design & Approach**: Describe proposed solution. Confirm alignment with existing patterns.
Do NOT write code until both steps are complete. If something goes sideways, STOP and re-plan.

### Autonomous Bug Fixing
When given a bug report: just fix it. Point at logs, errors, failing tests - then resolve them. Zero hand-holding required.

### Self-Improvement Loop
After ANY correction: update `tasks/lessons.md` with the pattern. Review lessons at session start.

## Custom Triggers

These are natural language phrases typed in conversation (not registered skills):

- **`qc`**: 3-iteration deep review - (1) correctness & completeness, (2) architecture & tech debt, (3) security, performance, production readiness
- **`runit`**: Execute all remaining steps without pausing. Only stop when fully implemented, tested, and production ready.
- **`name <label>`**: Set the terminal tab title to the given label so the user can identify this agent session. Run: `echo -ne "\033]0;<label>\007"` via Bash. Confirm with: "Session named: <label>". **VS Code prerequisite** (one-time): add `"terminal.integrated.tabs.title": "${sequence}"` to VS Code settings - without this, VS Code ignores title escape sequences.

## Compaction
When compacting, always preserve: the full list of modified files, any test commands that need to run, and the current task plan.
