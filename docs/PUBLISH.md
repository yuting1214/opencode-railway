# Publishing to the Railway marketplace

Requires **Railway CLI ≥ 5.x** (`railway templates` did not exist in 4.x).
Verify with `railway --version`; upgrade with `brew upgrade railway` if needed.

## Prerequisites

1. **Push this repo to a PUBLIC GitHub repo.** Deployers clone it, and the marketplace card image
   must be reachable. Replace the `REPLACE_ME` placeholders in `README.md` and `TEMPLATE.md` with
   the real repo URL.
2. **A project with a service linked to that GitHub repo**, with **root directory set to
   `opencode`** (this is the monorepo subdir that holds the Dockerfile). A service deployed
   only via local `railway up` has no git source to template.
3. **A volume** on that service mounted at `/workspace`.
4. Deploy once and verify cold-start works (see checklist below).

## Create the draft

```bash
railway link                       # link the project whose service points at the repo
railway templates create --json    # → returns an unpublished draft { id }; note the id
```

In a TTY this opens the dashboard template editor — confirm the captured variables match the
spec below and that the volume mount at `/workspace` is present.

> Note: `templateGenerate` captures variable *names* only, never values — so no secrets leak.

> ⚠️ **This document used to claim six provider/git variables were captured. They were not.**
> Verified 2026-09-06 against the live template: the only captured variables were
> `OPENCODE_SERVER_USERNAME` and `OPENCODE_SERVER_PASSWORD`, **both mandatory, with no default
> and no description**. So the deploy form demanded a password the entrypoint would have
> generated anyway, and offered no way to set a model key — deployers landed in a box that
> could not reach a provider. Read the spec below, not the old prose.
>
> Check what is actually published, never what the docs assert:
> ```
> GraphQL template(code:"opencode-cli-web-app"){ serializedConfig }
> ```

### Target variable spec

| Variable | Optional | Default | Description |
|---|---|---|---|
| `OPENCODE_SERVER_USERNAME` | yes | `opencode` | Username for the web UI login |
| `OPENCODE_SERVER_PASSWORD` | no | `${{secret(32, "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789")}}` | Password for the web UI — generated for you |
| `ANTHROPIC_API_KEY` | yes | — | Claude models. Or run `opencode auth login` in the box |
| `OPENAI_API_KEY` | yes | — | OpenAI models |
| `OPENROUTER_API_KEY` | yes | — | OpenRouter models |
| `GITHUB_TOKEN` | yes | — | Authenticates `gh` and HTTPS git on boot |
| `GIT_USER_NAME` | yes | — | Commit author name |
| `GIT_USER_EMAIL` | yes | — | Commit author email |

Two rules behind that table:

- **`${{secret(32, …)}}` for the password.** Railway generates a strong value at deploy time, so
  the field is prefilled rather than a blank mandatory box. Do **not** copy the competitor
  pattern of `defaultValue: "changeme"`.
- **Every variable carries a description.** The field-leading competitor does this and we did not;
  a blank unexplained field is the cheapest possible reason for someone to abandon a deploy.

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

## Switching the template to the prebuilt image

The template currently deploys from this repo, so **every deploy rebuilds the image** — apt, the
GitHub CLI apt repo, npm and the floating `node:20-slim` tag are all live failure surfaces on a
stranger's deploy, and they wait through a full build (~4 min measured) before seeing anything.
The two highest-health competitors deploy from a prebuilt image instead.

Neither of them is *frozen*, and neither should we be: both reference a **tagged** image plus
Railway `autoUpdates: {type: "patch"}`, which keeps the auto-update property this template
advertises. Copy that.

1. Tag a release so `release.yml` publishes the image:
   ```bash
   git tag v1.0.0 && git push origin v1.0.0
   ```
2. **Make the GHCR package public** — Packages → `opencode-railway` → Package settings → Change
   visibility. Deployers cannot pull a private image; skipping this fails every deploy.
3. Verify the published image runs standalone before it reaches anyone:
   ```bash
   docker run --rm -p 8080:8080 -e OPENCODE_SERVER_PASSWORD=test \
     ghcr.io/yuting1214/opencode-railway:1.0
   curl -o /dev/null -w '%{http_code}\n' localhost:8080/site.webmanifest   # expect 200
   ```
