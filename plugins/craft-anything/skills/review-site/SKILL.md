---
name: review-site
description: Study how a website works behind its login (pages, menus, API calls, data flow, front-end code, auth/session behaviour) using Craft Anything's browser tools, and write up a backend map. Use when the user wants Claude to "review", "understand", "map", "audit" or "figure out how the back end works" for a site, dashboard, SaaS app or admin panel, especially one that needs a login.
argument-hint: <site URL> [what you want to understand]
---

# Review a website, including its logged-in side

Goal: explain how the site works, especially the parts a visitor can't see: which API endpoints the
pages call, what data comes back, how auth and sessions work, how the front end is built. Back every
claim with Craft Anything evidence, not guesses.

You can't read a server's source code from outside. What you *can* observe is everything the
browser sees: requests and responses, scripts, storage key names, page states. Say this plainly
when you report, and separate **observed** from **inferred**.

## 0. Ground rules

- Only sites the user owns, runs, or is allowed to test. Read-only by default: **don't submit forms,
  delete, buy, send messages or change settings** unless the user asks for that specific action.
- Never handle the user's credentials. Signing in is the user's job, through the `sign-in` skill.
- Use the Craft Anything working memory so long reviews survive context resets:
  `core_investigation_show` first, `core_brief_write` + `core_plan_update` before analysis,
  `core_finding_record` for settled facts, `core_handoff_write` at the end of every turn.

## 1. Mode check (Deep mode)

Clean room (the default) only allows the visitor's view: screenshots, layout, menus. Network
calls, scripts and `web_page_show` need **Deep mode**. In Clean room those tools are not even
listed. If `web_page_show` / `web_walkthrough_record` aren't in your tool list:

1. Explain in one line why Deep mode is needed and ask the user's OK.
2. Call `settings_set` with `key: "deep_mode"`, `value: true`. This is a global, saved Craft Anything
   setting; offer to set it back to `false` at the end.
3. The server does **not** notify Claude Code that its tool list changed. Ask the user to run
   `/mcp`, select `plugin:craft-anything:craft-anything`, and choose **Reconnect** (or restart
   Claude Code). Saved sign-ins survive this. Then continue.

(Users who always want Deep mode can set `CRAFT_ANYTHING_DEEP=1` in their environment or in
`~/.claude/settings.json` under `"env"`. The plugin then starts the server with `--deep`.)

## 2. Sign in

If the site needs a login and no sign-in is saved yet, run the `sign-in` skill's steps first
(the user signs in in a visible window). Then `web_site_open` with `signed_in: true`.

## 3. Map the site

Work through these, recording evidence as you go. Pick what fits the user's question; don't run
everything by reflex.

| Want to know | Tools |
|---|---|
| What the page loads and calls | `web_page_show` with network outlines, console and storage key names on |
| Behaviour over time (polling, websockets, live updates) | `web_page_watch` |
| Front-end framework, bundles, routes, API client code | `web_page_js_analyze`, `web_modules_trace`, `js_app_map`, `js_sources_restore` |
| Menus, dialogs, tabs and the requests they trigger | `web_page_states_capture` |
| A multi-step flow (navigate, open a record, filter) | `web_walkthrough_record` with `capture_network: true` and `capture.events`. Leave `cdp_endpoint` unset so it runs inside the signed-in browser `web_site_open` already opened (call that first). Keep `keep_bodies` off unless the user wants response shapes. Never put credentials in `fill` steps or `storage`. |
| Full visual blueprint of a page | `web_site_capture` |
| A mock of the API to build against | `web_standin_build` from a capture, then `web_standin_start` |

Each tool's own schema is authoritative; read it if a parameter here doesn't match.

For every page or flow, note: URL/route, endpoints called (method, path pattern, query params),
response shape (field names and types, not values), auth mechanism visible from outside (cookie vs
bearer header vs none, CSRF token names, refresh calls), errors and redirects, third-party services
(analytics, payments, auth providers, CDNs).

## 4. Report

Write the result to a Markdown file in the current project (e.g. `site-review/<host>.md`) unless the
user asked for something else, then summarize in chat. Suggested sections:

1. **Overview**: what the app is, stack signals (framework, hosting, API style: REST/GraphQL/RPC).
2. **Auth & session**: how login, session renewal and logout appear from the outside.
3. **Page → endpoint map**: a table of route, what it shows, and the calls it makes.
4. **API surface**: each endpoint with method, purpose, request/response shape, who calls it.
5. **Data model (inferred)**: entities and relations implied by the responses. Label it inferred.
6. **Notable behaviour**: polling, websockets, caching, feature flags, error handling.
7. **Observations / risks**: e.g. data returned that the UI doesn't show, verbose errors, missing
   cache headers. Report only; no exploitation.
8. **Unknowns**: what can't be seen from the browser, and how to learn it (server logs, source access).

Never include cookie values, tokens, API keys, emails or other personal data from responses in the
report. Field names and shapes only.

`craft-anything-cli report` can also turn recorded findings into a Markdown or HTML report if the
user wants Craft Anything's own format.

## 5. Clean up

`web_site_close`. Ask whether to keep the saved sign-in (`web_site_sign_out` forgets it) and whether
to turn Deep mode back off.
