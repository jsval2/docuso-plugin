---
name: new
description: Create a new docu.so document from what is already in this conversation, or from a description. Use when someone wants a proposal, report, spec, SOP, guide, changelog, meeting notes or any piece of writing turned into a hosted document they can share.
---

You create docu.so documents — markdown in, a hosted, branded page out.

Read `${CLAUDE_PLUGIN_ROOT}/reference/api.md` first: the base URL, the key, the one envelope and what
each error code means. Everything here assumes it.

If the user asked for a *page* rather than a document — a landing page, a watch page, an invoice, an
SOP from a template — use `/docuso:page` instead: it picks the template and the brand and publishes in
one pass.

## Step 1 — find the content

Look back through this conversation for something the user would want to share: a proposal, a summary,
a report, a spec, an SOP, a guide, a post, documentation, release notes.

**If you find it,** say what you found and suggest a title:

> I see we just worked out a proposal for Acme's website redesign. Want me to publish it on docu.so?
>
> **Suggested title:** Website Redesign Proposal — Acme

Wait for a yes or a better title.

**If you do not,** ask what to create, then write it. Never invent facts: use what was discussed or what
the user describes.

## Step 2 — write the markdown

Before you write a single block, read the dialect:

```bash
curl -s "${DOCUSO_API_URL}/api/v1/syntax?format=markdown"
```

It is generated from docu.so's live registry, so it is the only description of the syntax you may work
from — do not write a block from memory and do not invent attributes. Ordinary markdown (headings,
lists, tables, links, code) always works.

Content only: no HTML, no CSS. A clear `# Title` first, then `##` sections. Keep the title under 100
characters and the markdown under 65,536.

## Step 3 — create it

Write the body to a file so the shell cannot mangle a quote or a newline:

```bash
cat > /tmp/docuso-new.json <<'JSON'
{"title": "…", "markdown": "…"}
JSON
curl -s -X POST "${DOCUSO_API_URL}/api/v1/documents" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${DOCUSO_API_KEY}" \
  --data-binary @/tmp/docuso-new.json
```

The document is at `.data`: take its `id` and `shortcode`. A failure is `error.code` and
`error.message` — say what that code means; only a 401 is about the API key.

**To brand it or to give it a template**, send `brand_profile_id` (from `GET /api/v1/brands`, the one
with `is_default: true`) and `template` (an `id` from `GET /api/v1/templates`) in the same body.
Without a brand the document renders in the base look.

## Step 4 — offer to publish

> Created as a draft. Publish it now so you have a link to share?

On yes:

```bash
curl -s -X POST "${DOCUSO_API_URL}/api/v1/documents/${DOC_ID}/publish" \
  -H "X-API-Key: ${DOCUSO_API_KEY}"
```

> Published: {url}

Use `.data.url` from that answer, verbatim.

On no:

> Saved as a draft. Publish it later with /docuso:publish.

## Rules

- Never fabricate content. What was discussed, or what the user describes — nothing else.
- Show the structure and get a yes before creating something the user only described in one line.
- A 429 is retried once after its `Retry-After`; nothing else is retried automatically.
