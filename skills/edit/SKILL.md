---
name: edit
description: Edit an existing docu.so document — change its text, add or rewrite a section, fix a typo, restyle it with a different brand or template. Use when someone wants to change a document or page they already have on docu.so.
---

You edit docu.so documents in place. A published document keeps its link: an edit is live at the same
URL as soon as it is saved.

Read `${CLAUDE_PLUGIN_ROOT}/reference/api.md` first: the base URL, the key, the one envelope and what
each error code means. Everything here assumes it.

## Step 1 — find the document

If the user named one (`/docuso:edit Q2 OKRs`, or a shortcode), match it by title or shortcode and go
straight to step 2. Otherwise:

```bash
curl -s "${DOCUSO_API_URL}/api/v1/documents?limit=10" -H "X-API-Key: ${DOCUSO_API_KEY}"
```

```
Your recent documents:
 1. Website Redesign Proposal (published) — x7k9m2fq — updated 2h ago
 2. Q2 OKRs (draft) — a3b8n1kp — updated 1d ago
```

Ask which one. A number, a title or a shortcode all work.

## Step 2 — read it

```bash
curl -s "${DOCUSO_API_URL}/api/v1/documents/${DOC_ID}" -H "X-API-Key: ${DOCUSO_API_KEY}"
```

The full markdown is `.data.markdown`; a list row never carries it. If the document is long, show its
headings rather than the whole thing and ask which part to change.

## Step 3 — agree the change

Ask what to change if the user has not already said. Then apply it to the markdown and summarise:

> **Changes:**
> - Timeline pushed back two weeks
> - Added a Pricing section before the close

Get a yes before saving. Preserve every section the user did not ask about — an edit never loses
content the user did not mention.

If the change adds a block, read the dialect first and write it exactly as the reference spells it:

```bash
curl -s "${DOCUSO_API_URL}/api/v1/syntax?format=markdown"
```

Do not write a block from memory. Content only: no HTML, no CSS.

## Step 4 — save

Send only the fields that changed. Write the body to a file so the shell cannot mangle it:

```bash
cat > /tmp/docuso-edit.json <<'JSON'
{"markdown": "…"}
JSON
curl -s -X PATCH "${DOCUSO_API_URL}/api/v1/documents/${DOC_ID}" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${DOCUSO_API_KEY}" \
  --data-binary @/tmp/docuso-edit.json
```

`title`, `template` and `brand_profile_id` are editable the same way — that is how a document is
restyled with another brand (`GET /api/v1/brands`) or given a template (`GET /api/v1/templates`)
without touching a word of its text.

Then:

> Saved and live at: {url}

for a published document, using `.data.url` verbatim — or, for a draft:

> Saved as a draft. Publish it with /docuso:publish?

## Rules

- Never lose content. Sections the user did not mention come back unchanged.
- Never save without showing what changed first.
- Ask when the instruction is ambiguous rather than guessing.
- Markdown is capped at 65,536 characters; over that the PATCH is a 422 naming `markdown` and nothing
  is saved. Say so before sending an edit that would cross it.
- A 429 is retried once after its `Retry-After`; nothing else is retried automatically.
