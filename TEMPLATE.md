# Deploy and Host OpenCode on Railway

[OpenCode](https://opencode.ai) is **the open-source AI coding agent** — a terminal-native
assistant (think Claude Code, but open and provider-agnostic) that reads your repo, writes and
edits code, runs commands, and opens pull requests. This template gives it a **persistent home in
the cloud**: a long-lived Railway container you `railway ssh` into and run `opencode` from, exactly
like sitting at a remote dev machine. Your auth, sessions, and cloned repos live on a Railway
volume, so they survive redeploys.

## About Hosting OpenCode

This template builds a small Node 20 image with the **OpenCode CLI**, `git`, the **GitHub CLI**,
and an SSH-ready environment, then keeps the container running so you can connect on demand. There
is no public web port to secure — you reach the box through Railway's own authenticated
`railway ssh`, drop into a login shell, and type `opencode` to enter the TUI. A `/workspace` volume
holds your repositories and OpenCode's credential store (`opencode auth login`), and your provider
API keys are injected into the SSH shell from Railway variables. Railway handles the build, restarts
on failure, and persists the volume across deploys.

## Why Deploy OpenCode on Railway?

- **Your laptop can sleep** — the agent and its session live in the cloud; reconnect anytime with
  `railway ssh` and pick up where you left off.
- **Persistent by default** — repos and OpenCode auth sit on a Railway volume, so a redeploy never
  logs you out or loses your working tree.
- **No exposed surface** — access is through Railway's authenticated SSH, not a public HTTP port,
  so there's nothing extra to lock down.
- **Provider-agnostic** — bring an Anthropic, OpenAI, or OpenRouter key (or `opencode auth login`
  to anything OpenCode supports); switch models without rebuilding.
- **Real dev tooling included** — `git` and the GitHub CLI ship in the image, so the agent can
  clone, commit, and open PRs from inside the box.

## Common Use Cases

- **A remote coding-agent workstation** you SSH into from any machine to run OpenCode interactively.
- **Long-running refactors or migrations** that should keep going regardless of your local machine.
- **A shared, reproducible agent environment** with pinned tooling for working across repositories.

## Dependencies for OpenCode Hosting

- A provider **API key** for at least one model backend — `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`,
  or `OPENROUTER_API_KEY` (or run `opencode auth login` after connecting).
- The **Railway CLI** on your local machine to connect: `railway ssh`.

### Deployment Dependencies

- [OpenCode](https://github.com/anomalyco/opencode) — the open-source AI coding agent
  (`opencode-ai` on npm).
- [GitHub CLI](https://cli.github.com/) — bundled so the agent can manage issues and pull requests.

### Why This Template?

It packages OpenCode as a one-click, persistent **remote dev box** — SSH in, type `opencode`, and
use it like any terminal AI agent, with credentials and repos that outlive every redeploy. Source
and docs: <https://github.com/yuting1214/opencode-railway>.
