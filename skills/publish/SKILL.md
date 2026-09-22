---
name: publish
description: Publish a docu.so document so it has a live link, or unpublish one so its link stops working. Use when someone wants to share a draft, get the link for something, or take a published page down.
---

You publish and unpublish docu.so documents. Publishing gives a document a live link; unpublishing
takes that link away.

Read `${CLAUDE_PLUGIN_ROOT}/reference/api.md` first: the base URL, the key, the one envelope and what
each error code means. Everything here assumes it.

## With an argument

`/docuso:publish x7k9m2fq` or `/docuso:publish Q2 OKRs`.

1. Eight lowercase alphanumeric characters is a shortcode; anything else is a title.
2. Find it in `GET /api/v1/documents` and read its `status`.
3. A draft: publish it.
4. Already published: *"This one is already live at {url}. Unpublish it instead?"*

## With no argument

```bash
curl -s "${DOCUSO_API_URL}/api/v1/documents" -H "X-API-Key: ${DOCUSO_API_KEY}"
```

Show drafts first — they are what someone came here to publish:

```
Drafts:
 1. Q2 OKRs — a3b8n1kp — updated 1d ago
 2. Meeting notes, March — p7q2w8xk — updated 5d ago

Published:
 3. Website Redesign Proposal — x7k9m2fq — updated 2h ago
```

> Which one? (a number, or "unpublish 3")

## Publishing

```bash
curl -s -X POST "${DOCUSO_API_URL}/api/v1/documents/${DOC_ID}/publish" \
  -H "X-API-Key: ${DOCUSO_API_KEY}"
```

> Published: {url}

`.data.url` from that answer, verbatim. Never assemble a link.

## Unpublishing

Confirm first — it takes a live link away:

> Unpublish "{title}"? {url} will stop working.

```bash
curl -s -X POST "${DOCUSO_API_URL}/api/v1/documents/${DOC_ID}/unpublish" \
  -H "X-API-Key: ${DOCUSO_API_KEY}"
```

> Unpublished. It is a draft again.

## Rules

- Publishing is the happy path and needs no confirmation. Unpublishing always asks first.
- A published page may take up to a minute to disappear from a shared cache after it is unpublished.
  Say so rather than letting someone think the unpublish failed.
- Nothing to publish and no argument: "Everything is already published. Pass a shortcode to unpublish
  one: /docuso:publish {shortcode}"
- On a failure show `error.code` and `error.message` and say what that code means. A 429 is retried once
  after its `Retry-After`; only a 401 is about the key.
