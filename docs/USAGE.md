# Usage — SSH in and run OpenCode

This box is meant to be used like a remote terminal: connect with `railway ssh`, type `opencode`,
work, disconnect. Nothing about your session is lost when you leave — it all lives on the
`/workspace` volume.

## 1. One-time setup

After deploying the template:

1. **Attach a volume** at mount path `/workspace` (the template includes it; confirm it's present).
2. **Set provider key(s)** in the service **Variables** tab — at least one of:
   - `ANTHROPIC_API_KEY`
   - `OPENAI_API_KEY`
   - `OPENROUTER_API_KEY`
3. *(Optional)* `GITHUB_TOKEN` to auto-authenticate `gh` and HTTPS git, and `GIT_USER_NAME` /
   `GIT_USER_EMAIL` for commit identity.

Install the Railway CLI locally if you haven't:

```bash
npm i -g @railway/cli      # or: brew install railway
railway login
```

## 2. Connect

```bash
railway link        # select this project + the opencode service
railway ssh         # opens a shell inside the container
```

You'll land in `/workspace/repos`. Then just:

```bash
opencode            # launches the OpenCode TUI
```

Switch agent modes with **Tab** (build / plan), and use OpenCode normally.

## 3. Working with repos

Clone into `/workspace/repos` so your code persists across redeploys:

```bash
cd /workspace/repos
git clone git@github.com:you/your-repo.git     # SSH (uses the box's generated key)
# or
gh repo clone you/your-repo                     # HTTPS via gh, if GITHUB_TOKEN is set
```

**Add the box's SSH key to GitHub** (for `git@github.com:` clones over SSH):

```bash
cat ~/.ssh/id_ed25519.pub      # copy this into GitHub → Settings → SSH keys
```

## 4. Authentication options

You have two equivalent ways to give OpenCode a model:

- **Railway variables** (set in step 1) — mirrored into the SSH shell automatically.
- **`opencode auth login`** — run it once inside the box; credentials are written to
  `~/.local/share/opencode/auth.json`, which is symlinked onto the `/workspace` volume and reused on
  every reconnect.

## 5. Headless / one-shot

You don't have to use the TUI — `run` executes a single prompt and exits:

```bash
opencode run "summarize what this repo does and list the entry points"
```

## 6. Housekeeping

```bash
opencode upgrade        # update OpenCode to the latest version
df -h /workspace        # check volume usage
```

## Notes & gotchas

- **Always work under `/workspace`.** Files written elsewhere (e.g. `/root`, `/tmp`) are **lost on
  redeploy** — only the volume persists.
- `railway ssh` runs a login shell, which sources `/etc/profile.d/00-opencode-env.sh`; that's how
  your provider keys reach `opencode`. If a key seems missing, re-check the Variables tab and
  reconnect.
- **Web UI:** the container's main process is `opencode web` on the public Railway domain, behind
  HTTP basic auth (user `opencode`, password from `OPENCODE_SERVER_PASSWORD` — auto-generated on
  first boot and printed in the deploy logs / stored at `/workspace/.opencode-web-password`). The
  TUI over SSH and the web UI run in the **same** container and share `/workspace`, auth, and repos.
- **Rotating the web password:** set `OPENCODE_SERVER_PASSWORD` in Variables (redeploys), or edit
  `/workspace/.opencode-web-password` and restart. To make the box private again, remove the
  service's public domain in Railway → only `railway ssh` remains.
