FROM node:20-slim

# Tools a coding-agent box needs: git (clone/commit), gh (PRs/issues),
# openssh-client (git over SSH), curl/ca-certs/gnupg (installers & TLS).
RUN apt-get update && apt-get install -y --no-install-recommends \
        git openssh-client ca-certificates curl gnupg \
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

# OpenCode — the open-source AI coding agent (https://opencode.ai).
#
# PINNED ON PURPOSE. This is a GitHub-sourced template, so every deploy rebuilds
# this image from scratch — `opencode-ai@latest` meant each deployer got whatever
# npm published that hour (~300 publishes/30 days, ~20 stable releases/month).
# One bad upstream release then crash-loops `opencode web` for everyone who
# deploys that day, and the template's deploy-success score takes the hit for a
# bug we didn't write.
#
# Bump this deliberately: change the ARG, redeploy, verify the web UI and TUI,
# then commit. `opencode upgrade` still works at runtime for anyone who wants
# newer inside their own box.
ARG OPENCODE_VERSION=1.18.29
RUN npm install -g "opencode-ai@${OPENCODE_VERSION}" \
    && opencode --version

WORKDIR /workspace
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# `opencode web` tries to launch a browser via xdg-open on start; on a headless
# container that throws a noisy (harmless) error. Shim it to a no-op.
RUN printf '#!/bin/sh\nexit 0\n' > /usr/local/bin/xdg-open \
    && chmod +x /usr/local/bin/xdg-open

# The web UI listens here; Railway maps the public domain to $PORT.
ENV PORT=8080
EXPOSE 8080

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
# Main process = the OpenCode web UI on the public Railway domain (basic-auth
# protected). The container stays alive via this server, and you can still
# `railway ssh` in and run `opencode` for the TUI in the same box.
CMD ["bash", "-lc", "exec opencode web --hostname 0.0.0.0 --port ${PORT:-8080}"]
