# docuso — the docu.so plugin for Claude Code

**Markdown becomes anything.** [docu.so](https://docu.so) turns markdown into a finished, branded,
hosted page at a short link. This plugin gives Claude Code five skills and the remote MCP server, so a
page is one sentence away:

> "Make me a landing page for Northwind's new pricing, in our Company 1 styling"

## Install

```sh
claude plugin marketplace add jsval2/docuso-plugin
claude plugin install docuso@docuso-plugin
```

Then set two environment variables in the shell you run `claude` from:

```sh
export DOCUSO_API_URL=https://docu.so
export DOCUSO_API_KEY=dso_your_key_here
```

Mint the key in your dashboard under **API Keys**, and copy both lines from **Connect → Claude Skills**.
A key with the `write` scope can create, edit and publish; a `read` key can only list and read.

`DOCUSO_API_URL` is the workspace the skills talk to — set it to your own deployment if you run one.

## The skills

| Command | What it does |
|---|---|
| `/docuso:page` | One sentence to a published, branded page: picks a template, writes the markdown, applies your default brand, publishes, hands back the link. |
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
`list_templates`, `get_syntax_reference` and the rest — without a skill in the loop. It authenticates
with the same `DOCUSO_API_KEY`.

The endpoint is `https://docu.so/mcp`, written into the file because a plugin is installed once on your
machine and cannot be configured per workspace. If you run your own docu.so, add the server yourself
instead:

```sh
claude mcp add --transport http docuso https://your-deployment.example/mcp \
  --header "X-API-Key: ${DOCUSO_API_KEY}"
```

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
