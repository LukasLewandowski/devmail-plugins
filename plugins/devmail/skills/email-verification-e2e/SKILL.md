---
name: email-verification-e2e
description: Write end-to-end tests (Playwright, Cypress or plain HTTP) that read emails from a DevMail inbox through the DevMail REST API, for sign-up verification, password reset, magic link, OTP or notification flows. Use when the user writes or fixes an e2e test that needs to receive an email.
---

# E2E email tests with DevMail

Tests in CI talk to the DevMail **REST API**, not to the MCP tools of this plugin. Use the MCP tools only to explore the inbox while writing the test (for example to see the real subject line and link format).

## API basics

- Base URL: `https://api.devmail.space/api/v1`
- Auth header: `Authorization: Bearer <key>`. Read the key from an environment variable (for example `DEVMAIL_API_KEY`), never hard-code it. Keys are created at https://devmail.space/settings.
- `GET /messages?to=&from=&subject=&since=&before=&has=attachment&limit=&start=` returns `{ total, count, start, messages: [{ id, from, to, subject, snippet, hasAttachments, attachmentCount, size, receivedAt }] }`, newest first. Text filters are case-insensitive partial matches. `limit` is 1–20.
- `GET /message/:id` returns the full message with `body.text`, `body.html`, `attachments: [{ partId, filename, contentType, size }]`, `size` and `receivedAt`.
- `GET /message/:id/headers`, `GET /message/:id/attachment/:partId` (binary), `GET /messages/count`, `GET /info` (inbox address and usage).
- Full reference: https://devmail.space/docs and Swagger UI at https://api.devmail.space/docs.

## Rules for reliable tests

1. **Unique recipient per test run.** Build a plus-address alias from the inbox address: `inbox+<test>-<uuid>@domain`. Sign up with it and filter with `to`. Parallel CI jobs share one inbox, so never pick "the newest message" without a `to` filter. Build query strings with `URLSearchParams` or the client's `params` option so `+` is encoded.
2. **Poll, do not sleep once.** Delivery is asynchronous. Retry the list request every 1–2 seconds with a total timeout of 30–60 seconds (Playwright: `expect.poll` or `expect(...).toPass()`).
3. **Extract with a specific pattern.** Match the link or code by the surrounding text of the real email (inspect one with the `/devmail:inbox` and `/devmail:wait-for-email` skills first), and prefer `body.text` over `body.html`.
4. **Keep the key in CI secrets** and the inbox address in config.

## Playwright example

```ts
import { test, expect, type APIRequestContext } from '@playwright/test';
import { randomUUID } from 'node:crypto';

const API = 'https://api.devmail.space/api/v1';
const headers = { Authorization: `Bearer ${process.env.DEVMAIL_API_KEY}` };

async function waitForEmail(request: APIRequestContext, params: Record<string, string>) {
  let id: number | undefined;
  await expect
    .poll(
      async () => {
        const res = await request.get(`${API}/messages`, { headers, params: { ...params, limit: '1' } });
        expect(res.ok()).toBeTruthy();
        id = (await res.json()).messages[0]?.id;
        return id;
      },
      { timeout: 60_000, intervals: [1_000, 2_000] }
    )
    .toBeDefined();

  const res = await request.get(`${API}/message/${id}`, { headers });
  expect(res.ok()).toBeTruthy();
  return res.json();
}

test('new user verifies their email', async ({ page, request }) => {
  const [local, domain] = process.env.DEVMAIL_INBOX!.split('@');
  const email = `${local}+signup-${randomUUID()}@${domain}`;

  await page.goto('/register');
  await page.getByLabel('Email').fill(email);
  await page.getByRole('button', { name: 'Sign up' }).click();

  const message = await waitForEmail(request, { to: email, subject: 'Verify' });
  const text: string = message.body.text || message.body.html || '';
  const link = text.match(/https?:\/\/\S+verify\S*/)?.[0];
  expect(link).toBeTruthy();

  await page.goto(link!);
  await expect(page.getByText('Email verified')).toBeVisible();
});
```

Adapt the selectors, subject and link pattern to the application under test. For other frameworks keep the same structure: unique alias, poll `GET /messages` with `to`, then `GET /message/:id`.
