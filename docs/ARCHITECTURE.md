# Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  YOUR MACHINE                                                     │
│   browser ──────────────┐            railway ssh ─────────┐       │
└─────────────────────────┼───────────────────────────────-─┼──────┘
        Railway public domain (HTTPS)      Railway authenticated SSH
                          ▼                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│  RAILWAY CONTAINER  (one service)                                │
│                                                                  │
│   CMD ▶ opencode web --hostname 0.0.0.0 --port $PORT             │
│            └─ basic auth (OPENCODE_SERVER_PASSWORD) ─ web UI      │
│                                                                  │
│   railway ssh ▶ login shell ▶ /etc/profile.d/00-opencode-env.sh  │
│                     └─ $ opencode  ▶ TUI / `opencode run`         │
│                                                                  │
│   tooling: opencode-ai · git · gh · openssh-client               │
│                                                                  │
│   symlinks ──▶  /workspace volume (persists across deploys)      │
│     ~/.local/share/opencode  → /workspace/opencode/data          │
│     ~/.config/opencode       → /workspace/opencode/config        │
│     ~/.ssh                   → /workspace/.ssh                   │
│     repos                     /workspace/repos                   │
└──────────────────────────────────────────────────────────────────┘
```

## Components

- **Dockerfile** — `node:20-slim` + `git`, `gh`, `openssh-client`, and the OpenCode CLI installed
  globally via npm. `ENTRYPOINT` runs the boot script; `CMD` runs `opencode web` (the main process,
  on `$PORT`, default 8080) so the service stays up and the browser UI is served.
- **entrypoint.sh** — runs on every boot:
  1. Creates the `/workspace` layout and symlinks OpenCode's state/config and `~/.ssh` onto it.
  2. Generates an ed25519 SSH key on first boot and trusts `github.com`.
  3. Authenticates `gh` from `GITHUB_TOKEN` (if set) and sets git identity.
  4. Resolves the web UI basic-auth password (`OPENCODE_SERVER_PASSWORD`, generated + persisted on
     the volume if unset) and writes `/etc/profile.d/00-opencode-env.sh` so SSH login shells export
     the provider keys / web creds and land in `/workspace/repos`.
- **railway.json** — Dockerfile builder + restart-on-failure.
- **Public domain** — Railway maps it to `$PORT` (the `opencode web` listener).
- **Volume** — mounted at `/workspace`; the single source of persistent state.

## Design choices

- **Two surfaces, one service.** `opencode web` is the always-on main process behind the public
  domain; `railway ssh` reaches the same container for the TUI. Both share `/workspace`, auth, and
  repos, so there's no second service to run or sync.
- **Secure public surface.** The web UI is protected by HTTP basic auth with an auto-generated,
  volume-persisted password; SSH is Railway-authenticated. Drop the public domain to go SSH-only.
- **Volume-first persistence.** All mutable state is symlinked onto `/workspace`, so redeploys keep
  your auth, sessions, and repos.
- **No tmux / no Tailscale.** Kept minimal — the web server + `railway ssh` are the only access paths.
- **Provider-agnostic auth.** Keys can come from Railway variables *or* `opencode auth login`; the
  image hardcodes no provider.
