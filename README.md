# Abu Zayd Online Institute (AZOI) — Phase 1 Scaffold

This is a working, compiled scaffold for the AZOI platform, matching the developer prompt.
It is **not wired to live services yet** — no real database, Firebase project, or Cloudinary
account is connected. It's built so a developer can connect those with minimal changes.

```
azoi-platform/
├── web/       Next.js 14 + TypeScript + Tailwind frontend (App Router)
└── server/    Express + TypeScript API + PostgreSQL schema
```

## What's built

**Frontend (`/web`)** — fully compiled, all routes render:
- `/` — homepage (hero, faculties, courses, testimonials, stats)
- `/courses`, `/courses/[id]` — course catalogue and detail page
- `/login`, `/register` — auth forms (UI only, not yet calling the API)
- `/dashboard/student`, `/dashboard/teacher`, `/dashboard/admin` — the three role dashboards
- Brand system in `tailwind.config.ts`: emerald/gold/sand palette, Fraunces + Inter + Noto
  Naskh Arabic typography, an eight-point khatam-star signature motif
- All content currently comes from `lib/mock-data.ts` — replace each export with a real
  `fetch()` call to the API below once it's connected

**Backend (`/server`)** — fully typechecked:
- `src/db/schema.sql` — the complete Postgres schema (users, courses, lessons, enrollments,
  assignments, submissions, live classes, attendance, certificates, payments, notifications,
  audit logs) — mirrors every entity referenced in the developer prompt
- `src/routes/*` — Express routes for auth, courses, enrollments, assignments/grading,
  certificates, live classes, and payments, with JWT auth + role-based access control
- `src/middleware/auth.ts` — `requireAuth` and `requireRole()` guards

## Running it locally

### Frontend
```bash
cd web
npm install
npm run dev        # http://localhost:3000
```

### Backend
```bash
cd server
npm install
cp .env.example .env      # then fill in real values
# create a Postgres database, then load the schema:
psql $DATABASE_URL -f src/db/schema.sql
npm run dev         # http://localhost:4000
```

## Connecting real services (in the order I'd do it)

1. **PostgreSQL** — provision on Railway (matches your deployment target) or Supabase.
   Set `DATABASE_URL` in `server/.env`, run the schema file above.
2. **Firebase Authentication** — create a Firebase project, enable Email/Password (and any
   social providers you want). On the frontend, install `firebase` and initialize the client
   SDK. On the backend, install `firebase-admin` and verify ID tokens in `middleware/auth.ts`
   instead of (or alongside) the current JWT check — this becomes your SSO hub for the whole
   future ecosystem (Library, TV, Community, etc.), per the developer prompt.
3. **Cloudinary** — create an account, set the three `CLOUDINARY_*` values in `.env`. Use
   Cloudinary's Next.js/Node SDKs for signed uploads from the teacher dashboard, and their
   HLS streaming profile for lecture video.
4. **Live classes** — pick a provider (Zoom SDK, LiveKit, Jitsi, or Agora) and populate
   `meeting_url` when creating a `live_classes` row via `POST /api/live-classes`.
5. **Payments** — Stripe for international cards, Paystack or Flutterwave for local African
   rails. Wire the provider's SDK into `POST /api/payments`, and update `status` from their
   webhook.
6. **Deployment** — `web` → Vercel, `server` + Postgres → Railway. Set `CLIENT_ORIGIN` on the
   server to your deployed frontend URL, and point the frontend's API base URL at your
   deployed backend.

## Notes on scope

This scaffold intentionally does **not** include: the AI Islamic Assistant, Qur'an
Memorisation Tracker, Arabic Learning Module, Parent Dashboard, or any Phase 3 ecosystem
module (Library, TV, Publications, Community, AI) — those are Phase 2/3 per the roadmap.
The schema and route structure are built so those can be added as new tables and routers
without restructuring what's here.
