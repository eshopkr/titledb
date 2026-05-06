# eshopkr/titledb — Output Specification

Generated: 2026-05-06

## Identifiers

| Field | Common Name | Format | Example | Description |
|-------|------------|--------|---------|-------------|
| `id` | Title ID | 16-char hex | `01004AB00A260000` | Software title identifier, shared across regions |
| `nsuId` | eShop ID | Integer | `70010000008802` | eShop listing identifier, region-specific |

## Content Type (Title ID suffix)

| Suffix | Type | Description |
|--------|------|-------------|
| `0x000` | Base application | The main game |
| `0x800` | Patch/Update | Game updates |
| `0x001`–`0x7FF` | Add-on content | DLC |

## Output Files

### KR.ko.global.json

Full enriched KR eShop dataset. Object keyed by KR eShop ID (string).

Each entry contains all original fields from `KR.ko.json` plus:
- `region`: Corrected to `"KR"` (source data has `null`)
- `titleIds`: (only if matched) Region mappings grouped by Title ID

```json
{
  "<krEshopId>": {
    "id": "string|null — Title ID (16-char hex)",
    "nsuId": "number — eShop ID",
    "name": "string — Title name",
    "region": "string — Always 'KR'",
    "publisher": "string|null",
    "releaseDate": "number|string|null — YYYYMMDD or YYYY-MM-DD",
    "size": "number|null — File size in bytes",
    "bannerUrl": "string|null — Banner image URL",
    "iconUrl": "string|null — Icon image URL",
    "category": "string[] — Game categories",
    "languages": "string[] — Supported languages",
    "titleIds": {
      "<titleId>": {
        "<foreignEshopId>": ["regionCode", "..."]
      }
    }
  }
}
```

### KR.ko.match.json

Lightweight mapping — only the `titleIds` structure keyed by KR eShop ID.

```json
{
  "<krEshopId>": {
    "<titleId>": {
      "<foreignEshopId>": ["regionCode", "..."]
    }
  }
}
```

### KR.ko.base.json

Same structure as `KR.ko.global.json`, filtered to base games only (Title ID suffix `000` or entries without a Title ID).

### KR.ko.extra.json

Same structure as `KR.ko.global.json`, filtered to DLC (`001`–`7FF`) and updates (`800`) only.

### KR.ko.review.json

Low-confidence name matches pending human verification.

```json
{
  "<krEshopId>": {
    "name": "string — KR title name",
    "id": "string|null — KR Title ID",
    "publisher": "string|null — KR publisher",
    "releaseDate": "number|string|null",
    "size": "number|null — bytes",
    "candidates": {
      "<candidateTitleId>": {
        "<candidateEshopId>": {
          "regions": ["string — region codes"],
          "name": "string — Candidate title name",
          "publisher": "string|null",
          "releaseDate": "number|string|null",
          "size": "number|null",
          "bannerUrl": "string|null",
          "iconUrl": "string|null",
          "reason": "string — Why auto-match failed (e.g. publisher_mismatch+date_too_far+size_diff_35pct)"
        }
      }
    }
  }
}
```

#### Reason Codes

| Code | Meaning |
|------|---------|
| `publisher_mismatch` | Both publishers exist but don't match |
| `publisher_not_comparable` | One/both publisher names normalize to empty |
| `date_too_far` | Both dates exist but differ by >730 days |
| `date_unavailable` | One/both dates missing |
| `size_diff_NNpct` | Sizes differ by NN% (20–50%) |
| `size_not_comparable` | One/both sizes missing or below 10MB |

### spec.json

Machine-readable schema metadata for all output files.

### review.html

Visual review page with thumbnail images for human review of low-confidence matches.

### index.html

Dashboard with matching statistics, file descriptions, and navigation links.

## Matching Strategy

See [MATCHING_STRATEGY.md](../MATCHING_STRATEGY.md) for full details.

1. **Title ID exact match** — highest confidence
2. **Normalized name match + confirmation** — publisher/date/size signals required; cross-type matches (base↔DLC) rejected
3. **Single-candidate size promotion** — 1 candidate Title ID + size <20% + both >10MB
4. **Single-candidate fallback promotion** — 1 candidate Title ID + non-generic name (8+ chars) + size <50%
5. **pHash icon match** — perceptual hash comparison of icon images (US only, hamming distance <5%)
6. **Review queue** — remaining unconfirmed candidates
