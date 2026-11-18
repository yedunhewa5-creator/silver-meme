# Forge Studio

A local, working AI website builder and automation studio. Rebuild sites from screenshots, edit them visually, let an AI agent modify real project files, connect databases, automate workflows, deploy with verified results — all from one browser app on your machine.

Nothing is mocked. Every action calls a real backend:
- The AI assistant and screenshot reconstruction use whichever model provider you configure (OpenAI, Anthropic, Gemini, Ollama, or any OpenAI-compatible endpoint). Without a provider, screenshot analysis falls back to **local heuristic reconstruction** and says so.
- Database actions hit real databases. Writes require confirmation and take a backup first.
- Automations run real HTTP, SQL, email, file and AI actions.
- Deployments are only marked successful after the hosted URL actually responded to an HTTP request.

---

## Requirements

- **Node.js 22.5 or newer** (this machine has a portable Node 24 in `~/.local/node/node-v24.11.1-win-x64`; it is not on PATH by default).
- npm (ships with Node).
- Windows, macOS or Linux. No external database is required to start — a local SQLite demo is seeded on first run.

### Using the portable Node on this machine

In a terminal:

```
%USERPROFILE%\.local\node\node-v24.11.1-win-x64\node.exe --version
```

To run commands with it, either use the full path to `node.exe`/`npm.cmd`, or add the folder to PATH for the session:

```
set PATH=%USERPROFILE%\.local\node\node-v24.11.1-win-x64;%PATH%
```

---

## Quick start

```
cd "C:\Users\SS COMPUTERS\Documents\Qoder\2026-11-18\82e8bbf7"
npm install
npm run dev
```

Open **http://localhost:5317** (development server with hot reload and API proxying).

For a production-style run:

```
npm run build
npm start
```

Open **http://127.0.0.1:4317**.

On first start the studio seeds:
- a demo project **“Nebula Ops (Demo)”** (clearly labelled demo data),
- a demo SQLite database with `customers`/`orders` tables,
- a paused demo automation.

You can edit or delete all of it.

---

## Connect an AI provider (optional but recommended)

1. Open **Settings** (left sidebar) → **AI providers**.
2. Pick a provider (OpenAI, Anthropic Claude, Google Gemini, Ollama, or an OpenAI-compatible endpoint).
3. Paste the API key and press **Save AI settings**. The key is encrypted with AES-256-GCM on the backend and is never returned to the browser.
4. Press **Test** to make a real request and confirm the connection.

Without a key the studio still works: the visual editor, database studio, automations, forms, deployment and version history are fully functional. AI chat and screenshot analysis will clearly report that no provider is connected (screenshots fall back to honest local heuristics).

Optional settings:
- **Email (SMTP)** powers email actions and form notifications.
- **Deployment providers** — paste a Netlify or Vercel token to enable those targets. Local hosting is always available.
- **Environment variables** (optional): `PORT`, `HOST`, `FORGE_DATA_DIR` (defaults to `./data`).

---

## What you can do

### Screenshot → website
**New from screenshot** → upload one or more screenshots → add a short brief → build. With an AI vision provider the studio reconstructs layout, palette and copy from the image; without one it produces a heuristic site and labels it as such. A screenshot contains no original source code — the studio reconstructs from visual evidence only and never claims to extract originals.

### Visual editor
Open a project → **Visual editor**. Click to select, edit text inline, drag sections to reorder, adjust theme colours/fonts, undo/redo, switch desktop/tablet/mobile. Saves are versioned automatically.

### AI assistant (autonomous agent)
Project → **AI assistant**. Ask in plain language (“add a pricing section”, “make it mobile responsive”, “create an admin dashboard”). The agent reads and edits real files, runs link/build checks and reports every step. Destructive actions (deleting files, running commands, non-GET requests, write SQL) pause for your approval. Every file write is snapshotted; you can roll back from **Version history**.

### Database studio
**Database** → add a connection: SQLite, PostgreSQL, MySQL/MariaDB, MongoDB, Supabase or Firebase. Test the connection, browse tables/collections, inspect schemas and rows, run SQL (writes require confirmation and create a backup first), manage migrations, and create/restore/download backups. Credentials are encrypted server-side and never sent to the browser.

