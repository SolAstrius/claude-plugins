---
name: configure
description: Set up the Matrix channel — save the homeserver URL and access token, review access policy. Use when the user pastes a Matrix access token, asks to configure Matrix, or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /matrix:configure — Matrix Channel Setup

Writes the homeserver and access token to `~/.claude/channels/matrix/.env` and
orients the user on access policy. The server reads both files at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Credentials** — check `~/.claude/channels/matrix/.env` for
   `MATRIX_HOMESERVER` and `MATRIX_ACCESS_TOKEN`. Show set/not-set; if set,
   show homeserver and first 10 chars of token masked (`syt_abc...`).

2. **Access** — read `~/.claude/channels/matrix/access.json` (missing file =
   defaults: `dmPolicy: "allowlist"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list user IDs
   - Rooms: count and list room IDs

3. **What next** — end with a concrete next step based on state:
   - No credentials -> *"Set homeserver and token with
     `/matrix:configure <homeserver> <access_token>`"*
   - Credentials set, nobody allowed -> *"Add yourself with
     `/matrix:access allow @you:server`"*
   - Credentials set, someone allowed -> *"Ready. Message the bot on Matrix
     to reach the assistant."*

### `<homeserver> <access_token>` — save credentials

1. Parse `$ARGUMENTS`. First arg is the homeserver URL, second is the access
   token. The homeserver should be a full URL like `https://matrix.org`.
2. `mkdir -p ~/.claude/channels/matrix`
3. Read existing `.env` if present; update/add the `MATRIX_HOMESERVER=` and
   `MATRIX_ACCESS_TOKEN=` lines, preserve other keys. Write back, no quotes.
4. Confirm, then show the no-args status so the user sees where they stand.
5. Remind: "Restart the session or run `/reload-plugins` for changes to take
   effect."

### `clear` — remove credentials

Delete the `MATRIX_HOMESERVER=` and `MATRIX_ACCESS_TOKEN=` lines (or the file
if those are the only lines).

---

## Getting a Matrix access token

If the user asks how to get a token, guide them:

1. **Create a bot account** on their homeserver (or use an existing account).
2. **Get an access token** — the easiest way:
   - Log in to Element (or any Matrix client) with the bot account
   - Go to Settings -> Help & About -> Advanced -> Access Token
   - Copy the token (it starts with `syt_` on newer homeservers)
3. **Or** use the login API:
   ```
   curl -X POST https://matrix.org/_matrix/client/v3/login \
     -H 'Content-Type: application/json' \
     -d '{"type":"m.login.password","user":"@bot:matrix.org","password":"..."}'
   ```
   The response contains `access_token`.

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error.
- The server reads `.env` once at boot. Credential changes need a session
  restart or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/matrix:access` take effect immediately, no restart.
