# Deploy and Host OpenCode on Railway

<p align="center">
  <img src="https://raw.githubusercontent.com/yuting1214/opencode-railway/__HERO_SHA__/assets/hero.png" alt="OpenCode — open-source AI coding agent" width="720">
</p>

[OpenCode](https://opencode.ai) is **the open-source AI coding agent** (think Claude Code, but open
and provider-agnostic) — it reads your repo, writes and edits code, runs commands, and opens pull
requests. This template runs it as a Railway service with **two front doors**: a **browser web UI**
on a public domain, and the **`opencode` terminal UI** over `railway ssh` — both sharing one
`/workspace`, auth, and repos that persist on a volume.

## About Hosting OpenCode

One container runs **`opencode web`** on a public, password-protected Railway domain; `railway ssh`
into the same container gives you the **`opencode` TUI**. A `/workspace` volume keeps your repos and
OpenCode credentials across redeploys, and `git` + the GitHub CLI are preinstalled for clone /
commit / PR work. Railway builds the image, provisions TLS and the domain, and restarts on failure.
Bring an LLM API key (or run `opencode auth login` after connecting).

## Why Deploy OpenCode on Railway?

- **Two front doors, one service** — a shareable browser **web UI** *and* the terminal **TUI** over
  SSH, both driving the same workspace, auth, and sessions.
- **Your laptop can sleep** — the agent lives in the cloud; reopen the web link or `railway ssh`
  back in and pick up where you left off.
- **Secure by default** — the public web UI is behind HTTP basic auth; SSH is Railway-authenticated.
- **Persistent** — repos and OpenCode auth sit on a volume, so a redeploy never logs you out or
  loses your working tree.
- **Provider-agnostic** — bring an Anthropic, OpenAI, or OpenRouter key (or `opencode auth login`
  to anything OpenCode supports); switch models without rebuilding.

## Common Use Cases

- **A cloud coding-agent workstation** reachable from a browser link *or* the terminal.
- **Long-running refactors or migrations** that keep going regardless of your local machine.
- **A shared, reproducible agent environment** with pinned tooling across repositories.

## Dependencies for OpenCode Hosting

- A provider **API key** for at least one model backend — `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`,
  or `OPENROUTER_API_KEY` (or run `opencode auth login` after connecting).
- The **Railway CLI** for terminal/TUI access (`railway ssh`); the web UI needs only a browser.

### Deployment Dependencies

- [OpenCode](https://github.com/anomalyco/opencode) — the open-source AI coding agent
  (`opencode-ai` on npm).
- [GitHub CLI](https://cli.github.com/) — bundled so the agent can manage issues and pull requests.

### Why This Template?

One click gives you OpenCode with two front doors — a browser web UI and an SSH terminal — over a
single service, with credentials and repos that outlive every redeploy. Source and docs:
<https://github.com/yuting1214/opencode-railway>.
