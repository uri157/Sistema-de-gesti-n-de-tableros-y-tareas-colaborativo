# Backend — Boards & Tasks (Node + TS + Prisma + PostgreSQL)

This backend powers a collaborative boards & tasks app (Trello-like).
Tech: **Node.js**, **TypeScript**, **Express**, **Prisma**, **PostgreSQL**.

## ✅ Requirements

* **Node.js 18+** and **npm**
* **Docker** (to run PostgreSQL locally)
* **OpenSSL** (to generate the JWT secret; optional but recommended)

---

## 1) Clone & install

```bash
cd backend
npm i
```

---

## 2) Local database (Docker)

Launch a local PostgreSQL instance on port **5432**:

```bash
docker run -d --name pg-boards \
  -e POSTGRES_USER=boards \
  -e POSTGRES_PASSWORD=boards \
  -e POSTGRES_DB=boards \
  -p 5432:5432 \
  postgres:16
```

> To stop/start: `docker stop pg-boards` / `docker start pg-boards`
> Logs: `docker logs -f pg-boards`

---

## 3) Environment variables

Create a **`.env`** file in `backend/` (next to `package.json`) with:

```env
# Local PostgreSQL
DATABASE_URL="postgresql://boards:boards@localhost:5432/boards?schema=public&connect_timeout=10"

# JWT (generate a strong one)
# Linux/macOS:
#   openssl rand -base64 32
# Windows PowerShell:
#   [Convert]::ToBase64String((1..32 | % {Get-Random -Max 256}))
JWT_SECRET="paste-your-32+char-secret-here"
```

> In production (Neon or another managed service), use `sslmode=require` in the URL and your real credentials:
> `postgresql://user:pass@host/db?schema=public&sslmode=require&connect_timeout=10`

---

## 4) Prisma: generate/migrate

```bash
# Create/update the DB schema and generate the client
npx prisma migrate dev --name init

# (optional) open Prisma Studio to browse tables and data
npx prisma studio
```

---

## 5) Run the server

```bash
# development (hot reload; if the script exists)
npm run dev

# alternative if the repo uses build+start:
# npm run build && npm start
```

The port is defined in code (default is usually **3000**).
Check `src/server.ts` or `src/api.ts` if you need to confirm.

---

## 6) Test the API

* **Swagger/OpenAPI** (depending on configuration):

  * `http://localhost:3000/docs` **or** `http://localhost:3000/swagger`
* If the frontend runs on Vite (5173), make sure it points to the backend URL (e.g., `VITE_API_URL=http://localhost:3000`).

---

## 🧩 Troubleshooting

* **P1001 / “Can’t reach database”**: ensure the `pg-boards` container is running (`docker ps`), port 5432 is free, and `DATABASE_URL` is correct.
* **JWT failure**: set `JWT_SECRET` in `.env` (32+ chars).
* **CORS from the frontend**: if you see CORS errors, check the middleware in `src/server.ts` and add the origin `http://localhost:5173`.
* **Prod migrations**: use `npx prisma migrate deploy` in CI/CD (not `migrate dev`).
* **Connection pooling (optional)**: consider pgBouncer for managed services.

---

## 🗃️ Deployment notes (optional)

* **Production (Neon/Supabase/RDS)**:
  `DATABASE_URL="postgresql://user:pass@host/db?schema=public&sslmode=require&connect_timeout=10"`
* **Shadow DB** (some providers): if `migrate dev` requires it, add `SHADOW_DATABASE_URL` in `.env`.
