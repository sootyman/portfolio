# Architecture & Tech Stack

## Overview

Investment portfolio dashboard designed to run on a local home server (miniPC or similar). The application allows a user to manage holdings, track performance over time, and view analytics through a dashboard.

---

## Tech Stack Decisions

### Frontend Framework: Next.js 15 (React)

**Choice**: Next.js 15 with TypeScript, using the App Router.

**Rationale**:
- Provides both the frontend (React) and backend API routes in a single framework, reducing deployment complexity.
- React's ecosystem has best-in-class charting libraries (Recharts, Chart.js) essential for portfolio analytics.
- File-based routing simplifies page organisation for a two-page application.
- TypeScript support is first-class and enforced from the start, catching data-shape errors early (critical for financial data).
- Static and server rendering options available — server-side rendering suits a private, local deployment well.
- Large, mature community; long-term support assured.

**Alternatives considered**:
- *SvelteKit*: Excellent performance and lightweight, but smaller ecosystem for financial charting components.
- *Vue/Nuxt*: Good option, but React ecosystem depth (especially for charts) tips the balance to Next.js.

---

### Backend / API: Next.js API Routes (Node.js)

**Choice**: Next.js API routes (`/app/api/...`) backed by Node.js 20 LTS.

**Rationale**:
- Collocated with the frontend — no separate service to run, manage, or containerise.
- Sufficient for a single-user local application; no throughput requirements that demand a dedicated microservice.
- Async/await, TypeScript, and Zod validation give a clean, type-safe API layer without ceremony.
- Node.js 20 LTS is well-supported in Docker and has a stable release track.

**Alternatives considered**:
- *Separate Express/Fastify service*: Adds operational complexity (two processes, two containers) without meaningful benefit for this use case.
- *Python/FastAPI*: Strong for data work but introduces a second runtime; not warranted here.

---

### Database: SQLite (via Prisma ORM)

**Choice**: SQLite as the data store, accessed through Prisma ORM.

**Rationale**:
- Zero-server operation: SQLite is a single file on disk — no database process to run or maintain.
- Perfect for a local, single-user application with modest write throughput.
- Prisma provides type-safe queries, schema migrations, and a developer-friendly workflow.
- The database file can be persisted in a Docker volume and backed up trivially (just copy the file).
- Sufficient performance: a personal portfolio has at most a few thousand holdings/price rows — well within SQLite's capabilities.

**Alternatives considered**:
- *PostgreSQL*: Powerful, but adds a second container and operational overhead not justified for a single-user app.
- *MySQL/MariaDB*: Same trade-offs as PostgreSQL.

---

### Charting: Recharts

**Choice**: Recharts for all data visualisations.

**Rationale**:
- Built on top of React and D3, composable and easy to integrate with Next.js/React.
- Supports line charts, area charts, bar charts, and pie/radial charts — all needed for portfolio analytics.
- Responsive container support built in, important for dashboard layouts.
- Actively maintained with TypeScript types included.

---

### Market Data Source: Yahoo Finance (via `yahoo-finance2`)

**Choice**: `yahoo-finance2` npm package for fetching equity prices and historical data.

**Rationale**:
- No API key required, reducing setup friction for a private home server.
- Supports equity lookup by ticker, historical OHLCV data, and quote data.
- Price data is cached in the local SQLite database (reducing live API calls and enabling user corrections).

**Note**: Yahoo Finance is an unofficial/undocumented API. For production-grade reliability, consider migrating to Alpha Vantage or Polygon.io (both offer free tiers) once the application is stable.

---

### Containerisation & Deployment: Docker Compose

**Choice**: Docker Compose for local deployment on the miniPC.

**Rationale**:
- Single `docker-compose.yml` defines the entire stack; `docker compose up -d` starts the application.
- The Next.js app is containerised as a standalone Node.js image — no cloud dependencies.
- SQLite database file is mounted via a named Docker volume for persistence across container restarts.
- No Kubernetes, no cloud orchestration — keeps operational complexity appropriate for a home server.
- Easy upgrades: `git pull && docker compose build && docker compose up -d`.

**Deployment topology**:

```
miniPC
└── Docker
    └── portfolio-app (Next.js, Node 20)
        ├── Port 3000 exposed on LAN
        └── ./data/portfolio.db (SQLite, Docker volume)
```

---

## Dependency Versions

| Dependency | Version | Purpose |
|---|---|---|
| Node.js | 20 LTS | Runtime |
| Next.js | 15.x | Full-stack framework |
| React | 19.x | UI library |
| TypeScript | 5.x | Type safety |
| Prisma | 6.x | ORM + migrations |
| SQLite (better-sqlite3) | 11.x | Database driver |
| Recharts | 2.x | Charting |
| yahoo-finance2 | 2.x | Market data |
| Zod | 3.x | Schema validation |
| Tailwind CSS | 4.x | Styling |
| Docker | 27.x | Containerisation |
| Docker Compose | 2.x | Local orchestration |

---

## Project Structure (Target)

```
portfolio/
├── app/                    # Next.js App Router
│   ├── api/                # API routes
│   │   ├── holdings/       # CRUD for portfolio holdings
│   │   └── prices/         # Price fetch/cache endpoints
│   ├── dashboard/          # Dashboard page
│   ├── portfolio/          # Portfolio management page
│   └── layout.tsx          # Root layout
├── components/             # Shared React components
├── lib/                    # Utilities (prisma client, validation, etc.)
├── prisma/
│   └── schema.prisma       # Database schema + migrations
├── public/                 # Static assets
├── docker-compose.yml      # Local deployment
├── Dockerfile              # App container image
└── package.json
```

---

## Design Principles

1. **Lightweight over heavyweight**: Every dependency earns its place. No cloud-native services on a miniPC.
2. **Single-user simplicity**: No multi-tenancy, no auth complexity (local network access assumed).
3. **Data ownership**: All data lives locally. No external persistence or telemetry.
4. **Type safety throughout**: TypeScript + Prisma + Zod form a type-safe pipeline from database to UI.
