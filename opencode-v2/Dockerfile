FROM node:24-slim

# Tools a coding-agent box needs: git (clone/commit), gh (PRs/issues),
# openssh-client (git over SSH), curl/ca-certs/gnupg (installers & TLS).
RUN apt-get update && apt-get install -y --no-install-recommends \
        git openssh-client ca-certificates curl gnupg procps \
    && rm -rf /var/lib/apt/lists/*

# GitHub CLI — so the agent can `gh pr create`, `gh issue`, etc.
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        -o /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && chmod 644 /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
        > /etc/apt/sources.list.d/github-cli.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/*

# OpenCode 2 — published as `@opencode/cli` on npm. (`opencode-ai` is the 1.x
# line and its `latest` tag stays on 1.x, so the two never cross.)
#
# PINNED ON PURPOSE, same as the 1.x template: 2.0 shipped ~30 releases in its
# first five weeks, and a floating version hands every bad one straight to
# whoever deploys that day. Bump deliberately; CI asserts the image runs it.
ARG OPENCODE_VERSION=2.1.4
RUN npm install -g "@opencode/cli@${OPENCODE_VERSION}" \
    && opencode --version

# Upgrades come from a new image, never from inside a running box.
ENV OPENCODE_DISABLE_AUTOUPDATE=1

WORKDIR /workspace
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# The web UI + API listen here; Railway maps the public domain to $PORT.
ENV PORT=8080
EXPOSE 8080

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
# Main process = OpenCode 2's server, which serves the web UI and the API on
# the public Railway domain (password protected). `--service` registers it as
# the box's background service, so `railway ssh` → `opencode` attaches the TUI
# to this same server: one set of sessions, visible from browser and terminal.
CMD ["bash", "-lc", "exec opencode serve --service --hostname 0.0.0.0 --port ${PORT:-8080}"]
