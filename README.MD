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