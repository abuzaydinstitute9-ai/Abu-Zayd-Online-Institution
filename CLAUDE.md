# AZOI — Abu Zayd Online Institute

A global Islamic learning platform. Phase 1 scaffold: monorepo with a Next.js frontend and an Express/Postgres backend that are **not yet wired together**.

## Repo layout

```
azoi-platform/
├── web/       Next.js 14 (App Router) + TypeScript + Tailwind — the actual site
└── server/    Express + TypeScript + PostgreSQL API — built but not connected to the frontend
```

Treat these as two independent apps that happen to live in one repo. Always `cd web` or `cd server` before running npm commands — there is no root `package.json`.

## Current state (read this before assuming something works)

- **Frontend renders entirely from mock data** in `web/lib/mock-data.ts` (courses, faculties, daily verse/hadith, testimonials, stats, student profile, assignments, teacher courses, admin stats, recent activity). No page currently calls the API.
- **Backend is fully built but has nothing to talk to**: no live Postgres database, no Firebase project, no Cloudinary account. `server/.env.example` shows what's needed; a real `.env` doesn't exist yet.
- **Login/register pages are UI only** — they don't call `POST /api/auth/*` yet.
- **The frontend is configured for static export** (`output: "export"` in `web/next.config.js`) so it can deploy to GitHub Pages via `.github/workflows/deploy-pages.yml`. This means: no API routes in `app/`, no Server Actions, no `next/image` remote loading without `unoptimized: true`, no `cookies()`/`next/headers`. Keep it that way unless we deliberately move off static export.
- `web/next.config.js` has a `repoName` constant used for `basePath`/`assetPrefix` under GitHub Actions — must match the actual GitHub repo name or deployed asset paths break.

## Frontend (`/web`)

- App Router, all pages are Server Components unless a file starts with `"use client"`.
- Routes: `/`, `/courses`, `/courses/[id]` (uses `generateStaticParams()` over `mock-data.ts` courses), `/login`, `/register`, `/dashboard/student`, `/dashboard/teacher`, `/dashboard/admin`.
- Shared components in `web/components/`: `Navbar`, `Footer`, `Button`, `CourseCard`, `StatCard`, `DashboardShell`. Reuse these rather than inlining new markup for nav/cards/dashboard chrome.
- Design tokens live in `web/tailwind.config.ts` — brand colors are `emerald` (trust/knowledge), `gold` (excellence/certification), `sand` (warmth/paper), `ink` (text). Fonts: `font-display` (Fraunces, headings), `font-body` (Inter, body), `font-arabic` (Noto Naskh Arabic, Qur'an/Arabic text). Don't introduce new ad-hoc colors or fonts — extend the token list instead.
- Run: `cd web && npm install && npm run dev` → http://localhost:3000
- Build/typecheck sanity check: `npm run build` (note: pulls Fraunces/Inter/Noto Naskh Arabic from Google Fonts at build time — needs real internet access, will fail in a network-restricted sandbox).

## Backend (`/server`)

- Express + TypeScript, JWT auth (`jsonwebtoken` + `bcryptjs`), Postgres via `pg`, validation via `zod`.
- `src/db/schema.sql` — full schema: users, courses, lessons, enrollments, assignments, submissions, live_classes, attendance, certificates, payments, notifications, audit_logs.
- `src/routes/`: `auth.ts`, `courses.ts`, `enrollments.ts`, `assignments.ts`, `certificates.ts`, `liveClasses.ts`, `payments.ts`, `users.ts`.
- `src/middleware/auth.ts` — `requireAuth` and `requireRole()` guards.
- Run: `cd server && npm install && cp .env.example .env` (fill in `DATABASE_URL` etc.) → `npm run dev` → http://localhost:4000. Without a real `DATABASE_URL`, the server boots fine but any route touching Postgres will error — that's expected, not a bug.

## What NOT to build yet

Per the original scope, these are Phase 2/3 and should not be added casually: AI Islamic Assistant, Qur'an Memorisation Tracker, Arabic Learning Module, Parent Dashboard, or any other ecosystem module (Library, TV, Publications, Community). Ask before starting one of these — the schema/routes are intentionally structured so they can be added later without restructuring what exists.

## Conventions

- British English in all user-facing copy (this platform serves a Nigerian curriculum audience).
- Prefer editing `mock-data.ts` to add/change displayed content over hardcoding values in page components.
- When eventually wiring frontend → backend, replace one `mock-data.ts` export at a time with a real `fetch()` call — don't do a big-bang rewrite.
- Don't commit `.env`, `node_modules/`, `.next/`, or `dist/` — already covered by `.gitignore`.
