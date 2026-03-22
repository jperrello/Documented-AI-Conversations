# Heuristic Review Splitting for Multi-Topic Modifications
**Date:** March 22, 2026
**Time:** Current session
---

## User

/document I was testing out my LLM pipeline and I noticed two problems with the result, the enhanced recipe overwrote the initial recipe entirely. GPT 3.5 turbo was trying to make one recipe based on all of the modifications, which resulted in a strange amalgamation off all the feedback from reviews.
I wanted the original recipe with each modification as suggested.
Additionally, I noticed with clustering, some of the modifications were forgotten if there were multiple. "I used 1 tsp. of salt instead of 1/2 tsp. and omitted the nuts" would only return a change in the salt.
The modified recipe is fine, as I still have the original stored.
The main issue is that my solution implements one cluster for one review which is inherently flawed because one cluster for a multi-topic review is bound to produce error.
The ModificationObject extraction schema encourages one modification per entry

The solution to this problem is to add heuristic splitting. Before I noticed that there were Regex patterns inside of scraper_v2.py. What if we split review text on coordinating conjunctions and clause boundaries? We can use those regex patterns to split before each new action verb match.
Example: "I used 1 tsp. of salt instead of 1/2 tsp. and omitted the nuts"
splitting on r"I (added|used|substituted|replaced|made with|changed)" and r"(instead of|rather than|in place of)" yield two clauses, each containing one atomic modification.

## Claude

Documenting.

You've identified two distinct issues with the pipeline:

1. **Recipe overwrite problem** — The enhanced recipe replaces the original entirely instead of presenting the original with each suggested modification separately.

2. **Multi-topic review loss** — Clustering assigns one cluster per review, so compound modifications like "I used 1 tsp. of salt instead of 1/2 tsp. and omitted the nuts" lose the second modification. The `ModificationObject` schema expects one modification per entry, but reviews often contain multiple.

Your proposed fix: heuristic splitting using regex patterns already present in `scraper_v2.py` — split review text on coordinating conjunctions and clause boundaries (action verb matches like `I added|used|substituted...` and contrast markers like `instead of|rather than|in place of`) to decompose compound reviews into atomic modification clauses before clustering.

## User

yes do you mind modifying @src/llm_pipeline/enhanced_recipe_generator.py @src/llm_pipeline/models.py @src/llm_pipeline/prompts.py @src/llm_pipeline/tweak_extractor.py @src/llm_pipeline/recipe_modifier.py

## Claude

Read all five files plus `pipeline.py` and `scraper_v2.py` for full context.

Changes made across 3 files (prompts.py and recipe_modifier.py needed no changes):

**`tweak_extractor.py`** — Added `split_review()` function with two regex patterns (`ACTION_VERB_RE` for action verbs, `CONTRAST_RE` for contrast markers). Splits compound reviews on coordinating conjunctions and clause boundaries before each new action verb match. Integrated into `extract_modifications()` — compound reviews are expanded into atomic clauses before embedding/clustering.

```tool
[Tool: Read] src/scraper_v2.py lines 76-90 — checked existing tweak_patterns regex
```

```tool
[Tool: Edit] tweak_extractor.py — added import re, ACTION_VERB_RE, CONTRAST_RE, split_review() function
```

```tool
[Tool: Edit] tweak_extractor.py — integrated split_review into extract_modifications() before embedding
```

**`models.py`** — Added `original_ingredients` and `original_instructions` fields to `EnhancedRecipe` so the original recipe is preserved in the output.

```tool
[Tool: Edit] models.py — added original_ingredients, original_instructions to EnhancedRecipe
```

**`enhanced_recipe_generator.py`** — Populates the new original fields from `original_recipe` in `generate_enhanced_recipe()`. Updated `generate_comparison_data()` to read originals from the enhanced recipe object.

```tool
[Tool: Edit] enhanced_recipe_generator.py — populate original_ingredients/original_instructions
```

```tool
[Tool: Edit] enhanced_recipe_generator.py — updated comparison data to use stored originals
```

Tested `split_review()` standalone:
- `"I used 1 tsp of salt instead of 1/2 tsp and omitted the nuts"` → `['I used 1 tsp of salt instead of 1/2 tsp', 'and omitted the nuts']`
- `"I added a teaspoon of cream of tartar to the batter and omitted the water"` → `['I added a teaspoon of cream of tartar to the batter', 'and omitted the water']`
- `"I substituted almond milk for regular milk, doubled the vanilla, and skipped the cinnamon"` → `['I substituted almond milk for regular milk', 'doubled the vanilla', 'and skipped the cinnamon']`
- `"Great recipe, loved it!"` → `['Great recipe, loved it!']` (no split, no modification signal)
