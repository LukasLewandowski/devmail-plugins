---
name: wait-for-email
description: Wait for an email to arrive in the DevMail test inbox and extract its verification link, magic link or one-time code. Use when testing sign-up, password reset, magic link or OTP flows that send an email to a DevMail address.
argument-hint: "<subject|from:<sender>|to:<alias>> [timeout seconds, default 60]"
allowed-tools: mcp__plugin_devmail_devmail, Bash(sleep:*)
---

# Wait for a DevMail email

Arguments: `$ARGUMENTS`

## 1. Work out the filter

- `from:<value>`, `to:<value>`, `subject:<value>` map to the `list_messages` filters of the same name. Free text is the subject.
- A trailing number is the timeout in seconds (default 60, max 300).
- If the triggering action (a sign-up you just made, a form you submitted) used a plus-address alias such as `inbox+signup-123@...`, filter by `to` with that alias. It is the most reliable filter when several emails look alike.

## 2. Poll

1. Note the current time minus 2 minutes as `since` (ISO 8601) to tolerate clock skew, unless the user wants older emails too.
2. Call `list_messages` with the filters, `since` and `limit: 5`.
3. If no message matches, wait about 5 seconds (`sleep 5` in Bash) and try again until the timeout. Tell the user briefly that you are still waiting every few attempts, not on every check.
4. On timeout, say what you filtered on and suggest causes: wrong recipient address, the app did not send the email, or the filter is too narrow. Offer `/devmail:inbox` to see what did arrive.

Do not combine filters with `sinceId`. The tool rejects that combination; `sinceId` is only for unfiltered incremental polling.

## 3. Extract

Take the newest matching message and call `get_message` with its id. Look at `body.text` first, then `body.html`. Extract:

- **Verification, confirmation, reset and magic links:** URLs whose path or anchor text suggests verify, confirm, activate, reset, sign in, login or token. Decode HTML entities such as `&amp;` in URLs taken from HTML.
- **One-time codes:** 4–8 digit (or short alphanumeric) codes next to words like code, OTP, PIN or verification.
- Ignore unsubscribe, tracking pixel, logo and footer links.

Reply with the subject, sender, received time and the extracted link or code in a code block so it can be copied. If several candidates exist, list them and say which one you think is the main action. If you are continuing a test flow (for example in a browser tool), use the link or code directly.
