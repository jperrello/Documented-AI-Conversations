# Enhanced vs Original Recipe JSON Comparison
**Date:** March 22, 2026
**Time:** 11:07 AM

---

## User

/document look at @data/enhanced/enhanced_10813_best-chocolate-chip-cookies.json and @data/recipe_10813_best-chocolate-chip-cookies.json what json fields are added to enhanced, modified, or kept the same?

## Claude

Documenting.

Comparing the two files:

**Added fields (new in enhanced):**
- `original_recipe_id` — references the source recipe
- `modifications_applied` — array of objects detailing each change with source review, modification type, reasoning, changes made, and confidence score
- `enhancement_summary` — object with total_changes, change_types, confidence_score, expected_impact
- `created_at` — timestamp of enhancement creation
- `pipeline_version` — version string

**Modified fields:**
- `recipe_id` — changed from `"10813"` to `"10813_enhanced"`
- `title` — appended `"(Community Enhanced)"`
- `ingredients` — values changed (white sugar halved to 0.5 cup, brown sugar increased to 1.5 cups, added "1 additional egg yolk")
- `prep_time`, `cook_time`, `total_time` — renamed from camelCase (`preptime`, `cooktime`, `totaltime`) to snake_case, and values set to `null`

**Kept the same:**
- `description`
- `servings`
- `instructions`

**Removed fields (present in original, absent in enhanced):**
- `url`
- `scraped_at`
- `rating`
- `nutrition`
- `author`
- `categories`
- `featured_tweaks`
- `reviews`

## User

is there difference between @src/data/enhanced/enhanced_10813_best-chocolate-chip-cookies.json and @data/enhanced/enhanced_10813_best-chocolate-chip-cookies.json

## Claude

```tool
[Tool: Read] {"file_path":"/Users/jperr/Documents/Casper-Assessment/src/data/enhanced/enhanced_10813_best-chocolate-chip-cookies.json"}
```

Yes, there are differences. Comparing:

**`src/data/enhanced/`** (1 modification applied):
- `ingredients` — keeps original sugar amounts (`1 cup white sugar`, `1 cup packed brown sugar`)
- `modifications_applied` — only 1 entry (egg yolk addition), no `confidence_score` field on modifications
- `enhancement_summary` — `total_changes: 1`, only `["addition"]`, no `confidence_score` field
- `reasoning` text: "...improving their texture."

**`data/enhanced/`** (2 modifications applied):
- `ingredients` — sugar amounts changed (`0.5 cup white sugar`, `1.5 cups packed brown sugar`)
- `modifications_applied` — 2 entries (egg yolk addition + sugar ratio adjustment), each has a `confidence_score` field
- `enhancement_summary` — `total_changes: 3`, `["quantity_adjustment", "addition"]`, has `confidence_score: 0.88`
- `reasoning` text: "...chewy." (shorter)

The `data/` version represents a later pipeline run that applied more modifications and added confidence scoring. The `src/data/` version is an earlier run with only the egg yolk change.

## User

I am done documenting.
