# Publishing to the Railway marketplace

This repo ships **two templates**, one per OpenCode major:

| | 1.x | OpenCode 2 |
|---|---|---|
| Deployable dir | `opencode/` | `opencode-v2/` |
| Marketplace readme | `TEMPLATE.md` (repo root) | `opencode-v2/TEMPLATE.md` |
| Slug | `opencode-cli-web-app` | `opencode-2-cli-web-app` |
| Template source | image `ghcr.io/yuting1214/opencode-railway:1.0` | image `ghcr.io/yuting1214/opencode-railway:2.0` |
| Git tags | `v1.*` | `v2.*` |
| npm package | `opencode-ai` | `@opencode/cli` |
| Healthcheck | `/site.webmanifest` (only unauthenticated path) | `/` (web shell is public; `/api/*` is protected) |
| Password variable | `OPENCODE_SERVER_PASSWORD` | `OPENCODE_PASSWORD` |

Both templates deploy from a **prebuilt, version-pinned image**, not from the repo. A deploy is
pull-and-start: no apt, npm or base-image tag can break a stranger's deploy. The repo is still the
buildable source of truth: it is what CI tests and what `release.yml` builds.

Requires **Railway CLI ≥ 5.x** (`railway templates` did not exist in 4.x).
Verify with `railway --version`; upgrade with `brew upgrade railway` if needed.

## Releasing an image

The **major of the git tag picks the template**:

```bash
git tag v1.0.2 && git push origin v1.0.2   # builds ./opencode    → :1.0.2 and :1.0
git tag v2.0.2 && git push origin v2.0.2   # builds ./opencode-v2 → :2.0.2 and :2.0
```

- The templates reference the **`:X.Y`** tag, so a patch release (`v1.0.x`, `v2.0.x`) reaches every
  new deploy with no republish. A new minor (`v1.1.0`) does **not**: it publishes `:1.1`, which no
  template points at until you republish with the new image.
- The GHCR package must stay **public**. Deployers cannot pull a private image, and every deploy
  would fail.
- Check the published image before anyone pulls it:
  ```bash
  docker run --rm -p 8080:8080 -e OPENCODE_SERVER_PASSWORD=test ghcr.io/yuting1214/opencode-railway:1.0
  curl -o /dev/null -w '%{http_code}\n' localhost:8080/site.webmanifest     # expect 200

  docker run --rm -p 8080:8080 -e OPENCODE_PASSWORD=test ghcr.io/yuting1214/opencode-railway:2.0
  curl -o /dev/null -w '%{http_code}\n' localhost:8080/                     # expect 200
  curl -o /dev/null -w '%{http_code}\n' localhost:8080/api/session          # expect 401
  ```

## Keeping the OpenCode pins current

Each Dockerfile pins `ARG OPENCODE_VERSION`. The pin is what keeps deploy success stable: unpinned,
every deploy got whatever npm published that hour, and one bad upstream release crash-looped the
server for everyone deploying that day. A pin nobody bumps is its own failure mode, so bumping is
automated:

- `update-upstream.yml` runs weekly, one matrix entry per template. Each opens a PR on its own
  branch (`chore/bump-opencode`, `chore/bump-opencode-v2`) and **never crosses a major**: a 2.x
  release on `opencode-ai` does not bump the 1.x template.
- **A bot-opened PR's CI run waits for approval** (`action_required`). Approve it in the Actions
  tab, or the PR sits green-less forever.
- The repo setting *Allow GitHub Actions to create and approve pull requests* must be on, or the
  workflow fails without opening a PR.

To land a bump:

1. Let CI pass on the bot PR. Both smoke jobs assert the image runs exactly the pinned version,
   the healthcheck path is 200 **without** credentials, and the API refuses anonymous callers.
   If an upstream release ever protects the healthcheck path, **every deploy fails**, and this
   is the check that catches it.
2. Read the upstream release notes for anything touching the web server, auth or the terminal.
   1.19.0, for example, tightened web-terminal security (origin checks and a connect token).
3. Deploy the branch's image to a scratch project and run the checklist below.
4. Merge, then tag a **patch** release for that major (see above).

## Cold-start verification checklist

Run it on a real Railway deploy, not only locally:

- [ ] Service comes up **Online** with the healthcheck green.
- [ ] The web UI loads on the public domain; a wrong password is refused.
- [ ] A free model answers with **no API key** set (`opencode models opencode` lists them).
- [ ] `railway ssh` lands in `/workspace/repos`, and `opencode` opens the TUI.
- [ ] A provider key set in Variables is visible in the SSH shell.
- [ ] A repo cloned into `/workspace/repos` and the login state survive a **redeploy**.

OpenCode 2 only:

- [ ] Open the domain in a **fresh browser profile**: "Connect to a server" → *Use password* →
      the **Start here** session is listed → open it → **New session** works.
      The web app keeps its project list in browser storage, so on a fresh browser New session
      does nothing until a project is open. The entrypoint seeds that session once per volume;
      this check proves it.
- [ ] The TUI over `railway ssh` shows the same sessions as the browser.
- [ ] A local OpenCode 2 attaches remotely:
      `OPENCODE_PASSWORD=… opencode --server https://<domain>`. A wrong password is refused.

## Test the template itself before announcing it

A scratch project built by hand proves the image. It does not prove the **template**: the
captured variables, defaults, volume and healthcheck. Deploy the published template the way a
stranger would, one click, into a throwaway project:

