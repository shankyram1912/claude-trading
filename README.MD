# Setup — Trader Claude

## Installation

Install Claude Code using the official install script:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

> **Note:** On Lubuntu, always pipe to `bash` explicitly — not `sh`. Lubuntu's default `sh` is `dash`, which doesn't support bash syntax and will throw a `Syntax error: "(" unexpected` error.

After install, reload your shell:

```bash
source ~/.bashrc
```

Verify the install:

```bash
claude --version
```

---

## Starting Claude Code in a Project

Clone your git repo and launch Claude Code from within it:

```bash
git clone <your-repo-url> /data/tools/trader-claude
cd /data/tools/trader-claude
claude
```

Claude Code will start in the context of that project directory. Any project-level settings, MCP configs, and custom commands will be scoped to this folder.

---

## Global Settings (First Run)

On first launch, Claude Code presents an interactive menu to configure two global settings. Use the arrow keys to select and Enter to confirm.

**1. Text style** — select the theme that renders best in your terminal:

```
❯ Choose the text style that looks best with your terminal
  > Dark
    Light
    Auto
```

**2. Login method** — authenticate your Anthropic account:

```
❯ Select login method
  > Login with Claude.ai (free or Pro account)
    Use an Anthropic API key
```

These are global settings, stored in `~/.claude/` and applied across all projects. You won't be prompted again on subsequent launches. Auth tokens are saved to `~/.claude/.credentials.json`.

---

## Install Discord Plugin

Plugins are user-scoped — installed once per machine to `~/.claude/`, not per project.

```bash
# Register the official marketplace (skips if already registered)
claude plugin marketplace add anthropics/claude-plugins-official

# Install Discord plugin (skips if already installed)
claude plugin install discord@claude-plugins-official

# Verify
claude plugin list
```

Do this before starting the daemon, otherwise the `--channels` flag will fail.

---

## Configure Discord Plugin (Project Level)

The Discord bot token, access pair code, and access policy allowlist are pre-configured in the project's local git files:

```
.discord/access.json
```

This file is project-scoped — committed to the repo so the configuration travels with the project, not the machine.

After installing the plugin, activate its configuration commands by running inside Claude Code:

```bash
/reload-plugins
```

This makes the `/discord:access` and related commands available in the current session.

### Generating a New Bot Token

If you need a fresh token:

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **New Application** and give it a name (e.g. `trader-claude`)
3. Navigate to the **Bot** section — set a username
4. Click **Reset Token** and copy the token immediately — it won't be shown again

> ⚠️ Treat the token like a password. Do not commit it to git. Store it securely and set it only at the project level as described below.

### Activate the Bot

> ⚠️ **Gotcha:** `/discord:configure <token>` writes to global `~/.claude/` settings, even when a project-level `.env` is present. To keep the token project-scoped, skip the command and set it directly in the project file instead:

```
.discord/.env
```

Update the token value there manually. This ensures the credential stays tied to the project and doesn't bleed into other Claude Code sessions on the same machine.

---

## TradingView MCP Server

[tradesdontlie/tradingview-mcp](https://github.com/tradesdontlie/tradingview-mcp) is an MCP server that connects Claude Code to your locally running TradingView Desktop app via Chrome DevTools Protocol (CDP). It gives Claude eyes and hands on your charts — Pine Script development, chart navigation, indicator reading, screenshots, and more.

> ⚠️ Not affiliated with TradingView Inc. Requires a valid TradingView subscription. All data stays local — no connection to TradingView's servers.

### Step 1 — Install TradingView Desktop (Local, No Root)

TradingView Desktop is installed locally by extracting the `.deb` package — no system-wide install required.

```bash
# Decompress
gunzip tradingview_amd64.deb.gz

# Extract into local directory
mkdir -p /data/tools/tradingview
dpkg-deb -x tradingview_amd64.deb /data/tools/tradingview/TradingView
```

Add to PATH in `~/.bashrc`:

```bash
# Trading View
export PATH="$PATH:/data/tools/tradingview/opt/TradingView"
```

```bash
echo '# Trading View' >> ~/.bashrc
echo 'export PATH="$PATH:/data/tools/tradingview/opt/TradingView"' >> ~/.bashrc
source ~/.bashrc
```

### Step 2 — Handle Chrome Sandbox Error

On first launch you may see:

```
FATAL: chrome-sandbox is not configured correctly...
You need to make sure that chrome-sandbox is owned by root and has mode 4755.
```

**Fix the sandbox (recommended, requires sudo):**

```bash
sudo chown root:root /data/tools/tradingview/opt/TradingView/chrome-sandbox
sudo chmod 4755 /data/tools/tradingview/opt/TradingView/chrome-sandbox
```

### Step 3 — Clone and Install the MCP Server

```bash
git clone https://github.com/tradesdontlie/tradingview-mcp.git /data/tools/tradingview/git/tradingview-mcp
cd /data/tools/tradingview/git/tradingview-mcp
npm install

# To enable claude to take screenshots of Trading View screen
mkdir /data/tools/tradingview/git/tradingview-mcp/screenshots
```

### Step 4 — Launch TradingView with Debug Port

The MCP server connects to TradingView via CDP on port 9222. TradingView must be running with the debug flag:

```bash
./scripts/launch_tv_debug_linux.sh
```

Or manually:

```bash
tradingview --remote-debugging-port=9222
```

Or use the MCP tool (auto-detects your install):

```bash
"Use tv_launch to start TradingView in debug mode"
```

### Step 5 — Add to Claude Code MCP Config

Add to your project-level `.mcp.json`:

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["/data/tools/tradingview/git/tradingview-mcp/src/server.js"]
    }
  }
}
```

### Step 6 — Verify Connection

Inside Claude Code ask:

```
Use tv_health_check to verify TradingView is connected
```