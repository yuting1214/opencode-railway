# OpenCode Devbox on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/REPLACE_ME)

A one-click, **persistent remote dev box for [OpenCode](https://opencode.ai)** — the open-source AI
coding agent. Deploy it, `railway ssh` in, type `opencode`, and use it just like Claude Code or any
other terminal AI agent. Your auth, sessions, and repos live on a Railway volume and survive
redeploys.

> No public port, no web server to secure — you reach the box through Railway's authenticated
> `railway ssh`.

---

## Quick start

1. **Deploy** this template (button above) and add a **Volume** mounted at `/workspace`.
2. Set at least one provider key in the service **Variables** (`ANTHROPIC_API_KEY`,
   `OPENAI_API_KEY`, or `OPENROUTER_API_KEY`). Optional: `GITHUB_TOKEN`, `GIT_USER_NAME`,
   `GIT_USER_EMAIL`.
3. From your machine:
   ```bash
   railway link            # pick this project/service
   railway ssh             # drop into the container
   opencode                # enter the OpenCode TUI
   ```
4. Clone a repo and start coding — work in `/workspace/repos` so it persists:
   ```bash
   cd /workspace/repos && git clone git@github.com:you/your-repo.git
   ```

If you'd rather not use environment variables, just run `opencode auth login` once inside the box;
the credentials are written to the `/workspace` volume and reused on every reconnect.

## What's in the box

| Tool | Why |
|------|-----|
| **OpenCode CLI** (`opencode-ai`) | the AI coding agent — TUI (`opencode`) and headless (`opencode run "…"`) |
| **git** + **SSH key** | clone/commit; an ed25519 key is generated on first boot (see `~/.ssh/id_ed25519.pub`) |
| **GitHub CLI** (`gh`) | `gh pr create`, `gh issue`, etc. — auto-authenticated from `GITHUB_TOKEN` |

## How it works

- The container's entrypoint sets up the `/workspace` volume and **symlinks OpenCode's config/state
  and your `~/.ssh` onto it**, so credentials and sessions persist across redeploys.
- `CMD ["sleep", "infinity"]` keeps the service alive so you can connect any time.
- Provider keys set as Railway variables are mirrored into the SSH login shell, so `opencode` picks
  them up immediately.

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