```bash
railway deploy -t <slug>          # or the GraphQL mutation templateDeployV2
```

Then run the checklist above against it and delete the project.

## Creating a template

1. **A project with a service whose source is the image** (`:1.0` or `:2.0`), a **volume at
   `/workspace`**, and a public domain. Verify it with the checklist.
2. Generate the draft from the linked project:
   ```bash
   railway link
   railway templates create --json    # → unpublished draft { id }
   ```
   **The project name becomes the title, and the title becomes the slug.** Name the project
   right before generating.
3. **Set the variables in the dashboard template editor.** There is no API or CLI for a template
   variable's default, description or optional flag. `templateGenerate` keeps `${{...}}`
   references and strips plain literals.

   | Variable | Optional | Default | Description |
   |---|---|---|---|
   | Password (`OPENCODE_SERVER_PASSWORD` / `OPENCODE_PASSWORD`) | no | `${{secret(32, "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789")}}` | Generated for you; where to find it |
   | `OPENCODE_SERVER_USERNAME` (1.x only) | yes | `opencode` | Username for the web UI |
   | `ANTHROPIC_API_KEY` | yes | — | Optional; free models need no key |
   | `OPENAI_API_KEY` | yes | — | Optional; free models need no key |
   | `OPENROUTER_API_KEY` | yes | — | Optional; free models need no key |

   - Use `${{secret(32, …)}}` for the password, so the field is prefilled with a strong value.
     Never a literal like `changeme`.
   - **Every variable carries a description.** A blank, unexplained field is the cheapest reason
     for someone to abandon a deploy.
4. Publish:
   ```bash
   railway templates publish <DRAFT_ID> \
     --category AI/ML \
     --description "<≤ 75 characters>" \
     --readme-file opencode-v2/TEMPLATE.md \
     --image https://raw.githubusercontent.com/yuting1214/opencode-railway/<sha>/assets/<card>.png \
     --json
   ```
   - The readme must follow Railway's fixed schema (`# Deploy and Host … on Railway`,
     `## About Hosting …`, `## Why Deploy …`, `## Common Use Cases`, `## Dependencies for …`,
     `### Deployment Dependencies`).
   - The command returns the `{ code }`. The deploy URL is `https://railway.com/deploy/<code>`.
5. Run the one-click test above.

## Images in the readme and card

- Use **commit-pinned** raw URLs (`raw.githubusercontent.com/…/<sha>/assets/…`), not `/main/`.
- **The commit must be on `main`.** A rebase-merge rewrites SHAs, so an image pinned to a PR
  branch's commit points at an orphaned commit after merge. Pin to the SHA as it exists on `main`
  (`git log --format=%H -1 -- assets/<file>`), then republish.

## Updating a live template

| Change | How |
|---|---|
| OpenCode bump, entrypoint or Dockerfile fix | Patch tag (`v1.0.x` / `v2.0.x`). No republish. |
| Readme, description, card image, category | `railway templates publish <id> …` (metadata only) |
| Variable defaults, descriptions, optional flags | Dashboard template editor |
| Adding/removing variables, the image ref, the title | Regenerate → publish new → delete old (below) |

Metadata-only update:

```bash
railway templates publish <id> --category AI/ML --readme-file TEMPLATE.md --json
```

**The page is cached for up to an hour** (CDN and browser `max-age=3600`). A publish that
"didn't take" usually did. Check the source of truth instead of the page:

```
GraphQL template(code:"<slug>"){ serializedConfig }
```

### Regenerate → publish new → delete old

`publish` cannot change captured variables or the source. Those need a new template, and deleting
the old one frees its slug for the new one. Batch every such change into **one** republish.

```bash
# 1. The referral code lives ONLY in links you author; a copy rewrite silently drops it.
grep -rn "referralCode=jk_FgY" README.md TEMPLATE.md opencode-v2/TEMPLATE.md

# 2. Record what is live now (GraphQL serializedConfig above).

# 3. Regenerate from the verified project, publish, delete the old template.
railway templates create --json
railway templates publish <NEW_DRAFT_ID> --category AI/ML --readme-file <readme> --json
railway templates delete <OLD_ID> --yes --json

# 4. Confirm the slug still resolves.
curl -s -o /dev/null -w '%{http_code}\n' https://railway.com/deploy/<slug>
```

- **Never retitle.** The title generates the slug; a new title kills every shared link.
- **Never read variables from the API and write them back.** The query returns them *resolved*,
  so a round-trip turns every `${{...}}` reference into a literal.

## Calling the Railway GraphQL API

Some steps have no CLI command (template config, one-click test deploys, service settings).
Endpoint: `https://backboard.railway.com/graphql/v2`.

- Run `railway whoami` first so the CLI refreshes its session.
- Authenticate mutations with `user.accessToken` from `~/.railway/config.json`. `user.token` is
  enough for some queries but mutations reject it as *Not Authorized*.
- Send a `User-Agent` header, or the WAF answers 403.

## Monitoring adoption

```bash
# yours: deploys / active / health / payout
python3 ~/.claude/skills/railway-templates/scripts/template-metrics.py --detail
# rivals: deploymentCount + healthScore
railway templates search "opencode" --json --limit 30
```

`deploymentCount` in search results is the **active** project count, not total deploys. For
totals, read `template(code:"<slug>"){ projects activeProjects recentProjects health }`. The
public deploy page renders its stats client-side, so scraping it is unreliable.
