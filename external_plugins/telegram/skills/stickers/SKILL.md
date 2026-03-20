---
name: stickers
description: Add Telegram sticker packs with AI-generated descriptions for each sticker. Use when the user wants to add a sticker pack, list known packs, or remove one.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(nix-shell *)
  - Bash(mkdir *)
  - Bash(ls *)
  - mcp__plugin_telegram_telegram__reply
---

# /telegram:stickers — Sticker Pack Management

Downloads sticker packs from Telegram, converts each sticker to a viewable
image, and generates a JSON mapping of sticker file_unique_ids to
human-readable descriptions. This lets you recognize stickers in future
conversations without re-downloading them.

Arguments passed: `$ARGUMENTS`

---

## State

`~/.claude/channels/telegram/sticker-packs.json`:

```json
{
  "<pack_name>": {
    "title": "Human-readable pack title",
    "stickers": {
      "<file_unique_id>": "<description of what the sticker depicts>"
    }
  }
}
```

Missing file = `{}`.

---

## Dispatch on arguments

### No args — list known packs

1. Read `~/.claude/channels/telegram/sticker-packs.json` (handle missing).
2. Show each pack name, title, and sticker count.
3. If empty, tell the user to run `/telegram:stickers add <pack_name>`.

### `add <pack_name>`

This is the main workflow. It downloads every sticker in the pack, converts
animated ones to PNG, shows each to you for description, and saves the mapping.

1. Read existing `sticker-packs.json` (create `{}` if missing).
2. Use the Telegram Bot API to get the sticker set. Run:
   ```
   curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getStickerSet?name=<pack_name>"
   ```
   Read the bot token from `~/.claude/channels/telegram/.env`.
3. Parse the response. Extract `title` and the `stickers` array.
4. For each sticker in the set:
   a. Download the file via `getFile` + the file URL.
   b. If the file is `.webm` or `.tgs`, convert to PNG using:
      `nix-shell -p ffmpeg --run "ffmpeg -i <input> -frames:v 1 -y <output.png>"`
   c. If the file is `.webp`, it can be read directly.
   d. Read the resulting image file with the Read tool.
   e. Write a short description of what the sticker depicts (character,
      emotion, action — 5-15 words). Include the emoji if present.
   f. Record: `stickers[file_unique_id] = description`
   g. Clean up the downloaded/converted files.
5. Save the updated `sticker-packs.json` (pretty-printed, 2-space indent).
6. Report: pack title, number of stickers described.

**Important**: Process stickers in batches of ~5 to avoid overwhelming context.
Read 5 images, describe them, then move on.

### `remove <pack_name>`

1. Read `sticker-packs.json`.
2. Delete the key. Write back.
3. Confirm.

### `describe <pack_name>`

1. Read `sticker-packs.json`.
2. Show all sticker descriptions for the given pack in a table.

---

## Working directory

Use `~/.claude/channels/telegram/inbox/` as a temp directory for downloaded
sticker files. Clean up after processing each batch.

## Environment

The bot token is in `~/.claude/channels/telegram/.env` as
`TELEGRAM_BOT_TOKEN=<value>`. Read it with:
```
grep TELEGRAM_BOT_TOKEN ~/.claude/channels/telegram/.env | cut -d= -f2
```
