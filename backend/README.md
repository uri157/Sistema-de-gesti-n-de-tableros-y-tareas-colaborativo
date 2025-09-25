# Backend — Boards & Tasks (Node + TS + Prisma + PostgreSQL)

This backend powers a collaborative boards & tasks app (Trello-like).
Tech: **Node.js**, **TypeScript**, **Express**, **Prisma**, **PostgreSQL**.

## ✅ Requisitos

* **Node.js 18+** y **npm**
* **Docker** (para levantar PostgreSQL local)
* **OpenSSL** (para generar el JWT secret; opcional pero recomendado)

---

## 1) Clonar e instalar

```bash
cd backend
npm i
```

---

## 2) Base de datos local (Docker)

Levanta un PostgreSQL local en el puerto **5432**:

```bash
docker run -d --name pg-boards \
  -e POSTGRES_USER=boards \
  -e POSTGRES_PASSWORD=boards \
  -e POSTGRES_DB=boards \
  -p 5432:5432 \
  postgres:16
```

> Para parar/iniciar: `docker stop pg-boards` / `docker start pg-boards`
> Para logs: `docker logs -f pg-boards`

---

## 3) Variables de entorno

Crea un archivo **`.env`** en `backend/` (junto al `package.json`) con:

```env
# PostgreSQL local
DATABASE_URL="postgresql://boards:boards@localhost:5432/boards?schema=public&connect_timeout=10"

# JWT (genera uno seguro)
# Linux/macOS:
#   openssl rand -base64 32
# Windows PowerShell:
#   [Convert]::ToBase64String((1..32 | % {Get-Random -Max 256}))
JWT_SECRET="paste-your-32+char-secret-here"
```

> En producción (Neon u otro gestionado), usa `sslmode=require` en la URL y tus credenciales reales:
> `postgresql://user:pass@host/db?schema=public&sslmode=require&connect_timeout=10`

---

## 4) Prisma: generar/migrar

```bash
# Crea/actualiza el esquema en la DB y genera el cliente
npx prisma migrate dev --name init

# (opcional) abrir Prisma Studio para ver tablas y datos
npx prisma studio
```

---

## 5) Ejecutar el servidor

```bash
# desarrollo (hot reload; si existe el script)
npm run dev

# alternativa si el repo usa build+start:
# npm run build && npm start
```

El puerto se define en el código (por defecto suele ser **3000**).
Revisa `src/server.ts` o `src/api.ts` si necesitas confirmar.

---

## 6) Probar la API

* **Swagger/OpenAPI** (según configuración):

  * `http://localhost:3000/docs` **o** `http://localhost:3000/swagger`
* Si el frontend corre en Vite (5173), asegúrate de que el front apunte a la URL del backend (por ej. `VITE_API_URL=http://localhost:3000`).

---

## 🧩 Troubleshooting

* **P1001 / “Can’t reach database”**: verifica que el contenedor `pg-boards` esté arriba (`docker ps`), puerto 5432 libre y `DATABASE_URL` correcta.
* **Falla JWT**: define `JWT_SECRET` en `.env` (32+ chars).
* **CORS desde el front**: si hay error de CORS, revisa el middleware en `src/server.ts` y agrega el origen `http://localhost:5173`.
* **Migraciones en producción**: usa `npx prisma migrate deploy` en CI/CD (no `migrate dev`).
* **Pool de conexiones (opcional)**: en servicios gestionados considera pgBouncer.

---

## 🗃️ Notas de despliegue (opcional)

* **Producción (Neon/Supabase/RDS)**:
  `DATABASE_URL="postgresql://user:pass@host/db?schema=public&sslmode=require&connect_timeout=10"`
* **Shadow DB** (algunos proveedores): si `migrate dev` lo requiere, añade `SHADOW_DATABASE_URL` en `.env`.

