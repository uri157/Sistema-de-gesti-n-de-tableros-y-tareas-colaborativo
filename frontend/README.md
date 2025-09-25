# Frontend — Boards & Tasks (React + Vite + Tailwind)

Web client for a collaborative **boards & tasks** app (Trello-like).
Tech: **React 18**, **TypeScript**, **Vite**, **TailwindCSS**.

> Backend defaults: API at `http://localhost:4000`, Swagger at `/docs`.

---

## ✅ Requirements

* **Node.js 18+** and **npm**
* The backend running locally or accessible via URL

---

## 1) Install

```bash
cd frontend
npm install
```

---

## 2) Configure environment

Create **`.env.local`** in `frontend/`:

```env
# Base URL of the backend API (include protocol and port)
VITE_API="http://localhost:4000"
```

The app reads this in `src/api.ts` and sends requests with `credentials: 'include'` so the browser will include the auth cookie.

---

## 3) Run in development

```bash
npm run dev
```

Vite will print the URL (typically `http://localhost:5173`).

---

## 4) Build & preview

```bash
npm run build      # outputs production assets to dist/
npm run preview    # serves the built app (usually http://localhost:4173)
```

Deploy the contents of `dist/` to any static host. Remember to set `VITE_API` in your hosting/build environment to point at your API.

---

## 📂 Project structure (high level)

```
src/
├─ pages/          # Boards, BoardDetail, Preferences, Login, Register
├─ components/     # TaskItem, TaskModal, ShareModal, Navbar, etc.
├─ hooks/          # useDarkMode, useToast, useViewMode
├─ api.ts          # HTTP client (uses VITE_API, credentials:'include')
├─ auth.tsx        # client-side auth helpers
├─ App.tsx / main.tsx
└─ index.css / App.css
```

Tailwind is configured in `tailwind.config.js` and included via `index.css`.

---

## 🔐 Auth flow (expected)

1. **Register** or **Login** via the backend.
2. Backend sets an HTTP-only auth cookie (`token`).
3. Subsequent requests from the frontend include credentials; protected views use that session.

---

## 🧩 Troubleshooting

* **CORS / cookie not sent:** the backend must allow your frontend origin and `credentials: true`. Default dev origin is `http://localhost:5173`.
* **401 Unauthorized:** log in first; ensure `VITE_API` matches the API URL and that the backend is running.
* **Env changes not applied:** restart `npm run dev` after editing `.env.local`.
* **Styles not applied:** ensure `@tailwind base; @tailwind components; @tailwind utilities;` are in `src/index.css`.

---

## 📝 NPM scripts

```bash
npm run dev       # start Vite dev server
npm run build     # production build to dist/
npm run preview   # preview built app
npm run lint      # (if configured) lint the project
```
