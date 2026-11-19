---
name: sign-in
description: Sign in to a website through Craft Anything so Claude can study its logged-in pages. Opens a visible browser window on the user's own screen where THEY type their credentials; Claude never sees the password. Use when the user wants Claude to look at a site that needs a login, says "log in to", "sign in to", "use my account on", or before reviewing a dashboard, admin panel or members area.
argument-hint: <site URL, e.g. https://app.example.com/login>
---

# Sign in to a website (user signs in, Claude never sees credentials)

Craft Anything's `web_site_sign_in` tool opens a **real, visible browser window on the user's
computer** with a profile kept just for that site. The user signs in there by hand and then closes
the window. Craft Anything saves that browser profile, and later `web_site_open` calls for the same
host and port start from a copy of it. It never reads the password, cookie values or storage values.

## Rules

- **Never ask the user for their password, 2FA code, cookies or session tokens, and never type them
  yourself** (not in `web_walkthrough_record` `fill` steps, not in `storage`, not anywhere). If the user
  pastes one into the chat, tell them it isn't needed and suggest they change it if it's sensitive.
- Only do this on sites the user owns, administers, or is otherwise allowed to inspect. If that's
  unclear for a third-party site, ask once.
- The saved sign-in lives in Craft Anything's folder on this computer. Say so once, and say that
  `web_site_sign_out` forgets it.

## Steps

1. Get the URL from `$ARGUMENTS` or ask for it. Prefer the site's login page.
2. Tell the user, before calling the tool, in one or two lines:
   "A browser window will open on your screen. Sign in there as you normally would (including any
   2FA), go to a page that shows you're logged in, then **close the window**. I'll wait up to 10 minutes."
3. Call `web_site_sign_in` with `url` (and `timeout_secs` up to 1800 if they need longer, e.g. an
   email code). The call blocks until the window closes.
4. Confirm it worked: `web_site_open` with the dashboard / account URL and `signed_in: true`, then
   report the final address and title. If it bounced to the login page, the sign-in didn't stick:
   offer to try again (some sites need "remember me" ticked).
5. Leave the page open for the next step, or `web_site_close` if nothing else is planned.

## If the tool is missing or fails

- No `web_site_sign_in` tool listed: the plugin's MCP server isn't running. Use the `doctor` skill.
- "Needs a desktop where a window can open": Claude Code is running somewhere without a screen
  (a cloud session, SSH, a container). Sign-in must happen in Claude Code on the user's own computer.
- Refused as restricted: the server was started with `--restricted`; the plugin doesn't do that,
  so another Craft Anything MCP entry is probably being used instead (see `doctor`).
