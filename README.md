# docuso — the docu.so plugin for Claude Code

**Markdown becomes anything.** [docu.so](https://docu.so) turns markdown into a finished, branded,
hosted page at a short link. This plugin gives Claude Code five skills and the remote MCP server, so a
page is one sentence away:

> "Make me a landing page for Northwind's new pricing, in our Company 1 styling"

## Install

First, in the shell you run `claude` from, set two environment variables — the plugin reads both,
the skills and the MCP server alike:

```sh
export DOCUSO_API_URL=https://docu.so
export DOCUSO_API_KEY=dso_your_key_here
```

Mint the key in your dashboard under **API Keys**, and copy both lines from **Connect → Claude Skills**.
Give the key **both** the `read` and the `write` scope: `write` does not include `read`, so a
write-only key can create a document but cannot list your documents or read your brands, and
`/docuso:page` and `/docuso:list` stop at the first request that reads your account. A `read`-only
key can list and read, nothing more.

`DOCUSO_API_URL` is the workspace the plugin talks to — `https://docu.so`, or your own deployment if
you run one. No trailing slash.

Then install:

```sh
claude plugin marketplace add jsval2/docuso-plugin
claude plugin install docuso@docuso-plugin
```

## The skills

| Command | What it does |
|---|---|
| `/docuso:page` | One sentence to a published, branded page: picks a template, writes the markdown, applies your brand, publishes, hands back the link. |
| `/docuso:new` | Turns what you just worked on in the conversation into a document. |
| `/docuso:edit` | Changes an existing document in place. A published page keeps its link. |
| `/docuso:list` | What is in your workspace, with status and links. |
| `/docuso:publish` | Publishes a draft, or unpublishes a page. |

The skills call the REST API at `$DOCUSO_API_URL/api/v1` with `curl` and read the markdown dialect and
the template registry from the API at run time, so they never go stale as docu.so gains blocks and
templates. `reference/api.md` is the API contract they share.

## The MCP server

`.mcp.json` declares docu.so's remote MCP endpoint, so the same account is reachable as tools —
`create_document`, `update_document`, `publish_document`, `list_documents`, `list_brands`,
`list_templates`, `get_syntax_reference` and the rest — without a skill in the loop. It connects to
`$DOCUSO_API_URL/mcp` and authenticates with the same `DOCUSO_API_KEY`, so it always talks to the
workspace the skills talk to, and your key is only ever sent to the host you named.

If `claude mcp list` shows `plugin:docuso:docuso` failing:

- **`Missing environment variables: DOCUSO_API_URL`** — it is not set in the shell `claude` was started
  from.
- **`Dynamic Client Registration rejected …`** (or any other OAuth wording) — `DOCUSO_API_KEY` is
  missing, mistyped or revoked. docu.so's MCP server takes an API key, not an OAuth sign-in; when the
  key is refused, Claude Code tries OAuth instead and reports how that failed. Fix the key and restart
  `claude`.

## What is in here

```
.claude-plugin/plugin.json       the plugin manifest
.claude-plugin/marketplace.json  the marketplace this repo is
.mcp.json                        the remote MCP server
skills/<name>/SKILL.md           one skill each
reference/api.md                 the API contract the skills share
```

This repository is **generated**. It is published from the `plugin/` directory of docu.so's own
repository by `scripts/publish-plugin.mjs`, so the skills and the product cannot drift apart. Open
issues here; changes land upstream.
