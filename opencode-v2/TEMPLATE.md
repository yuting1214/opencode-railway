# Deploy and Host OpenCode 2 on Railway

<p align="center">
  <img src="https://raw.githubusercontent.com/yuting1214/opencode-railway/147398795a20abc1d5ccf4669aa94594d0593f2b/assets/v2-session.png" alt="The OpenCode 2 web app on this template: an agent touring a repo on the free DeepSeek model" width="860">
</p>

[OpenCode](https://opencode.ai) is **the open-source AI coding agent** — it reads your repo, writes
and edits code, runs commands, and opens pull requests. **OpenCode 2** rebuilt it around one shared
server that the terminal, web and desktop apps all connect to. This template runs that server on
Railway: the **OpenCode 2 web app** on a public domain — and when you'd rather work in a terminal,
**your own OpenCode 2** or **`railway ssh`** connects to the same server and the same sessions.

## About Hosting OpenCode 2

One container runs **`opencode serve`** — OpenCode 2's server, which serves both the web app and
its API — on a public Railway domain, with every API call password-protected. A `/workspace` volume
keeps sessions, credentials and repos across redeploys; `git` + the GitHub CLI are preinstalled.

**Three ways in, one set of sessions** — start a task in one, pick it up in another:

- **Browser** — open the public URL, choose **Use password**, and paste the password from the
  deploy logs (`[boot] opencode web auth → …`) or the `OPENCODE_PASSWORD` variable. It is
  generated for your deploy.
- **Your own OpenCode 2** — on any machine with OpenCode 2 installed:
  `OPENCODE_PASSWORD=<password> opencode --server https://<your-domain>`. The TUI runs locally;
  the agent, repos and sessions stay in the cloud. It needs OpenCode 2 (a 1.x client cannot talk
  to a 2.x server), ideally the same version the box runs.
- **`railway ssh`** — then `opencode` for the TUI, with a real shell, `git` and `gh` on the box
  beside it.

Deploys come from a **prebuilt, version-pinned image** — no build to sit through, and every deploy
runs the exact OpenCode 2 build that was tested. OpenCode 2 is moving fast; pinning is what keeps a
bad upstream release from breaking your deploy. You can **start coding free with no API key**:
OpenCode's free models (such as DeepSeek) work as soon as the box boots.

<p align="center">
  <img src="https://raw.githubusercontent.com/yuting1214/opencode-railway/147398795a20abc1d5ccf4669aa94594d0593f2b/assets/v2-review.png" alt="OpenCode 2 Review panel: side-by-side diff of an edit the agent made" width="860">
</p>

Every edit the agent makes shows up in the web app's **Review** panel as a side-by-side diff, next
to the conversation that produced it, alongside a file browser for the repo.

## Why Deploy OpenCode 2 on Railway?

- **OpenCode 2 from day one** — the new shared-server architecture and redesigned web app, pinned to
  a tested build and upgraded deliberately, not whatever was published an hour ago.
- **Start free, no API key** — OpenCode's free models (such as DeepSeek) work out of the box. The
  free lineup is set by OpenCode and changes over time.
- **Review every change in the browser** — the Review panel shows each file the agent touched as a
  side-by-side diff, so you can check its work from any device.
- **Bring your own terminal** — point your local OpenCode 2 at the box with `--server`: your TUI,
  a cloud agent, no SSH required.
- **A real shell when you need one** — `railway ssh` gives you the TUI plus a shell, `git` and `gh`
  on the box itself.
- **One set of sessions everywhere** — browser, your own OpenCode 2 and SSH all attach to the same
  server: same sessions, same history, same repos.
- **Your laptop can sleep** — the agent lives in the cloud; reopen the link, reconnect your TUI or
  SSH back in and pick up where you left off.
- **Secure by default** — the API refuses every call, from the browser or a remote TUI, without the
  password generated for your deploy; SSH is Railway-authenticated.
- **Provider-agnostic** — add an Anthropic, OpenAI or OpenRouter key, or `opencode auth login` to
  anything OpenCode supports.

## Common Use Cases

- **Try OpenCode 2** without installing anything — free models, one click.
- **A cloud coding-agent workstation** you drive from a browser, your own OpenCode 2, or SSH.
- **Long-running refactors or migrations** that keep going regardless of your local machine.

## Dependencies for OpenCode 2 Hosting

- **No API key required to start** — OpenCode's free models work out of the box. For premium
  models, add `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, or `OPENROUTER_API_KEY` (all on the deploy
  form), or run `opencode auth login` after connecting.
- Nothing to install for the web app — just a browser. Your own terminal needs **OpenCode 2**
  plus the URL and password; `railway ssh` needs the **Railway CLI**.

The box also honours **`GITHUB_TOKEN`**, **`GIT_USER_NAME`** and **`GIT_USER_EMAIL`** if you add
them under **Variables** later. It generates its own SSH key on first boot
(`cat ~/.ssh/id_ed25519.pub` → add it to GitHub).

### Deployment Dependencies

- [OpenCode](https://github.com/anomalyco/opencode) — the open-source AI coding agent
  (`@opencode/cli` on npm).
- [GitHub CLI](https://cli.github.com/) — bundled so the agent can manage issues and pull requests.

### Why This Template?

OpenCode 2 on one server you can reach three ways — the web app, your own OpenCode 2 terminal, or
SSH — all sharing the same sessions, pinned to a tested build, free models included. Prefer the 1.x line?
[OpenCode CLI + Web App](https://railway.com/deploy/opencode-cli-web-app?referralCode=jk_FgY&utm_medium=integration&utm_source=template&utm_campaign=generic).
Source and docs: [github.com/yuting1214/opencode-railway](https://github.com/yuting1214/opencode-railway).
