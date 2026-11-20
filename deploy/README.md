# Run our TrendRadar fork

Use `CSMENG211/TrendRadar` as `origin` and `sansan0/TrendRadar` as `upstream`. Commit and push patches to our fork so other machines can pull them. Run the checked-out source; official Docker images do not include our changes.

The fork uses local SQLite storage and `America/Los_Angeles` time. Built-in AI analysis, translation, notifications, and HTML reports are disabled. The connected MCP client handles analysis. No separate model API key is needed for collection or MCP queries.

## Linux

Install `uv`, then run:

```bash
git clone git@github.com:CSMENG211/TrendRadar.git "$HOME/projects/repos/TrendRadar"
cd "$HOME/projects/repos/TrendRadar"
git remote add upstream https://github.com/sansan0/TrendRadar.git
uv sync --frozen
mkdir -p "$HOME/.config/systemd/user"
cp deploy/systemd/trendradar-* "$HOME/.config/systemd/user/"
systemctl --user daemon-reload
systemctl --user enable --now trendradar-mcp.service trendradar-crawl.timer
systemctl --user start trendradar-crawl.service
```

The service files expect this checkout path. Change their paths if installing elsewhere. User services start when the user manager starts; enable lingering with `sudo loginctl enable-linger "$USER"` if this machine must collect before login.

Connect the MCP client to `http://127.0.0.1:3333/mcp` using Streamable HTTP. The collector runs every 30 minutes. Data stays in `output/`; new data, credentials, logs, and virtual environments are ignored by Git. The sample databases inherited from upstream remain tracked.

```bash
systemctl --user status trendradar-mcp.service trendradar-crawl.timer
journalctl --user -u trendradar-crawl.service -n 50 --no-pager
```

## macOS

```bash
git clone git@github.com:CSMENG211/TrendRadar.git "$HOME/projects/TrendRadar"
cd "$HOME/projects/TrendRadar"
git remote add upstream https://github.com/sansan0/TrendRadar.git
uv sync --frozen
TZ=America/Los_Angeles .venv/bin/python -m trendradar
TZ=America/Los_Angeles .venv/bin/python -m mcp_server.server --transport http --host 127.0.0.1 --port 3333 --project-root "$PWD"
```

Use the assistant repo's `docs/modules/macos_migration.md` for LaunchAgents that keep MCP running and collect every 30 minutes. Keep both the process `TZ` and `config/config.yaml` timezone the same so queries for today use the correct database.

## Update our fork

```bash
git pull --ff-only origin master
uv sync --frozen
```

Restart MCP after updating source or configuration. On Linux, run `systemctl --user restart trendradar-mcp.service`. Review upstream updates before merging and pushing them into our fork.
