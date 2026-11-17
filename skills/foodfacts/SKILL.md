---
name: foodfacts
description: Look up packaged food by barcode and check nutrition and ingredients with the FoodFacts API on RapidAPI - send an EAN or UPC barcode (8 to 14 digits) and get the product name, brands, quantity, Nutri-Score grade, NOVA group, Eco-Score grade, ingredients text, allergens, categories, image, and per-100g nutrition facts (energy kcal, fat, saturated fat, carbohydrates, sugars, fiber, proteins, salt, sodium), or search products by keyword with an optional Nutri-Score grade filter, or browse a whole food category like beverages or snacks. Data from Open Food Facts. Use when the user asks what is in a food product, scans or types a barcode, wants the nutrition facts, calories, sugar or protein content of a packaged food, checks whether a product contains an allergen such as milk, nuts or soy, compares products by Nutri-Score, builds a barcode scanner, pantry, grocery, diet or allergy app, needs healthier-swap candidates, or asks what nutriscoreGrade, novaGroup, ecoscoreGrade, nutrimentsPer100g, PRODUCT_NOT_FOUND, or INVALID_BARCODE mean. Unknown barcodes return an honest 404. For auth, hosts, and billing details use the newkwlab-router skill.
---

# FoodFacts — Barcode Nutrition Lookup and Food Search

## Overview

FoodFacts turns a barcode into structured nutrition data. Scan a product, call one endpoint, and get back a clean, flat record: product name, brands, categories, quantity, Nutri-Score grade, NOVA group, Eco-Score grade, ingredients, allergens, and per-100g values for energy, fat, saturated fat, carbohydrates, sugars, fiber, proteins and salt — normalized from Open Food Facts, the free crowd-sourced database of millions of packaged products worldwide.

Three endpoints: `/product` for one barcode, `/search` for keyword search with an optional Nutri-Score grade filter, `/category` for browsing a whole category. Built for barcode scanners, pantry and grocery apps, diet trackers, allergy checkers, and healthier-swap tools.

Full plumbing reference: the `newkwlab-router` skill.

## Access

- REST host: `foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com` — copied from the listing's Playground MCP panel and verified on 2026-11-17 with an MCP `tools/list` call (Browse_Category, Search_Products and Get_Product came back; see the repo TODO.md). Do not substitute the origin hostname — calls must go through RapidAPI so they bill against the caller's subscription.
- MCP: the RapidAPI hosted gateway at `https://mcp.rapidapi.com` serves one tool per endpoint with headers `x-api-host: foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com` and `x-api-key: YOUR_RAPIDAPI_KEY`.
- Subscribe first (free BASIC tier: 500 calls/month): https://rapidapi.com/binbin7418/api/foodfacts-product-nutrition-barcodes-and-food-search

## Endpoints

| Job | Endpoint | Params |
|---|---|---|
| One product by barcode | `GET /product` | `?barcode=` — 8–14 digit EAN/UPC |
| Search products | `GET /search` | `?q=` keyword; optional `grade=` (a, b, c, d or e), `page_size=` (1–25, default 10), `page=` |
| Browse a category | `GET /category` | `?name=` (e.g. `beverages`, `snacks`, `breakfasts`); optional `grade=`, paging as above |

`/product` returns `{product: {...}}` with fields: `barcode`, `name`, `brands`, `quantity`, `nutriscoreGrade`, `novaGroup`, `ecoscoreGrade`, `ingredientsText`, `allergens`, `categories`, `imageUrl`, `nutrimentsPer100g` (`energyKcal`, `fatG`, `saturatedFatG`, `carbohydratesG`, `sugarsG`, `fiberG`, `proteinsG`, `saltG`, `sodiumG`), and sometimes `note` (see Pitfalls). Search and category return `{count, page, pageSize, products: [{barcode, name, brands, nutriscoreGrade, novaGroup, imageUrl}]}`.

## Workflow

1. **Barcode in hand?** Call `/product`. Lead with the verdict-style facts a person actually wants: Nutri-Score, sugar and protein per 100g, and any allergens — then the full panel if they ask.
2. **Allergen checks:** read `allergens` (and the ingredients text) and answer from the record. If `allergens` is null or the record is sparse, say the data does not confirm it either way — never turn missing data into "safe". The physical label wins for allergies.
3. **No barcode?** `/search?q=` finds products by name or keyword; add `grade=a` or `grade=b` when the user wants healthier options. `/category?name=` browses a shelf (drinks, snacks, breakfasts) and pairs well with a grade filter for swap suggestions.
4. **Errors are answers:** `404 PRODUCT_NOT_FOUND` means the barcode is not in Open Food Facts — report that plainly instead of retrying variants. `400 INVALID_BARCODE` means the input was not 8–14 digits — ask for the digits under the barcode lines.
5. **Comparing products:** fetch each product, then compare like with like — per-100g values and grades, not per-package numbers, since package sizes differ.

## Real response (verified live, trimmed)

`GET /product?barcode=3017620422003`:

```json
{
  "product": {
    "barcode": "3017620422003",
    "name": "Nutella",
    "brands": "Ferrero, Nutella",
    "quantity": "400 g e",
    "nutriscoreGrade": "e",
    "novaGroup": 3,
    "ecoscoreGrade": "f",
    "ingredientsText": "sugar, palm oil",
    "allergens": "milk, nuts, soybeans",
    "nutrimentsPer100g": {
      "energyKcal": 539, "fatG": 30.9, "saturatedFatG": 10.6,
      "carbohydratesG": 57.5, "sugarsG": 56.3, "fiberG": 0,
      "proteinsG": 6.3, "saltG": 0.107, "sodiumG": 0.0428
    },
    "note": "Source data may be incomplete: Open Food Facts lists only 2 ingredient(s) for this product…"
  },
  "sources": ["Open Food Facts"]
}
```

## Pitfalls

- **Sparse records exist.** The data is crowd-sourced. When a product's ingredient list has two or fewer entries, the response adds a `note` field warning that the source data may be incomplete — surface that note; a very short ingredient list is partial, not exhaustive (the Nutella record above lists only "sugar, palm oil", which is the upstream record being sparse, not the recipe).
- **Grades can be `unknown` or absent** on sparse records; report them as missing, never invent a grade.
- Every response carries `_attribution` to Open Food Facts (ODbL) — keep the attribution when republishing data.

## Runnable curl

```bash
: "${RAPIDAPI_KEY:?set RAPIDAPI_KEY to your own RapidAPI key}"

# One product by barcode (host is a placeholder until backfilled - see TODO.md)
curl -s "https://foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com/product?barcode=3017620422003" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com"

# Healthier breakfast spreads: keyword search with a grade filter
curl -s "https://foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com/search?q=hazelnut%20spread&grade=b&page_size=10" \
  -H "x-rapidapi-key: $RAPIDAPI_KEY" \
  -H "x-rapidapi-host: foodfacts-product-nutrition-barcodes-and-food-search.p.rapidapi.com"
```

## Billing

BASIC $0 — 500 calls/month · PRO $5/month — 10,000 calls · ULTRA $14/month — 50,000 calls.
