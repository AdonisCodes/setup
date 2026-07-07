# setup

Personal Cursor skills, MCP config, and automation helpers for Simon.

Synced to GitHub so skills and setup travel across machines. **Secrets stay local.**

## What's in here

| Path | Purpose |
| --- | --- |
| `cursor/skills/` | Agent skills (workflows the agent follows) |
| `cursor/mcp.json.example` | MCP server config template |
| `bin/` | Helper scripts (`chrome-cdp`, etc.) |

## Install on a new machine

```bash
git clone https://github.com/AdonisCodes/setup.git ~/dev/personal/setup

# Skills → Cursor
ln -sf ~/dev/personal/setup/cursor/skills/* ~/.cursor/skills/

# MCP config
cp ~/dev/personal/setup/cursor/mcp.json.example ~/.cursor/mcp.json

# Scripts
ln -sf ~/dev/personal/setup/bin/chrome-cdp ~/.local/bin/chrome-cdp
```

Then reload MCP in Cursor Settings and run `chrome-cdp` before browser-driven workflows.

## Never synced (secrets)

- Chrome profile copies (cookies, sessions)
- Any API keys or tokens
- `.env` files

## Skills

| Skill | Description |
| --- | --- |
| [fitness-overlay-youtube](cursor/skills/fitness-overlay-youtube/SKILL.md) | Strava FIT → fitoverlay → YouTube upload via real Chrome |
