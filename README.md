# URL Shortener with Analytics

A full-stack URL shortener application where authenticated users can create short links, track clicks, and view detailed analytics including click trends and visit history.

## Live Demo

> Add your Loom or YouTube demo link here before submission.

---

## Setup Instructions

### Prerequisites

- Node.js 20+
- pnpm 9+
- PostgreSQL database

### Installation

```bash
# Clone the repository
git clone <your-repo-url>
cd <project-directory>

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env
# Fill in DATABASE_URL and SESSION_SECRET in .env

# Push database schema
pnpm --filter @workspace/db run push

# Start the API server (terminal 1)
pnpm --filter @workspace/api-server run dev

# Start the frontend (terminal 2)
pnpm --filter @workspace/web run dev
```

### Environment Variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `SESSION_SECRET` | Secret key for JWT signing |

---

## Features Implemented

### Mandatory Features

| Feature | Status |
|---|---|
| User signup & login (JWT) | ✅ |
| Protected dashboard routes | ✅ |
| Per-user URL management | ✅ |
| URL shortening with unique short codes | ✅ |
| URL validation before shortening | ✅ |
| Server-side redirect handling | ✅ |
| Dashboard: original URL, short URL, created date, click count | ✅ |
| Delete shortened URLs | ✅ |
| Copy short URL from UI | ✅ |
| Click count per URL | ✅ |
| Timestamp recorded per visit | ✅ |
| Analytics: total clicks, last visited, recent visit history | ✅ |
| Responsive interface | ✅ |
| Loading, success, and error states | ✅ |
| Form validation messages | ✅ |

### Bonus Features

| Feature | Status |
|---|---|
| Custom alias for short URL | ✅ |
| QR code generation (downloadable SVG) | ✅ |
| Charts for daily click trends (Recharts) | ✅ |
| IP address & user agent tracking | ✅ |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser (React)                      │
│  /login  /signup  /dashboard  /urls  /urls/:id/analytics    │
│                  Tailwind CSS + shadcn/ui                   │
└─────────────────────┬───────────────────────────────────────┘
                      │ REST API (JWT Bearer token)
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  API Server (Express 5)                     │
│                                                             │
│  POST /api/auth/signup    — register user                   │
│  POST /api/auth/login     — login, returns JWT              │
│  GET  /api/urls           — list user's URLs                │
│  POST /api/urls           — create short URL                │
│  DELETE /api/urls/:id     — delete URL                      │
│  GET  /api/urls/:id/analytics  — click history + chart      │
│  GET  /api/analytics/summary   — dashboard summary          │
│  GET  /r/:code            — redirect + record click         │
└─────────────────────┬───────────────────────────────────────┘
                      │ Drizzle ORM
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  PostgreSQL Database                        │
│                                                             │
│  users         (id, email, password_hash, created_at)       │
│  short_urls    (id, user_id, original_url, short_code,      │
│                 click_count, created_at)                    │
│  click_events  (id, short_url_id, user_agent, ip_address,   │
│                 clicked_at)                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## AI Planning Document

### Planning Phase

The app was designed using an OpenAPI-first approach:

1. **Define the contract** — All endpoints were specified in `lib/api-spec/openapi.yaml` before any code was written.
2. **Generate typed helpers** — Orval generated React Query hooks and Zod validation schemas from the spec automatically.
3. **Build backend** — Express route handlers were implemented using the generated Zod schemas for input validation.
4. **Build frontend** — A design-focused subagent built the React UI using the generated React Query hooks for all data fetching and mutations.

### Architecture Decisions

- **JWT over sessions** — Stateless auth scales horizontally; tokens stored in `localStorage` with a `setAuthTokenGetter` interceptor on the fetch client.
- **Drizzle ORM** — Type-safe SQL without a heavy ORM; migrations are managed via `drizzle-kit push`.
- **OpenAPI codegen** — Single source of truth for the API contract; frontend types and backend validators are both derived from `openapi.yaml`, preventing drift.
- **Click tracking at redirect time** — Each `GET /r/:code` request inserts a `click_events` row and increments `short_urls.click_count` atomically, keeping counts accurate.
- **nanoid for short codes** — 7-character URL-safe codes; collision checked before insert.

### Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, Tailwind CSS, shadcn/ui, Recharts, qrcode.react |
| Backend | Node.js 24, Express 5, TypeScript |
| Database | PostgreSQL + Drizzle ORM |
| Auth | JWT (jsonwebtoken), bcryptjs |
| Validation | Zod (via Orval codegen) |
| Monorepo | pnpm workspaces |

---

## Assumptions

- Short codes are case-sensitive (e.g. `/r/abc123` ≠ `/r/ABC123`).
- Users can only see and manage URLs they created (no shared/public URLs between accounts).
- Click analytics are retained even after the link is deleted (analytics are cascade-deleted with the URL to keep things simple).
- The redirect endpoint (`/r/:code`) is publicly accessible without authentication so shared links work.
- The `click_count` column is denormalized for fast dashboard reads; it is kept in sync by the redirect handler.

---

This project is a part of a hackathon run by https://katomaran.com
