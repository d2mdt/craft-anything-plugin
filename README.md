# Craft Anything for Claude Code

A personal Claude Code plugin that connects the **Craft Anything** desktop app's inspection engine
([storytold/craft-anything](https://github.com/storytold/craft-anything)) to Claude, in every
project. You sign in to a website yourself, in a real browser window, and Claude can then study
how the logged-in side of the site works.

## What you get

| | |
|---|---|
| **MCP server** `craft-anything` | All ~320 Craft Anything tools (browser, JavaScript, network, native apps, and more), started from the copy of `craft-anything-cli` inside your installed app. |
| `/craft-anything:sign-in <url>` | Opens a **visible browser window on your screen**. You sign in (2FA included) and close the window. The session is saved for that site, and Claude never sees your password. |
| `/craft-anything:review-site <url> [question]` | Maps a site's pages, API calls, auth/session behaviour and front-end code, then writes a report to `site-review/<host>.md`. |
| `/craft-anything:doctor` | Fixes a connection that won't start. |

Claude also picks these skills up by itself when you ask something like *"log in to my dashboard
and figure out how its API works"*.

## Install (on your own computer)

Prerequisite: install the Craft Anything app. It ships `craft-anything-cli`, which this plugin uses.

1. Put this folder somewhere permanent, e.g. `~/claude-plugins/craft-anything-plugin`.
2. In Claude Code (terminal or desktop app), run:

   ```
   /plugin marketplace add ~/claude-plugins/craft-anything-plugin
   /plugin install craft-anything@craft-anything-local
   ```

   Pick **user scope** when asked, so the plugin is available in every project.
3. Restart Claude Code, then run `/mcp` and check that `plugin:craft-anything:craft-anything` is connected.

To keep it on GitHub instead, push this folder to a repository and use
`/plugin marketplace add <you>/<repo>`.

## How a logged-in review works

1. `/craft-anything:sign-in https://app.example.com/login`. A browser window opens and you sign
   in. Then close the window.
2. `/craft-anything:review-site https://app.example.com "how do orders and invoices get loaded?"`
3. Claude opens the site with your saved session (hidden browser), records network calls,
   scripts, menus and flows, and writes the report. It stays **read-only**: no form submissions,
   deletions or purchases unless you ask for that specific action.

Network and script inspection needs Craft Anything's **Deep mode**. Claude will ask before turning
it on and offer to turn it off again afterwards. After switching it on, run `/mcp` and reconnect
the `craft-anything` server so Claude Code picks up the extra tools. To skip that step, start in
Deep mode every time by adding this to `~/.claude/settings.json`:

```json
{ "env": { "CRAFT_ANYTHING_DEEP": "1" } }
```

The saved sign-in lives in Craft Anything's folder on your computer. Ask Claude to "forget the
sign-in for example.com" (`web_site_sign_out`) to remove it.

## Notes

- The sign-in window needs a desktop. Use this plugin in Claude Code on your own machine, not in a
  cloud/web session.
- If the app is installed somewhere unusual, set `CRAFT_ANYTHING_CLI=/full/path/to/craft-anything-cli`.
- **Windows:** the launcher runs through `sh` (from Git for Windows). If that isn't available,
  change `plugins/craft-anything/.mcp.json` to call `C:\Program Files\Craft Anything\craft-anything-cli.exe`
  directly with `["mcp", "--all-tools"]`.
- If you previously ran `craft-anything-cli setup`, you'll have a second copy of the tools.
  `craft-anything-cli uninstall --dry-run` shows what removing that copy would do.
- Only inspect sites you own or are allowed to test.
