# DevMail plugins for Claude Code

Official [Claude Code](https://code.claude.com) plugin marketplace for [DevMail](https://devmail.space), the test inbox for developers and QA.

The `devmail` plugin connects Claude Code to your DevMail inbox. Claude can wait for verification emails, pull out links and one-time codes, check attachments and help you write e2e tests that read email.

## Install

```
/plugin marketplace add LukasLewandowski/devmail-plugins
/plugin install devmail@devmail-plugins
```

Claude Code asks for your DevMail API key (`mlb_...`) when the plugin is enabled. Create one at https://devmail.space/settings. The key is stored in your system keychain, not in settings files.

Check the connection with `/mcp`. The `plugin:devmail:devmail` server should be connected with 6 tools.

## Skills

| Skill | What it does |
| --- | --- |
| `/devmail:inbox [filter]` | Inbox address, mailbox usage and the latest emails |
| `/devmail:wait-for-email <subject\|from:\|to:> [timeout]` | Waits for an email and extracts the verification link, magic link or OTP code |
| `/devmail:attachments [filter]` | Lists attachments and downloads one to check its type, size or contents |
| `email-verification-e2e` | Used automatically when you write e2e tests that need to receive an email (Playwright and REST API patterns) |

`wait-for-email` and `email-verification-e2e` can also start on their own when Claude is testing a sign-up flow or writing a test.

## MCP tools

All tools are read-only. Nothing in your inbox is changed or deleted.

| Tool | Purpose |
| --- | --- |
| `list_messages` | List and search emails (`from`, `to`, `subject`, `since`, `before`, attachments), with pagination and incremental polling |
| `count_messages` | Count emails and attachments among the newest matches |
| `get_message` | Full message: text and HTML body, attachment metadata |
| `get_message_headers` | Raw headers of a message |
| `get_message_attachment` | Download one attachment (up to 5 MB) |
| `get_inbox_info` | Inbox address and mailbox usage |

## Configuration

| Option | Default | Notes |
| --- | --- | --- |
| `api_key` | – | Required. Your `mlb_...` key. |
| `mcp_url` | `https://mcp.devmail.space/mcp` | Change only to test a local MCP server, for example `http://localhost:8787/mcp`. |

To change them later, open `/plugin`, select `devmail` and configure it.

## Privacy

The MCP server at `mcp.devmail.space` forwards each request to the DevMail public API with your key and does not store the key or email content. See the [privacy policy](https://devmail.space/privacy) and [terms](https://devmail.space/terms).

## Without the plugin

You can also add the MCP server directly:

```
claude mcp add --transport http devmail https://mcp.devmail.space/mcp --header "Authorization: Bearer mlb_xxx"
```

The plugin adds the skills on top and keeps the key out of your shell history.

## License

[MIT](LICENSE)
