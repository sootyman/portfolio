# ADR-001: Full-Stack Architecture

## Context

The investment dashboard needs a frontend to display portfolios, charts, and AI chat, plus a backend to handle data storage, authentication, financial calculations, and AI agent calls. The frontend and backend choices are tightly coupled — picking one often constrains the other — so we're deciding them together.

The key constraints for this project:
- AI portfolio advice requires calling an LLM (likely Claude) with streaming responses
- Financial calculations (returns, allocations, performance) benefit from good numeric libraries
- Investment data needs to be persisted securely per user
- We want a good developer experience without unnecessary complexity

---

## Option A — Next.js Monolith

**Everything in one codebase.** Next.js handles both the React frontend and the API (via API routes and Server Actions). You write TypeScript throughout, deploy to Vercel with zero config, and never have to context-switch between two repos.

- Frontend: React with Next.js App Router
- Backend: Next.js API routes + Server Actions
- Language: TypeScript end-to-end
- Deployment: Vercel (or any Node.js host)

**Why it works here:** The AI streaming chat can be wired directly into a Next.js Route Handler with no separate service. Financial logic lives in server-side utilities. One repo, one deploy command.

> **Recommended** — best balance of simplicity, capability, and ecosystem for this project.

---

## Option B — SvelteKit Monolith

**Same idea as Option A but with Svelte instead of React.** SvelteKit has server routes built in, so you get backend + frontend in one repo. Svelte is genuinely faster at runtime and has less boilerplate than React — components are smaller and easier to read.

- Frontend: Svelte with SvelteKit
- Backend: SvelteKit server routes
- Language: TypeScript end-to-end
- Deployment: Vercel, Netlify, or Node adapter

**The trade-off:** Svelte has a smaller ecosystem than React. Charting libraries, data table components, and financial UI packages are more plentiful in the React world. For a data-heavy dashboard, that gap matters.

---

## Option C — React Frontend + Separate Node.js API

**Two separate services.** A Vite-powered React app for the frontend, and a standalone Fastify or Hono API server for the backend. They talk to each other over HTTP.

- Frontend: React + Vite
- Backend: Fastify or Hono (Node.js) — fast, TypeScript-native, purpose-built APIs
- Language: TypeScript throughout
- Deployment: Frontend to Vercel/Netlify, API to Fly.io or Railway

**When this makes sense:** If you expect the backend to be consumed by multiple clients (mobile app, third-party integrations) or you want strict separation of concerns. The overhead is real though — two repos, two CI pipelines, CORS to manage, and local dev requires running two processes.

---

## Option D — Next.js Frontend + Python FastAPI Backend

**Separate services, but with Python powering the backend.** Next.js handles the React UI; a Python FastAPI service handles data, calculations, and AI calls. Python has excellent financial libraries (pandas, numpy, scipy) and native support for many AI/ML tools.

- Frontend: React with Next.js
- Backend: Python FastAPI
- Languages: TypeScript (frontend) + Python (backend)
- Deployment: Next.js to Vercel, FastAPI to Fly.io or Railway

**When this makes sense:** If financial calculations get complex (portfolio optimisation, Monte Carlo simulations, risk modelling), Python pays off fast. If you're keeping it to basic returns and allocations, it's overkill — two languages means two contexts to maintain and two teams (or one developer who enjoys pain).

---

## Comparison Table

|                          | Option A — Next.js Mono | Option B — SvelteKit Mono | Option C — React + Node | Option D — Next.js + Python |
|--------------------------|:-----------------------:|:-------------------------:|:-----------------------:|:---------------------------:|
| Dev experience           | ★★★★★                  | ★★★★★                    | ★★★★                   | ★★★                        |
| AI streaming integration | Easy                    | Easy                      | Easy                    | Easy                        |
| Financial library support| Good (JS libs)          | Good (JS libs)            | Good (JS libs)          | Excellent (pandas, numpy)   |
| Charting / UI ecosystem  | Excellent               | Good                      | Excellent               | Excellent (frontend)        |
| Deployment complexity    | Low                     | Low                       | Medium                  | Medium-High                 |
| Monorepo needed?         | No                      | No                        | Optional                | Yes (or two repos)          |
| Best for                 | Most projects           | Lean/fast dashboards      | Multi-client APIs       | Complex financial maths     |

---

## Recommended Option

**Option A — Next.js Monolith.**

For an investment dashboard at this stage, the simplicity of one codebase, one deployment, and one language wins. Next.js App Router handles server-side rendering for fast initial loads, API routes handle the AI streaming and data endpoints, and the React ecosystem has every charting and data table library you might need. If the financial calculations ever grow complex enough to need Python, you can extract them into a microservice later — but you almost certainly won't need to start there.

## Consequences

- All application logic lives in one Next.js codebase
- API routes handle auth, data, and AI calls server-side
- Financial calculations written in TypeScript (using libraries like `financejs` or plain maths utilities)
- Deployed as a single service — simpler CI/CD and cost model
- Easy to pivot to a separate API later if needed
