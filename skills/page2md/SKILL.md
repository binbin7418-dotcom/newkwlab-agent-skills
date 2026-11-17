---
name: page2md
description: Convert web pages to clean Markdown with the Page2MD API on RapidAPI - send a URL, get back the page content as Markdown with title, byline and excerpt, ready for LLM and RAG pipelines, or send a batch of URLs in one call and get one result per URL in request order. Use when the user asks to convert a URL to Markdown, fetch a web page as Markdown, scrape an article into Markdown, turn a reading list, bookmark list, RSS backlog or list of links into Markdown notes, ingest web pages into a RAG pipeline or vector store, archive web content, migrate content off a website, extract the readable article text from a page without nav, ads and boilerplate, or asks what markdown, title, byline, excerpt, or the error codes CONTENT_TOO_SHORT, 400, 413, 415, 422, 502 or 504 from Page2MD mean. One URL per convert call; batch accepts an array of URLs in a single call. For auth, hosts, and billing details use the newkwlab-router skill.
---

# Page2MD — Web Page to Clean Markdown

## Overview

Page2MD turns a URL into the thing an LLM actually wants: the page's content as clean Markdown, with its `title`, `byline` and `excerpt` separated out — no nav bars, cookie banners, or footer boilerplate to tokenize and throw away. Built for RAG ingestion, reading-list processing, archiving, and content migration.

Two shapes: `POST /convert` for one URL, `POST /convert/batch` for a list of URLs in a single call (results come back in request order, one per URL, each either a convert result or an error object for that URL).

Full plumbing reference: the `newkwlab-router` skill.

## Access

- REST host: `page2md-url-to-clean-markdown-api.p.rapidapi.com` — copied from the listing's Playground MCP panel and verified on 2026-11-17 with an MCP `tools/list` call (the Page2MD convert tools came back; see the repo TODO.md). Do not call the origin server directly — calls must go through RapidAPI so they bill against the caller's subscription.
- MCP: the RapidAPI hosted gateway at `https://mcp.rapidapi.com` serves one tool per endpoint with headers `x-api-host: page2md-url-to-clean-markdown-api.p.rapidapi.com` and `x-api-key: YOUR_RAPIDAPI_KEY`.
- Subscribe first (free BASIC tier on the listing): https://rapidapi.com/binbin7418/api/page2md-url-to-clean-markdown-api
- There is also a free, no-key web tool for one-off human use: https://page2md-api.pages.dev/url-to-markdown — fine for a single page a person asked about; use the API for anything programmatic or repeated.

## Endpoints

| Job | Endpoint | Body | Returns |
|---|---|---|---|
| Convert one page | `POST /convert` | `{"url": "https://…"}` | `{markdown, title, url, byline, excerpt}` |
| Convert many pages | `POST /convert/batch` | `{"urls": ["https://…", …]}` | array in request order; item = convert result or `{error: {code, message}}` |

Response fields: `markdown` (the full converted content), `title` (the page's title), `url` (the URL as converted), `byline` (author line when the page carries one), `excerpt` (short summary text when available).

## Workflow

1. **One URL?** One `/convert` call. Deliver the Markdown (or save it where the user wants it); mention the title so they can see the right page came back.
2. **A list of URLs?** Use `/convert/batch` — one call for the whole list instead of a loop of single calls. State the cost shape first: the batch is one request returning one result per URL.
3. **Per-URL errors are data, not failure.** In a batch, a dead or blocked URL comes back as an error object in its slot; report which URLs failed and why, and deliver the rest.
4. **Error codes** (single calls return them as HTTP status + `{error: {code, message}}`): 400 bad request (missing or malformed `url`), 413 page too large, 415 the URL did not return HTML, 422 the page could not be processed (for example `CONTENT_TOO_SHORT` — nothing readable to extract), 502 the fetch failed upstream, 504 the fetch timed out. Retry 502/504 once; do not retry 4xx unchanged.
5. **Respect the content.** Converted Markdown is the page author's text — quote and transform it for the user's own reading, RAG, or migration work; do not republish it wholesale as if it were original content.

## Real response (verified live, excerpt)

`POST /convert` with `{"url": "https://en.wikipedia.org/wiki/Markdown"}` returned a `markdown` field beginning:

```markdown
From Wikipedia, the free encyclopedia

| Markdown |  |
| --- | --- |
| Filename extensions | `.md`, `.markdown` |
| Internet media type | `text/markdown` |
| Developed by | John Gruber |
| Initial release | March 9, 2004 |
```

(excerpt trimmed; the full response also carries `title`, `url`, `byline` and `excerpt` fields.)

## Runnable curl

```bash
: "${RAPIDAPI_KEY:?set RAPIDAPI_KEY to your own RapidAPI key}"

# One page (host is a placeholder until backfilled - see TODO.md)
curl -s -X POST "https://page2md-url-to-clean-markdown-api.p.rapidapi.com/convert" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: page2md-url-to-clean-markdown-api.p.rapidapi.com" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://en.wikipedia.org/wiki/Markdown"}'

# A reading list in one call
curl -s -X POST "https://page2md-url-to-clean-markdown-api.p.rapidapi.com/convert/batch" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: page2md-url-to-clean-markdown-api.p.rapidapi.com" \
  -H "Content-Type: application/json" \
  -d '{"urls": ["https://en.wikipedia.org/wiki/Markdown", "https://example.com"]}'
```

## Billing

Plans (read off the live listing on 2026-11-17): BASIC $0 — 100 calls/mo · PRO $15/mo — 5,000 calls (overage $0.005/call) · ULTRA $49/mo — 25,000 calls (overage $0.003/call). Listing: https://rapidapi.com/binbin7418/api/page2md-url-to-clean-markdown-api
