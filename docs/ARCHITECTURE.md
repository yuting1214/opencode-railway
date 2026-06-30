# Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  YOUR MACHINE (can disconnect anytime)                      │
│                                                             │
│   railway ssh ───────────────┐                              │
└──────────────────────────────┼──────────────────────────────┘
                               │ Railway authenticated SSH
                               ▼
┌─────────────────────────────────────────────────────────────┐
│  RAILWAY CONTAINER  (CMD: sleep infinity — always running)  │
│                                                             │
│   login shell ──sources──▶ /etc/profile.d/00-opencode-env.sh│
│        │                     (exports ANTHROPIC/OPENAI/...)  │
│        ▼                                                     │
│   $ opencode  ───▶  OpenCode TUI / `opencode run`           │
│                                                             │
│   tooling: opencode-ai · git · gh · openssh-client          │
│                                                             │
│   symlinks ──▶  /workspace volume (persists across deploys) │
│     ~/.local/share/opencode  → /workspace/opencode/data     │
│     ~/.config/opencode       → /workspace/opencode/config   │
│     ~/.ssh                   → /workspace/.ssh              │
│     repos                     /workspace/repos              │
└─────────────────────────────────────────────────────────────┘
```

## Components

- **Dockerfile** — `node:20-slim` + `git`, `gh`, `openssh-client`, and the OpenCode CLI installed
  globally via npm. `ENTRYPOINT` runs the boot script; `CMD` is `sleep infinity` so the service
  stays up and connectable.
- **entrypoint.sh** — runs on every boot:
  1. Creates the `/workspace` layout and symlinks OpenCode's state/config and `~/.ssh` onto it.
  2. Generates an ed25519 SSH key on first boot and trusts `github.com`.
  3. Authenticates `gh` from `GITHUB_TOKEN` (if set) and sets git identity.
  4. Writes `/etc/profile.d/00-opencode-env.sh` so SSH login shells export the provider keys and
     land in `/workspace/repos`.
- **railway.json** — Dockerfile builder + restart-on-failure.
- **Volume** — mounted at `/workspace`; the single source of persistent state.

## Design choices

- **SSH-only, no public port.** Access goes through Railway's authenticated `railway ssh`, so there's
  no exposed HTTP surface to secure. (OpenCode *can* run as an HTTP server — see the note in
  `USAGE.md` — but that's deliberately out of scope here.)
- **Volume-first persistence.** All mutable state is symlinked onto `/workspace`, so redeploys keep
  your auth, sessions, and repos.
- **No tmux / no Tailscale.** Kept intentionally minimal: a single connection method (`railway ssh`)
  and the smallest image that still has real dev tooling.
- **Provider-agnostic auth.** Keys can come from Railway variables *or* `opencode auth login`; the
  image hardcodes no provider.
