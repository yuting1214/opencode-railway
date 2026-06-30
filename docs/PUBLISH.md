# Publishing to the Railway marketplace

Requires **Railway CLI ≥ 5.x** (`railway templates` did not exist in 4.x).
Verify with `railway --version`; upgrade with `brew upgrade railway` if needed.

## Prerequisites

1. **Push this repo to a PUBLIC GitHub repo.** Deployers clone it, and the marketplace card image
   must be reachable. Replace the `REPLACE_ME` placeholders in `README.md` and `TEMPLATE.md` with
   the real repo URL.
2. **A project with a service linked to that GitHub repo**, with **root directory set to
   `opencode-devbox`** (this is the monorepo subdir that holds the Dockerfile). A service deployed
   only via local `railway up` has no git source to template.
3. **A volume** on that service mounted at `/workspace`.
4. Deploy once and verify cold-start works (see checklist below).

## Create the draft

```bash
railway link                       # link the project whose service points at the repo
railway templates create --json    # → returns an unpublished draft { id }; note the id
```

In a TTY this opens the dashboard template editor — confirm the captured variables
(`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `GITHUB_TOKEN`, `GIT_USER_NAME`,
`GIT_USER_EMAIL`) are all **optional**, and that the volume mount at `/workspace` is present.

> Note: `templateGenerate` captures variable *names* only, never values — so no secrets leak.

## Publish

```bash
railway templates publish <DRAFT_ID> \
  --category AI/ML \
  --description "SSH into a persistent Railway box and run the OpenCode AI agent" \
  --readme-file TEMPLATE.md \
  --image https://raw.githubusercontent.com/yuting1214/opencode-railway/<sha>/assets/card.png \
  --json
```

- **Category** must be `AI/ML` (from the fixed list).
- **Description** must be ≤ 75 characters.
- Use a **commit-pinned** raw image URL (`/<sha>/`, not `/main/`) so the card resolves immediately.

The command returns the published `{ code }`; the deploy URL is then
`https://railway.com/deploy/<code>` and the button is:

```md
[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/<code>)
```

## Cold-start verification checklist (before publishing)

- [ ] `railway ssh` connects and lands in `/workspace/repos`.
- [ ] `opencode --version` works; `opencode` opens the TUI.
- [ ] A provider key set in Variables is visible in the SSH shell (`echo $ANTHROPIC_API_KEY`).
- [ ] `opencode run "hello"` returns a model response.
- [ ] `git clone` of a repo into `/workspace/repos` succeeds; survives a redeploy.
- [ ] After a **redeploy**, `opencode auth login` state and cloned repos are still present.

## Updating later

- **Metadata only** (description/readme/image/category):
  ```bash
  railway templates publish <id> --category AI/ML --readme-file TEMPLATE.md --json
  ```
- **Build changes** (Dockerfile/entrypoint): push to the GitHub repo and redeploy the source service.
- **Renaming or changing captured variables**: regenerate from the updated project → publish new →
  delete old (reclaims the same slug, keeping the URL stable).

## Monitoring adoption

There is **no CLI/API** for template metrics. Scrape the public page:

```
WebFetch https://railway.com/deploy/<code>
  → parse: N total projects · N active · N recent · N% deploy success
```
