# The docu.so API, as the skills use it

Every skill in this plugin talks to the same REST API. This file is the part they share; read it once
per session, before the first request.

## Where, and as whom

- **Base URL:** the `DOCUSO_API_URL` environment variable. There is no default — the workspace decides
  which docu.so it talks to, which is what lets a preview deployment be driven by the same skills.
- **Auth:** send `X-API-Key: ${DOCUSO_API_KEY}` on every request, read from the environment.
- If either variable is missing, stop and say: *"Set DOCUSO_API_URL to your docu.so workspace and
  DOCUSO_API_KEY to an API key. Both are on the Claude Skills page of your dashboard, under Connect."*
- **Never build a document link yourself.** Every document the API returns carries a `url` field — show
  that, and nothing else. It is `null` while the document is a draft.

## One envelope

Every success is `{ "data": … }`, with `{ "meta": { "total", "page", "limit" } }` beside it on a list.
Every failure is `{ "error": { "code", "message", "details"? } }`. A document is at `.data`; a list is
at `.data[]`. Read `error.code`, never the HTTP status alone.

## What a failure means — and do not blame the key for what is not the key

| Code | Status | What it is | What to do |
|---|---|---|---|
| `rate_limited` | 429 | Too many requests **from this address** — never a wrong key. Counted per address (120 a minute), not per key or account. | Say so, wait the `Retry-After` seconds, retry the same request once. |
| `forbidden` | 403 | The key is real but may not do this: a `read` key covers `GET`, `HEAD` and `OPTIONS`, a `write` key everything else. No key at all may touch `/api/v1/keys` — that needs a signed-in session. | Name the missing scope. A new key is the fix, not a retry. |
| `payload_too_large` | 413 | The body was over the bound and refused before it was read. | Send less. |
| `validation_error` | 422 | `details[]` names each field refused, as `{ "path", "message" }`. | Show them and fix the field. |
| `unauthorized` / `invalid_token` | 401 | **This** is the key: missing, wrong or revoked. | The one failure to answer with "check `DOCUSO_API_KEY`". |
| `not_found` | 404 | No such document, or it belongs to another account. The two are the same answer. | Do not retry. |

Do not retry automatically. The one exception is a 429, retried once after its `Retry-After`.

## The endpoints these skills use

| Endpoint | What it does |
|---|---|
| `GET /api/v1/syntax?format=markdown` | The markdown dialect docu.so understands, generated from the live registries. Public, no key needed. **Read it before writing markdown; never work from memory.** |
| `GET /api/v1/templates` | Every template: `id`, `name`, `description`, `mode`, `blocks`, `schema.sections`, `schema.required`, `starter`. Public, no key needed. |
| `GET /api/v1/brands` | The account's brand presets, each with `id`, `name` and `is_default`. At most one is the default, and an account can have none — see *Choosing a brand*. |
| `POST /api/v1/documents` | Create. Body: `title`, `markdown`, and optionally `template`, `brand_profile_id`, `mode`. |
| `GET /api/v1/documents` | List. `?page=` and `?limit=` walk the pages; a row carries no `markdown`. |
| `GET /api/v1/documents/:id` | One document, `markdown` included. |
| `PATCH /api/v1/documents/:id` | Update any of `title`, `markdown`, `template`, `brand_profile_id`. Only what you send changes. |
| `POST /api/v1/documents/:id/publish` | Publish. Answers with the document, now carrying a live `url`. |
| `POST /api/v1/documents/:id/unpublish` | Unpublish. The `url` goes back to `null` and the link stops working. |
| `DELETE /api/v1/documents/:id` | Delete. 204, no body. Gone for good. |

## Choosing a brand

A document takes its look from the brand preset named in `brand_profile_id`; without one it renders
in the base look, however good the account's brand book is. From `GET /api/v1/brands`:

1. If the user named a brand ("in our Company 1 styling"), match it by `name`, case-insensitively.
2. Otherwise take the one with `is_default: true`.
3. **None is marked default but the account has some** — that happens, and it is not a reason to
   leave the document unbranded. One brand: use it. Several: the one whose `name` best matches what
   the document is about, and failing that the most recently updated. Say which one you used and why,
   in one line.
4. No brands at all: send no `brand_profile_id`, and say that a brand preset in the dashboard will
   restyle the document with no edit to it.

## Writing markdown for docu.so

- The markdown is **content only**. Never write HTML, never write CSS: the brand applies itself, and the
  template decides the structure. A page looks designed because of what it is created with, not because
  of markup in the body.
- Blocks are docu.so's additions to markdown. Which ones exist, what attributes each takes and how to
  write them is `GET /api/v1/syntax?format=markdown`, read at the time you write — this plugin
  deliberately carries no copy of the dialect, because the registry it comes from grows.
- A document's markdown is capped at **65,536 characters**; over that the API answers 422 naming
  `markdown`. Trim before sending rather than letting the save fail.
- A title is at most **200 characters**; over that the API answers 422 naming `title`.

## Calling it

`curl` is enough, and the two variables are already in the environment:

```bash
curl -s "${DOCUSO_API_URL}/api/v1/documents" -H "X-API-Key: ${DOCUSO_API_KEY}"
```

Put a JSON body in a file and send it with `--data-binary @file` rather than inlining it: markdown is
full of quotes and newlines, and a shell-escaped heredoc is where documents get mangled.
