---
name: inbox
description: Show the DevMail test inbox address, mailbox usage and the most recent emails, optionally filtered by sender, recipient alias or subject. Use when the user asks what is in their DevMail inbox or what their DevMail address is.
argument-hint: "[from:<sender>] [to:<alias>] [subject:<text>]"
allowed-tools: mcp__plugin_devmail_devmail
---

# DevMail inbox overview

Arguments: `$ARGUMENTS`

1. Call the DevMail `get_inbox_info` tool. Report the inbox address and mailbox usage (used / limit, percent) in one line. If usage is 90% or more, warn that new emails may be rejected.
2. Call `list_messages` with `limit: 10`. Map the arguments to filters:
   - `from:<value>` → `from`
   - `to:<value>` → `to` (plus-address aliases such as `inbox+signup-123@...` are stored as the recipient)
   - `subject:<value>` → `subject`
   - Any other free text → `subject`
   All filters are case-insensitive partial matches.
3. Show the result as a compact table: received time (local, relative if recent), from, to, subject, attachment count, message id. Newest first. Mention `total` when it is larger than the rows shown.
4. If nothing matches, say so and suggest loosening the filter. Do not fetch full message bodies unless the user asks; point them to `/devmail:wait-for-email` or `/devmail:attachments` instead.

If a tool returns an authentication error, tell the user to check the API key in the plugin settings (`/plugin` → devmail → configure) and that keys come from https://devmail.space/settings.