4. Point a scratch service at the image, attach a `/workspace` volume and a domain, deploy, and
   run the cold-start checklist above.
5. Regenerate and republish from that project (see the next section — this is the same single
   republish that fixes the variables; do not do it twice).

Keep this repo as the buildable source of truth: it is what CI tests, what `release.yml` builds,
and what the docs describe.

## Republishing (variables, source model, or title)

`railway templates publish` only updates **metadata** — readme, description, image, category. It
cannot change captured variables or the source model. Those require
**regenerate → publish new → delete old**, which reclaims the same slug so the deploy URL survives.

Because it is one operation, **batch the variable spec and the image switch into a single
republish**. Do not spend two slug-risking operations on them.

Guard rails, in order:

```bash
# 1. The referral code lives ONLY in links you author — a copy rewrite silently drops it.
grep -rn "referralCode=jk_FgY" README.md TEMPLATE.md docs/ | wc -l

# 2. Confirm what is live right now, before changing anything.
#    GraphQL: template(code:"opencode-cli-web-app"){ serializedConfig }

# 3. Regenerate from the verified project, publish, then delete the old template.
railway templates create --json
railway templates publish <NEW_DRAFT_ID> --category AI/ML --readme-file TEMPLATE.md --json
railway templates delete <OLD_ID> --yes --json

# 4. Confirm the slug still resolves.
curl -s -o /dev/null -w '%{http_code}\n' https://railway.com/deploy/opencode-cli-web-app
```

- **Never retitle.** The title generates the slug; changing it kills every shared link.
- Express every value as a `${{...}}` reference — `templateGenerate` strips plain literals
  (`scripts/template-vars.py` in the `railway-templates` skill checks this).
- **Never read variables from the API and write them back**: the query returns them *resolved*, so
  a round-trip converts every reference into a literal.
- Verify cold-start on a real deploy **before** republishing, not after.

## Updating later

- **Metadata only** (description/readme/image/category):
  ```bash
  railway templates publish <id> --category AI/ML --readme-file TEMPLATE.md --json
  ```
- **Build changes** (Dockerfile/entrypoint): push to the GitHub repo — GitHub-sourced templates
  rebuild from it. Once on the prebuilt image, ship a tagged release instead and let Railway's
  `autoUpdates: patch` roll it out.
- **Renaming, captured variables, or the source model**: see *Republishing* above.

## Keeping the OpenCode pin current

The agent is pinned via `ARG OPENCODE_VERSION` in `opencode/Dockerfile`. That pin is what
keeps deploy-success stable — unpinned, every deploy rebuilt against whatever npm published
that hour (~300 publishes/30 days, ~20 stable releases/month), so one bad upstream release
crash-looped `opencode web` for everyone deploying that day.

**But a pin nobody bumps is its own failure mode.** Check every week or two:

```bash
npm view opencode-ai version                       # current stable
gh api repos/anomalyco/opencode/releases --jq '.[0:5][] | .tag_name + "  " + .published_at'
```

To bump:

1. Edit `ARG OPENCODE_VERSION` in `opencode/Dockerfile`.
2. `docker build` locally — the layer runs `opencode --version`, so a yanked or broken
   version fails the **build** instead of shipping an image that crash-loops at runtime.
3. Deploy to a scratch project and run the cold-start checklist above.
4. Confirm `/site.webmanifest` still returns **200 unauthenticated** — it is the
   healthcheck path, and it is the only unauthenticated endpoint on the service. If an
   upstream release ever changes or protects it, **every deploy will fail**, so re-check
   this on every bump.
5. Commit. No republish needed — this is a build change.

## Monitoring adoption

Prefer the CLI/GraphQL over scraping (the public page renders stats client-side):

```bash
# yours — deploys / active / health / payout
python3 ~/.claude/skills/railway-templates/scripts/template-metrics.py --detail
# rivals — deploymentCount + healthScore
railway templates search "opencode" --json --limit 20
```

Legacy fallback — scrape the public page:

```
WebFetch https://railway.com/deploy/<code>
  → parse: N total projects · N active · N recent · N% deploy success
```
