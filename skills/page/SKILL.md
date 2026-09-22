---
name: page
description: One sentence to a published, branded docu.so page. Picks the right template, reads the live markdown syntax, writes the page, applies the account's brand, publishes it and returns the link. Use whenever someone asks for a landing page, a launch page, a video watch page, a proposal, an invoice, an SOP, a one-pager or any web page they want a link to.
---

You turn one sentence into a published, branded page on docu.so. The user should not have to open a
dashboard, pick a template by hand or choose a colour: everything below happens in one pass, and what
comes back is a link.

Read `${CLAUDE_PLUGIN_ROOT}/reference/api.md` first — the base URL, the key, the one envelope and what
each error code means. Everything here assumes it.

## The shape of the job

> "Make me a landing page for Northwind's new pricing"

becomes: the template registry decides the structure, the syntax reference decides what may be written,
the account's brand decides how it looks, and the user gets the published page's own link back.

## Step 1 — ask the product what it can do

Two public endpoints, no key required. Fetch both before writing anything.

```bash
curl -s "${DOCUSO_API_URL}/api/v1/templates"
curl -s "${DOCUSO_API_URL}/api/v1/syntax?format=markdown"
```

- **Templates.** Each one carries `id`, `name`, `description`, `mode`, the `blocks` it expects and a
  `schema` (`sections`, and the subset in `required`). Pick the template whose `description` and
  `sections` match what the user asked for; if the user named one, use that. If nothing fits, use
  `document`.
- **The syntax reference** is the markdown dialect, generated from the live registry. It is the only
  description of the dialect you may work from — do not write a block from memory, and do not invent
  attributes. If a block you want is not in there, it does not exist.

Never fetch either of these twice in one session.

## Step 2 — find the brand

```bash
curl -s "${DOCUSO_API_URL}/api/v1/brands" -H "X-API-Key: ${DOCUSO_API_KEY}"
```

- If the user named a brand ("in our Company 1 styling"), match it by `name`, case-insensitively.
- Otherwise take the one with `is_default: true`.
- If the account has no brands at all, carry on without one and say at the end that the page is in the
  default look and that a brand preset in the dashboard will restyle it with no edit to the page.

Keep its `id` for `brand_profile_id`. **This is the step that makes the page branded** — a document
created without it renders in the base look however good the brand book is.

## Step 3 — write the markdown

Content only: no HTML, no CSS, no front matter, no `<style>`. Write to the template's schema — one
section of markdown per entry in `schema.sections`, in that order, and never skip one listed in
`schema.required`. If the template's `starter` is not empty, it is the shape to follow.

Use the blocks the template lists in `blocks`, written exactly as the syntax reference spells them.
Prose-only markdown is a valid page; a landing page that is only prose is not.

The first `#` heading becomes the document title. Fill in what the user actually said and nothing more:
invent no prices, no testimonials, no client names, no dates. Where the user's sentence leaves a real
gap, write an obvious placeholder and list the placeholders at the end so they can fix them in one pass.

## Step 4 — create it

Write the body to a file first, so no quote or newline is mangled by the shell.

```bash
cat > /tmp/docuso-page.json <<'JSON'
{"title": "…", "markdown": "…", "template": "landing", "brand_profile_id": "…"}
JSON
curl -s -X POST "${DOCUSO_API_URL}/api/v1/documents" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${DOCUSO_API_KEY}" \
  --data-binary @/tmp/docuso-page.json
```

`template` is a template `id` from step 1; the document takes that template's `mode`. Omit
`brand_profile_id` only when the account has no brands. A 422 naming `template` means the id is not one
the registry has — re-read step 1 rather than guessing another.

Take `.data.id` from the answer.

## Step 5 — publish, and hand back the link

```bash
curl -s -X POST "${DOCUSO_API_URL}/api/v1/documents/${DOC_ID}/publish" \
  -H "X-API-Key: ${DOCUSO_API_KEY}"
```

Report `.data.url` verbatim — never a link you assembled:

> Published: {url}
> Template: Landing page · Brand: Company 1

Then, in one short line each: any placeholder you left, and that `/docuso:edit` changes the page in
place (the link stays the same).

## Rules

- One pass. Do not ask which template, which brand or whether to publish — decide, do it, and say what
  you decided. Ask only when the request is too thin to write anything at all.
- If publishing fails after the document was created, say the draft exists and name it; do not delete it.
- Never invent content, and never write a link yourself.
