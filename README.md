# Roan's Hermes Dashboard

A dashboard-only fork of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent). The dashboard keeps the upstream code as its base, so the dashboard itself stays a clean copy that can be updated from `upstream` at any time. Everything that is not the dashboard or the runtime it needs is removed from this tree.

This repo is the install source. The install process is the same process Hermes uses when you install it from GitHub, only pointed at this fork.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/roanh47/Roans-Hermes-Dashboard/main/scripts/install.sh | bash
```

The installer clones `https://github.com/roanh47/Roans-Hermes-Dashboard.git` (branch `main`), builds the Python environment, and builds the web frontend into `hermes_cli/web_dist/`. Useful flags:

```bash
install.sh --dir /opt/roans-hermes     # install location, default $HERMES_HOME/hermes-agent
install.sh --branch main               # branch to install
install.sh --non-interactive           # skip the setup and gateway prompts
install.sh --skip-browser              # do not install the browser tools
```

`HERMES_HOME` defaults to `~/.hermes`, and that is where config, sessions and skills live.

## Run

```bash
hermes dashboard
```

The dashboard listens on `127.0.0.1:9120` by default in this fork, and it opens the browser on `/chat`. For a host that is reachable from outside the machine:

```bash
hermes dashboard --host 0.0.0.0 --port 9120
```

On this machine the reverse proxy in front of `roans-hermes.roanheemstra.nl` forwards to `127.0.0.1:9120`.

## Port 9120

Upstream defaults to 9119, and the live Hermes install on the same host already uses it. The default is 9120 in these places:

```text
hermes_cli/subcommands/dashboard.py   --port default
hermes_cli/web_server.py              start_server() default
hermes_cli/main_dashboard.py          status and stop path
docker/s6-rc.d/dashboard/run          HERMES_DASHBOARD_PORT fallback
docker-compose.yml                    ssh tunnel example
docker-compose.windows.yml            published port
web/vite.config.ts                    dev proxy to the backend
```

Because it is a default, the port is still a flag: `hermes dashboard --port 9131` works. `HERMES_DASHBOARD_PORT` overrides it for the s6 service.

## What is in this tree

Kept, because the dashboard imports or reads it at runtime:

```text
hermes_cli/      FastAPI app, web_routers/, CLI, dashboard process handling
agent/           agent runtime behind every dashboard call
gateway/         API and platform serving
tools/           tool implementations the dashboard panels call
providers/       model providers
plugins/         plugin host, including the dashboard plugin API
cron/            scheduled jobs shown in the dashboard
tui_gateway/     websocket bridge for the embedded terminal
ui-tui/          the real TUI the dashboard embeds on /chat
acp_adapter/     editor and client adapters
web/             the dashboard frontend (Vite, React), builds to hermes_cli/web_dist/
apps/shared/     @hermes/shared, the JSON-RPC and websocket client web/ imports
skills/          bundled skills shown on the skills page
optional-skills/ optional skill catalog the skills page scans
optional-mcps/   optional MCP catalog
assets/ locales/ native/ docker/ scripts/
```

Removed, because the dashboard does not use it:

```text
apps/desktop/    Electron desktop app, frontend generator and installer material
tests/ tests-js/ test suites for the agent and the frontend
website/         the docs site
contributors/ evals/ nix/      contributor tooling, evaluation suites, Nix packaging
.github/         upstream CI for the full agent
docs/            agent design notes
README.<lang>.md translated upstream readmes
```

Two consequences of the removal: `install.sh --include-desktop` no longer exists, and `apps/shared` is the only workspace left in `apps/`.

## Update from upstream

This stays a fork, so the dashboard code can be pulled from the original repo instead of being maintained by hand:

```bash
git remote add upstream https://github.com/NousResearch/hermes-agent.git
git fetch upstream
git merge upstream/main
```

Conflicts are expected in the files that carry this fork's changes: the port defaults listed above, `web/index.html`, the brand strings in `web/src/i18n/`, `scripts/install.sh` and the removed directories.

## Docker

`docker-compose.yml` runs the container with the dashboard on 9120 for a host without a local Python install:

```bash
docker compose up -d
```

The compose file publishes the dashboard on `127.0.0.1:9120`. For access from another machine, tunnel it instead of publishing it on all interfaces:

```bash
ssh -L 9120:localhost:9120 user@host
```

## Credits

The dashboard, the agent runtime and the install script come from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) (Apache-2.0). This fork only changes the packaging, the default port and the branding.
