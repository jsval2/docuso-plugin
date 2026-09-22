---
name: list
description: List the docu.so documents in this workspace with their status and links. Use when someone asks what they have on docu.so, wants the link to something they published, or is looking for a document by name.
---

You list docu.so documents.

Read `${CLAUDE_PLUGIN_ROOT}/reference/api.md` first: the base URL, the key, the one envelope and what
each error code means. Everything here assumes it.

## Step 1 — fetch

```bash
curl -s "${DOCUSO_API_URL}/api/v1/documents" -H "X-API-Key: ${DOCUSO_API_KEY}"
```

`?page=` and `?limit=` walk the pages; `meta.total` is how many there are in all. A row carries no
`markdown` — fetch the document itself for that.

## Step 2 — show a table

```
| # | Title                      | Status    | Link / shortcode | Updated     |
|---|----------------------------|-----------|------------------|-------------|
| 1 | Website Redesign Proposal  | published | {url}            | 2 hours ago |
| 2 | Q2 OKRs                    | draft     | a3b8n1kp         | 1 day ago   |
```

- A published row shows the `url` the API returned, exactly as it came back. Never assemble a link.
- A draft has no live URL: show its shortcode.
- Dates relative: "2 hours ago", "1 day ago".
- Sort by `updated_at`, newest first. Truncate a title over 40 characters.

## Step 3 — one line underneath

> **3 documents** (2 published, 1 draft)

If `meta.total` is larger than the rows you showed, say that this is the first page.

## Rules

- Keep it short. The table and the summary line, no commentary.
- Nothing yet: "No documents here. Create one with /docuso:new, or a full page with /docuso:page."
- On a failure show `error.code` and `error.message` and say what that code means. A 429 is retried once
  after its `Retry-After`; only a 401 is about the key.
