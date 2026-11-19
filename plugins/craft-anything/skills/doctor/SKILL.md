---
name: doctor
description: Diagnose and fix the Craft Anything plugin connection, i.e. when the craft-anything MCP tools are missing, fail to start, or a sign-in window doesn't open. Use when the user says the Craft Anything tools aren't showing up, the server won't start, or asks how to set the plugin up.
---

# Craft Anything plugin doctor

The plugin starts `craft-anything-cli mcp --all-tools` through `bin/craft-anything-mcp` in this
plugin's folder. That launcher looks for the CLI in this order: `$CRAFT_ANYTHING_CLI`, `PATH`,
`/Applications/Craft Anything.app/Contents/MacOS/`, `~/Applications/...`, `/usr/bin`,
`/usr/local/bin`, `~/.cargo/bin`, `C:\Program Files\Craft Anything\`, then the Flatpak.

Check in this order, and show the user each result:

1. **Is the CLI there?** Run `sh "${CLAUDE_PLUGIN_ROOT}/bin/craft-anything-mcp" --version`.
   - Not found: the user needs the Craft Anything app installed (it ships the CLI inside it), or
     `cargo install --path apps/craft-anything-cli` from a clone. If it's installed somewhere
     unusual, have them set `CRAFT_ANYTHING_CLI` to the full path in their shell profile, or in
     `~/.claude/settings.json` under `"env"`, then restart Claude Code.
   - macOS "cannot be opened" / quarantine: they should open the app once from Finder first.
2. **Does it run?** `"<cli path>" doctor`. Relay what it says is missing (Chromium is needed for the
   browser tools; it reports where it looked).
3. **Is the server connected?** Ask the user to run `/mcp` in Claude Code and look for
   `plugin:craft-anything:craft-anything`. If it shows failed, run the launcher by hand with
   `mcp --all-tools` and read the error on stderr.
4. **Duplicate entries.** If the user earlier ran `craft-anything-cli setup` (or the app's
   "connect to Claude Code"), there may be a second `craft-anything` server in `~/.claude.json` plus
   a `~/.claude/skills/craft-anything` skill. Both work, but the tools appear twice. To use only the
   plugin, they can run `craft-anything-cli uninstall --dry-run` to see what it would remove, then
   run it for real. Never run `setup --yes` over an entry ending in `--all-tools`.
5. **Windows.** The plugin starts the launcher with `sh`, which needs Git for Windows' `sh` on
   PATH. If it isn't, edit this plugin's `.mcp.json` to run the program directly:
   `"command": "C:\\Program Files\\Craft Anything\\craft-anything-cli.exe", "args": ["mcp", "--all-tools"]`.
6. **Only ~110 tools listed / web_page_show missing.** That's Clean room mode, not a fault. See
   step 1 of the `review-site` skill (Deep mode, then reconnect via `/mcp`), or set
   `CRAFT_ANYTHING_DEEP=1`.
7. **No window opens on sign-in.** `web_site_sign_in` needs a desktop. It works in Claude Code on
   the user's own computer (terminal, desktop app or IDE), not in a cloud/web session or over SSH.
