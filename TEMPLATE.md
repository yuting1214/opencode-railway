# Deploy and Host OpenCode on Railway

[OpenCode](https://opencode.ai) is **the open-source AI coding agent** — an AI assistant (think
Claude Code, but open and provider-agnostic) that reads your repo, writes and edits code, runs
commands, and opens pull requests. This template gives it a **persistent home in the cloud** with
**two ways in from one service**: a **browser web UI** on a public Railway domain, and the
**`opencode` TUI** over `railway ssh` — both backed by the same `/workspace`, auth, and repos,
which live on a Railway volume and survive redeploys.

## About Hosting OpenCode

This template builds a small Node 20 image with the **OpenCode CLI**, `git`, and the **GitHub CLI**.
The container's main process is **`opencode web`**, served on the **public Railway domain** and
protected with **HTTP basic auth** — a password is auto-generated on first boot and persisted on
the volume (it's printed in the deploy logs; set `OPENCODE_SERVER_PASSWORD` to choose your own).
For terminal work, `railway ssh` into the very same container and type `opencode` to get the TUI.
A `/workspace` volume holds your repositories and OpenCode's credential store (`opencode auth
login`), and your provider API keys are injected from Railway variables. Railway handles the build,
provisions TLS + the domain, restarts on failure, and persists the volume across deploys.

## Why Deploy OpenCode on Railway?

- **Two surfaces, one box** — a shareable browser **web UI** *and* the terminal **TUI** over SSH,
  both driving the same workspace, auth, and sessions.
- **Your laptop can sleep** — the agent lives in the cloud; reopen the web link or `railway ssh`
  back in and pick up where you left off.
- **Secure by default** — the public web UI is behind HTTP basic auth with an auto-generated
  password; SSH access is Railway-authenticated.
- **Persistent** — repos and OpenCode auth sit on a Railway volume, so a redeploy never logs you
  out or loses your working tree.
- **Provider-agnostic** — bring an Anthropic, OpenAI, or OpenRouter key (or `opencode auth login`
  to anything OpenCode supports); switch models without rebuilding.
- **Real dev tooling included** — `git` and the GitHub CLI ship in the image, so the agent can
  clone, commit, and open PRs from inside the box.

## Common Use Cases

- **A cloud coding-agent workstation** reachable from a browser link *or* the terminal.
- **Long-running refactors or migrations** that should keep going regardless of your local machine.
- **A shared, reproducible agent environment** with pinned tooling for working across repositories.

## Dependencies for OpenCode Hosting

- A provider **API key** for at least one model backend — `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`,
  or `OPENROUTER_API_KEY` (or run `opencode auth login` after connecting).
- The **Railway CLI** for terminal/TUI access (`railway ssh`); the web UI needs only a browser.

### Deployment Dependencies

- [OpenCode](https://github.com/anomalyco/opencode) — the open-source AI coding agent
  (`opencode-ai` on npm).
- [GitHub CLI](https://cli.github.com/) — bundled so the agent can manage issues and pull requests.

### Why This Template?

It packages OpenCode as a one-click, persistent **remote dev box** — SSH in, type `opencode`, and
use it like any terminal AI agent, with credentials and repos that outlive every redeploy. Source
and docs: <https://github.com/yuting1214/opencode-railway>.