### Automation center
**Automations** → build workflows with triggers (manual, schedule/cron, incoming webhook, form submission), optional conditions, and ordered actions (HTTP request, AI generate, database query, email, update project text, record activity, wait). Activate/pause, run manually with test payloads, and inspect step-by-step run logs. Webhook URLs include a secret.

### API integrations
**API integrations** → store a base URL + auth once (bearer, basic, X-API-Key or custom headers). Test it live and reuse it from automations without duplicating secrets.

### Deployment
Project → **Deploy**. Local hosting publishes a public link on this machine; Netlify/Vercel upload a real build when a token is configured. Deploying requires typing the project name to confirm, and the result is only marked successful after the URL responds with HTTP 200. Domain notes and Git (init/commit/log) live in project settings.

### FeelFree site (prepared for Vercel)

The project **FeelFree** (`prj_bmDpVoRRfsMs`) is fully built and passes the studio checks. It is already published locally at:

```
http://127.0.0.1:4317/s/e187ea7acdc86b2026eaf6d4f286c57f/
```

To put it online on Vercel under the name `feelfree`:

1. Create a token at <https://vercel.com/account/tokens> (scope: your personal account or team).
2. In the studio open **Settings → Deployment providers → Vercel**, paste the token, and save. It is stored encrypted on the backend.
3. Open the **FeelFree** project → **Deploy** → choose **Vercel**, project name `feelfree`, and type `FeelFree` to confirm.
4. The studio uploads the site, waits for the deployment to reach `READY`, then verifies the public URL with a real HTTP request. A deployment is never marked successful before that check passes.

Expected address: `https://feelfree.vercel.app`. If that subdomain is already taken, Vercel reserves the project name and returns a suffixed URL (`feelfree-xxxx.vercel.app`); a custom domain can be attached in the Vercel dashboard afterwards.

Note: only the generated website is deployed to Vercel. The studio itself is a long-running Node server with a persistent `data/` directory (SQLite + uploaded files), so it is meant to run on this machine or a server with a disk, not on Vercel's static/serverless hosting.

### Safety and history
- Automatic version snapshots on every save; non-destructive rollback with confirmation.
- Database backup before any confirmed write, restore, migration or automation write.
- File deletion requires confirmation; the home page cannot be deleted.
- Project deletion requires typing its exact name.
- Demo data is labelled; the studio never fabricates live trading, payment or transaction activity.
- All operations are recorded in **Activity log**.

---

## Tests

```
npm test
```

The suite boots the server on an isolated temporary data directory (it never touches your real `data/`) plus a mock AI endpoint, and verifies:
project creation and link checks, editor save guards and rollback, ZIP export, screenshot → project pipeline, real SQLite browsing, guarded writes → backup → confirm, backups/restore, migrations, integrations, automations (manual, form trigger, webhook, pause), AI settings encryption at rest, AI connectivity, screenshot analysis through the provider adapter, the agent tool loop writing real files, destructive-action guards, deployment confirmation and live HTTP verification, dashboard counters and activity logging.

---

## Project layout

```
server/            Express API (ES modules)
  ai/              provider adapters, screenshot analysis, agent loop
  automation/      workflow engine + scheduler
  build/           static site renderer/templates
  db/              database connectors (sqlite, pg, mysql, mongo, supabase, firebase)
  routes/          REST endpoints
  store/           projects, connections, integrations
src/               React app (Vite)
  pages/           dashboard, editor, code, chat, database, automations, deploy, settings…
  components/      shared UI kit
tests/             end-to-end API tests
data/              created at runtime: SQLite studio DB, projects, uploads, backups, encryption key
dist/              production build (generated)
```

## Notes

- The server binds to `127.0.0.1` by default — it is a local studio, not a public server. Change `HOST` deliberately if you know what you are doing.
- Encryption master key lives in `data/.master-key`. Back it up together with `data/` if you want to restore encrypted credentials later.
- Use `Ctrl+C` to stop `npm run dev` / `npm start`.
