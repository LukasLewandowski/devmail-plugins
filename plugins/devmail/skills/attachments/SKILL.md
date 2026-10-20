---
name: attachments
description: Find emails with attachments in the DevMail test inbox, list their files and download one to check its type, size or contents. Use when the user wants to verify that an invoice, report, export or other attached file was emailed correctly.
argument-hint: "[from:<sender>] [to:<alias>] [subject:<text>] [file name]"
allowed-tools: mcp__plugin_devmail_devmail
---

# DevMail attachments

Arguments: `$ARGUMENTS`

1. Call `list_messages` with `hasAttachment: true`, `limit: 10` and any `from:`, `to:` or `subject:` filters from the arguments. Free text that looks like a file name (has an extension) is matched later against attachment names; other free text is the subject.
2. For the matching messages (newest first, at most 5 unless the user asks for more), call `get_message` and read the `attachments` list: `partId`, `filename`, `contentType`, `size`.
3. Show a table: message subject, received time, file name, content type, human-readable size, message id and partId.
4. Download an attachment only when the user asked to check its contents or a single file clearly matches the request. Call `get_message_attachment` with the message `id` and the `partId`.
   - The tool rejects files over 5 MB. For those, report the metadata only.
   - The result is the file as an embedded resource. For text formats (CSV, JSON, TXT, ICS, XML, HTML) summarise the contents and check what the user asked about, for example column names, row counts or totals. For PDFs and images, describe what you can see if the content is available to you; otherwise say that only the metadata could be verified.
5. Point out anything suspicious for a QA check: an empty file (size 0), a content type that does not match the extension, or a file name that looks unrendered (for example containing `{{` or `undefined`).

Attachments are read-only; nothing in the inbox is changed.
