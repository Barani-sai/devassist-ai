# DevAssist AI

An AI-powered enterprise development assistant that lets users interact with business
databases, generate code, analyze bugs, search information, and manage AI conversations
using natural language — built layer by layer following a 9-layer enterprise architecture.

This workspace contains **two separate projects**, side by side:

```
.
├── devassist-ai/          Backend — FastAPI, Python
└── devassist-frontend/    Frontend — Vue 3 + TypeScript + Quasar
```

Each has its own README with full setup details. This file is just the map.

## Architecture layers and where they live

| Layer | What it is | Status | Where |
|---|---|---|---|
| 1 | User layer (Admin/Developer/Employee/Business user) | Conceptual | — |
| 2 | Presentation layer — Vue dashboard | ✅ Done | `devassist-frontend/` |
| 3 | Application layer — FastAPI (auth, RBAC, middleware) | ✅ Done | `devassist-ai/app/` |
| 4 | AI orchestration — intent detection, workflow manager | ✅ Done | `devassist-ai/app/services/orchestrator/` |
| 5 | AI reasoning — Claude AI integration | ✅ Done | `devassist-ai/app/services/claude_client.py` |
| 6 | MCP layer — SQL MCP server, Mongo MCP server | ✅ Done | `devassist-ai/app/mcp/` |
| 7 | Database layer — SQL Server, MongoDB | ✅ Done (with fallback) | `devassist-ai/app/db/` |
| 8 | Response builder — Markdown, JSON, table, chart, PDF, Excel | ✅ Done | `devassist-ai/app/services/response_builder/` |
| 9 | Deployment pipeline — Docker, GitHub Actions, Azure | ✅ Done (Azure deploy step disabled until provisioned) | `devassist-ai/Dockerfile`, `.github/workflows/` |

All 9 layers have working code. What's left is refinement — natural-language-to-SQL
translation, real Azure deployment once resources are provisioned, and UI polish.

## Running everything locally

You need two terminals — the frontend talks to the backend over HTTP.

**Terminal 1 — backend:**
```powershell
cd devassist-ai
venv\Scripts\Activate.ps1
uvicorn app.main:app --reload
```
Runs on `http://127.0.0.1:8000`. Interactive API docs at `http://127.0.0.1:8000/docs`.

**Terminal 2 — frontend:**
```powershell
cd devassist-frontend
npm install
npm run dev
```
Runs on `http://localhost:5173` and proxies API calls to the backend automatically.

**Demo accounts** (seeded in-memory, not a real database yet): `admin` / `admin123`,
`dev` / `dev123`.

## Testing

Backend:
```powershell
cd devassist-ai
python -m pytest tests/ --ignore=tests/playwright -v    # unit/integration tests
pytest tests/playwright/ -v                              # API tests (needs server running)
```

Frontend:
```powershell
cd devassist-frontend
npm run build    # type-checks with vue-tsc, then builds
```

## Known gotchas (learned the hard way)

- **Moving/renaming the project folder breaks `venv` and `node_modules`.** Windows venv
  launchers (`uvicorn.exe`, etc.) bake in an absolute path at creation time; Vite/esbuild
  cache absolute paths too. If you move either project folder, delete and recreate both:
  ```powershell
  # backend
  Remove-Item -Recurse -Force venv
  python -m venv venv
  venv\Scripts\Activate.ps1
  pip install -r requirements.txt

  # frontend
  Remove-Item -Recurse -Force node_modules
  Remove-Item -Force package-lock.json
  npm install
  ```
- **Avoid spaces in the parent folder name** if you can (e.g. `Dev assist AI` →
  `DevAssistAI`). Not fatal, but it's one less category of tooling bugs to debug later.
- `devassist-ai/pytest copy.ini` looks like an accidental duplicate of `pytest.ini` —
  safe to delete unless you meant to keep both.
- Quasar's `<q-page>` component requires a parent `<q-layout>`. Standalone routes that
  don't render inside `MainLayout.vue` (currently `LoginPage.vue` and `NotFoundPage.vue`)
  must use a plain `<div>` instead — this caused a white-screen crash once and is now
  fixed in both files.

## Git — pushing each project to GitHub

These are **two separate repos** (different languages, different release cadences),
not one combined repo. Push them separately, each from its own folder.

**Backend:**
```powershell
cd devassist-ai
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/devassist-ai.git
git push -u origin main
```

**Frontend:**
```powershell
cd devassist-frontend
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/devassist-frontend.git
git push -u origin main
```

(Create each empty repo on GitHub first at https://github.com/new — don't check
"Add a README" or ".gitignore", since both already exist locally.)

If a repo already exists and you're just pushing updates:
```powershell
git add .
git commit -m "Describe what changed"
git push
```
