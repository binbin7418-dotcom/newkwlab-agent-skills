# Pre-publish TODO — do not publish until every item is checked

This repo was built locally on 2026-11-17 and is **not published yet**.
The two API hosts that started as placeholders were backfilled and
verified on 2026-11-17 (see the checked items below). The remaining
checks are hard gates, not paperwork.

## Hosts: backfilled and verified (2026-11-17)

- [x] **Page2MD host** — done 2026-11-17. Verification: host copied
  from the listing's Playground MCP panel, then confirmed by the main
  session with a curl MCP `tools/list` call against
  `https://mcp.rapidapi.com`, which returned the Page2MD tools
  (Batch_Convert_Pages_to_Markdown, Convert_to_Markdown). Every
  placeholder in `llms.txt`, `skills/newkwlab-router/SKILL.md` and
  `skills/page2md/SKILL.md` has been replaced with
  `page2md-url-to-clean-markdown-api.p.rapidapi.com`.
- [x] **FoodFacts host** — done 2026-11-17. Verification: host copied
  from the listing's Playground MCP panel, then confirmed with a curl
  MCP `tools/list` call, which returned Browse_Category,
  Search_Products and Get_Product — matching the endpoint table in
  `skills/foodfacts/SKILL.md` (`/category`, `/search`, `/product`).
  Every placeholder has been replaced with
  `foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com`.
  (The backend hostname `foodfacts-api.pages.dev` is the origin
  server, not the RapidAPI host, and was not used.)

## Blocking: confirm Page2MD plan numbers

- [x] Done 2026-11-17. Plans read off the live listing through a
  read-only Studio check (nothing was changed): BASIC $0 — 100
  calls/mo · PRO $15/mo — 5,000 (overage $0.005/call) · ULTRA $49/mo —
  25,000 (overage $0.003/call); no MEGA plan, no Deprecated plan.
  Written into `README.md`, `llms.txt`, `skills/page2md/SKILL.md` and
  the router billing table in the same format as the other three APIs.

## Before publishing

- [ ] Re-run the no-secrets check: `grep -rE "x-rapidapi-key: [A-Za-z0-9]{20,}" .`
  and a general scan for key-shaped strings must return nothing. Every
  example key in this repo is the literal placeholder `YOUR_RAPIDAPI_KEY`.
- [ ] Re-verify the two known hosts still answer an MCP `tools/list`
  (EmailCheck: 3 tools; FlightScope: 8 tools — counts verified 2026-11-17).
- [ ] Confirm all four RapidAPI listing URLs in the README open publicly.
- [ ] Publish through the main session's GitHub flow (new repos default
  to private first per house rules; making this one public is a separate,
  explicit decision, since the repo is a distribution surface).
