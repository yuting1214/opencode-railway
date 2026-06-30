# OpenCode Devbox on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/opencode-devbox)

A one-click, **persistent dev box for [OpenCode](https://opencode.ai)** — the open-source AI coding
agent — with **two ways in from one service**:

- 🌐 **Web UI** — `opencode web` on a **public Railway domain** (HTTP basic auth), open it in a browser.
- 💻 **TUI** — `railway ssh` into the same box and type `opencode`, just like Claude Code in a terminal.

Both share the same `/workspace`, auth, and repos, which live on a Railway volume and survive redeploys.

---

## Quick start

1. **Deploy** this template (button above). It provisions the service, a **Volume** at `/workspace`,
   and a **public domain**.
2. *(Optional)* set a provider key in **Variables** (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, or
   `OPENROUTER_API_KEY`) — or just `opencode auth login` later. Optional: `GITHUB_TOKEN`,
   `GIT_USER_NAME`, `GIT_USER_EMAIL`, `OPENCODE_SERVER_PASSWORD`.
3. **Web UI:** open the service's public URL. Log in with user `opencode` and the password printed
   in the **deploy logs** (`[boot] opencode web auth → ...`), or whatever you set in
   `OPENCODE_SERVER_PASSWORD`.
4. **TUI:** from your machine —
   ```bash
   railway link            # pick this project/service
   railway ssh             # drop into the same container
   opencode                # enter the OpenCode TUI
   ```
5. Clone a repo and start coding — work in `/workspace/repos` so it persists:
   ```bash
   cd /workspace/repos && git clone git@github.com:you/your-repo.git
   ```

## What's in the box

| Tool | Why |
|------|-----|
| **OpenCode CLI** (`opencode-ai`) | `opencode web` (browser UI, the main process), `opencode` (TUI), `opencode run "…"` (headless) |
| **git** + **SSH key** | clone/commit; an ed25519 key is generated on first boot (see `~/.ssh/id_ed25519.pub`) |
| **GitHub CLI** (`gh`) | `gh pr create`, `gh issue`, etc. — auto-authenticated from `GITHUB_TOKEN` |

## How it works

- The container's main process is **`opencode web --hostname 0.0.0.0 --port $PORT`**, mapped to the
  public Railway domain and protected by HTTP basic auth (`OPENCODE_SERVER_PASSWORD`, auto-generated
  and persisted on the volume if unset). This keeps the service alive.
- `railway ssh` connects to the **same** container independently of that process, so `opencode` (TUI)
  works alongside the web UI.
- The entrypoint symlinks OpenCode's config/state and `~/.ssh` onto `/workspace`, so credentials and
  sessions persist across redeploys; provider keys are mirrored into the SSH login shell.

See [`docs/USAGE.md`](docs/USAGE.md) for the full walkthrough,
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for how it's wired, and
[`docs/PUBLISH.md`](docs/PUBLISH.md) to publish it to the Railway marketplace.

## Layout

```
opencode-railway/
├── opencode-devbox/        ← the deployable (Railway root directory)
│   ├── Dockerfile
│   ├── entrypoint.sh
│   ├── railway.json
│   └── .env.example
├── docs/
└── assets/
```
