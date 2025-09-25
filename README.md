# Boards & Tasks — Collaborative Kanban (Node/TS + Prisma + React/Vite)

A lightweight **Trello-style** app with **shared boards** and **tasks** using role-based access.

* **Backend:** Node.js, Express, TypeScript, Prisma (PostgreSQL), JWT (HTTP-only cookie), Swagger UI
* **Frontend:** React 18, TypeScript, Vite, TailwindCSS

> **Local defaults:** API at `http://localhost:4000`, Swagger at `/docs`; Frontend at `http://localhost:5173`.

---

## Features

* **Auth:** email/password, **JWT** issued as an **HTTP-only cookie** named `token` (`/api/auth/login`, `/api/auth/logout`, `/api/auth/me`, `/api/auth/register`).
* **Boards:** create/list/update/delete; **share** with users via roles (**OWNER / EDITOR / VIEWER**).
* **Tasks:** CRUD inside a board (filters + bulk delete).
* **Preferences:** per-user settings (e.g., view mode, auto-refresh).
* **OpenAPI/Swagger:** interactive docs at **`/docs`** on the API host.

---
📸 Screenshots


![1](./docs/screenshot1.png)


![2](./docs/screenshot2.png)



## Monorepo Layout

```
.
├─ backend/
│  ├─ src/
│  │  ├─ server.ts          # Express bootstrap, CORS, Helmet, JSON, cookies, Swagger at /docs
│  │  ├─ api.ts             # App setup/mount
│  │  ├─ auth.ts            # JWT sign/verify + cookie helpers
│  │  ├─ authorize.ts       # role guard (OWNER/EDITOR/VIEWER)
│  │  ├─ validate.ts        # request validation middleware
│  │  ├─ swagger.ts, openapi.ts
│  │  └─ routes/
│  │     ├─ auth.routes.ts
│  │     ├─ boards.routes.ts
│  │     ├─ tasks.routes.ts
│  │     └─ preferences.routes.ts
│  ├─ prisma/schema.prisma  # PostgreSQL models
│  └─ package.json, tsconfig.json, .env.example
└─ frontend/
   ├─ src/
   │  ├─ pages/ (Boards, BoardDetail, Preferences, Login, Register)
   │  ├─ components/ (TaskItem, TaskModal, ShareModal, Navbar, ...)
   │  ├─ hooks/ (useDarkMode, useToast, useViewMode)
   │  ├─ api.ts   # uses import.meta.env.VITE_API with credentials:'include'
   │  └─ auth.tsx
   ├─ tailwind.config.js, vite.config.ts, index.html
   └─ package.json
```

---

## Backend — Setup & Run

### Prereqs

* Node.js **18+**
* **Docker** (for local PostgreSQL) or a Postgres instance
* **OpenSSL** (to generate a JWT secret)

### 1) Start PostgreSQL (Docker)

```bash
docker run -d --name pg-boards \
  -e POSTGRES_USER=boards \
  -e POSTGRES_PASSWORD=boards \
  -e POSTGRES_DB=boards \
  -p 5432:5432 \
  postgres:16
```

### 2) Configure environment

Create `backend/.env`:

```env
# PostgreSQL connection for Prisma
DATABASE_URL="postgresql://boards:boards@localhost:5432/boards?schema=public&connect_timeout=10"

# JWT secret used to sign/verify the cookie (32+ bytes recommended)
# mac/linux:  openssl rand -base64 32
# windows pwsh:  [Convert]::ToBase64String((1..32 | % {Get-Random -Max 256}))
JWT_SECRET="paste-your-32+char-secret-here"

# Optional, defaults to 4000
# PORT="4000"
```

### 3) Install, create tables, run

```bash
cd backend
npm install

# Option A (with migration files, recommended)
npx prisma migrate dev --name init

# Option B (quick, no migration files)
# npx prisma db push

# Optional: inspect DB
npx prisma studio   # opens http://localhost:5555

# Start API (hot reload)
npm run dev
# → API ready on http://localhost:4000 — Swagger at /docs
```

**CORS:** server allows **`http://localhost:5173`** with `credentials: true` so the browser can send/receive the `token` cookie.

**Auth details:**

* `POST /api/auth/register` → create user
* `POST /api/auth/login` → sets `token` (HTTP-only cookie)
* `POST /api/auth/logout` → clears cookie (protected)
* `GET  /api/auth/me` → current user (protected)

Other groups:

* `/api/boards/*` (owner/member routes + share management)
* `/api/tasks/*` (board-scoped)
* `/api/preferences/*` (user-scoped)

---

## Frontend — Setup & Run

### 1) Configure environment

Create `frontend/.env.local`:

```env
# Base URL of the backend API
VITE_API="http://localhost:4000"
```

### 2) Install & run

```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

### Build/Preview

```bash
npm run build
npm run preview
# → http://localhost:4173
```

The frontend sends requests with `credentials: 'include'`, so make sure CORS on the server matches your frontend origin.

---

## Prisma Model (High-level)

* **User**: `id`, `email (unique)`, `name`, `passwordHash`, `createdAt`
* **Board**: `id`, `name`, `ownerId → User`, `createdAt`
* **BoardUser**: (`boardId`,`userId`) PK, `role: OWNER | EDITOR | VIEWER`
* **Task**: `id`, `boardId → Board`, `creatorId → User`, `title`, `content?`, `completed`, timestamps
* **Preference**: 1-to-1 with User (`autoRefreshInterval`, `taskView`)

---

## NPM Scripts

**Backend**

* `dev` — run with hot reload
* `build` — (if defined) build for production
* `start` — (if defined) run production build
* `prisma` — Prisma helpers (optional mapping)

**Frontend**

* `dev` — Vite dev server
* `build` — production build (`dist/`)
* `preview` — preview built app
* `lint` — lint sources

---

## Troubleshooting

* **“User table not found” / Prisma errors**
  Apply schema to DB: `npx prisma migrate dev` (or `npx prisma db push`). Ensure `DATABASE_URL` points to your running DB.

* **CORS / cookie not sent**
  Front must be at `http://localhost:5173`. Server CORS allows this origin with `credentials: true`. Keep the same protocol/host/port you configured.

* **Unauthorized (401)**
  Log in first; the backend expects the **`token`** cookie. The frontend must use `fetch(..., { credentials: 'include' })` (already done in `src/api.ts`).

* **Port mismatch**
  API uses `process.env.PORT` with fallback **4000**. Adjust `VITE_API` accordingly.

---

## License / Credits

Educational/demo project for collaborative boards & tasks.
Built with open-source tools listed above.

---
