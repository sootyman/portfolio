# Investment Portfolio Dashboard

A self-hosted investment portfolio management application for tracking holdings and viewing performance over time. Runs on a local home server (miniPC or similar).

## Features

- **Portfolio management**: Record holdings with asset type, ticker, quantity, purchase price, and dates
- **Dashboard**: Visualise portfolio performance with charts over user-selectable periods
- **Local data**: All data stored in a local SQLite database — no cloud dependencies
- **Price caching**: Market prices fetched from Yahoo Finance and cached locally to reduce API calls

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router, TypeScript) |
| UI | React 19 + Tailwind CSS 4 |
| Charts | Recharts 2 |
| Database | SQLite via Prisma ORM |
| Market data | yahoo-finance2 |
| Validation | Zod |
| Runtime | Node.js 20 LTS |
| Deployment | Docker Compose |

See [docs/architecture.md](docs/architecture.md) for full tech stack rationale and dependency versions.

## Getting Started

### Development

```bash
npm install
npx prisma migrate dev
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Production (Docker)

```bash
docker compose up -d
```

The application is available on port 3000. Data is persisted in a Docker volume.

## Commands

```bash
npm run dev          # Dev server (http://localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
npm test             # Test suite
npx tsc --noEmit     # Type check
npx prisma migrate dev    # Apply database migrations
npx prisma studio        # Database browser UI
```

## Project Structure

```
app/                 # Next.js App Router (pages + API routes)
components/          # Shared React components
lib/                 # Utilities (Prisma client, validation, helpers)
prisma/              # Database schema and migrations
docs/
  architecture.md    # Tech stack decisions and rationale
  requirements/      # Feature requirements docs
```

## Documentation

- [Architecture & Tech Stack](docs/architecture.md) — framework choices, database, deployment rationale
- [Requirements](docs/requirements/initial-feature.md) — initial feature specification

## Deployment Notes

Designed for a single-user local network deployment. No authentication layer is included by default — access is restricted by your local network. Do not expose this application directly to the internet without adding an authentication layer.
