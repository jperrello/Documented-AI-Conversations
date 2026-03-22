# Embedding Clustering Pipeline Refactor
**Date:** Sunday, March 22, 2026
**Time:** Current session
---

## User

/document

## Claude

Documenting.

## User

our user story is:
I want to see the enhanced recipes and inspect the line-level diffs explaining which community suggestions were applied and why.

Our problem statement is:
The pipeline processes only one randomly selected review and extracts only one modification from it, remember the goal is to aggregate all community-tested tweaks into a single enhanced recipe.

And our solution approach is:
Embedding-Based Clustering with Batched Extraction. All reviews where has_modification=True are embedded using text-embedding-3-small, then clustered by cosine similarity to collapse semantically duplicate modifications into distinct groups. Within each cluster, the highest-rated review is selected as the representative, preserving attribution. The representative review texts are then batched into a single LLM call to gpt-3.5-turbo for structured modification extraction, where each representative produces one ModificationObject. This ensures every unique modification surfaces (no random sampling), scales sublinearly with review count (clustering absorbs duplicates before the LLM sees anything), and confines the expensive LLM call to a bounded set of deduplicated representatives rather than raw review volume.

We will ultimately only need to change two files:
Tweak_extractor.py:
Extract_single_modification need to be refactored to extract_modifications. This new method will filter reviews with has_modifcation=True then call text-embedding-3-small on each review's text to get embedding vectors
Cluster these vectors with cosine similarity threshold using something like scikit-learn
Select highest rated review in each cluster as representative
Batch each cluster's representative review into a single LLM prompt that returns a list of ModificationObjects (one per representative)
Lastly, return a list of tuple [ModificationObject, Review] instead of a single tuple
pipeline.py:
Update the call to receive a list of (modification, review) pairs instead of one
downstream format needs to handle multiple modifications

## Claude

Got it. Clear plan — two files, embedding-cluster-extract in `tweak_extractor.py`, multi-modification plumbing in `pipeline.py`.

Read all source files: `tweak_extractor.py`, `pipeline.py`, `models.py`, `prompts.py`, `enhanced_recipe_generator.py`, `recipe_modifier.py`.

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/tweak_extractor.py
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/pipeline.py
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/models.py
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/prompts.py
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/enhanced_recipe_generator.py
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/recipe_modifier.py
```

**Current flow (single modification):**
1. `tweak_extractor.extract_single_modification` → picks 1 random review → returns `(ModificationObject, Review)`
2. `recipe_modifier.apply_modification` → applies it → returns `(Recipe, [ChangeRecord])`
3. `enhanced_recipe_generator.generate_enhanced_recipe` → takes single mod + review + changes → wraps into `EnhancedRecipe`

**Good news:** `RecipeModifier` already has `apply_modifications_batch` (line 192) and `EnhancedRecipeGenerator` already handles `List[ModificationApplied]` internally. So the downstream is mostly ready — the main work is in `tweak_extractor.py` and `pipeline.py` as you said.

**What needs to happen:**

In `tweak_extractor.py`:
- New `extract_modifications` method: filter → embed → cluster → select reps → batched LLM call → return `list[tuple[ModificationObject, Review]]`
- New batched prompt in `prompts.py` for multi-review extraction

In `pipeline.py`:
- Replace `extract_single_modification` call with `extract_modifications`
- Loop over pairs, apply each via `apply_modification` (or use `apply_modifications_batch`)
- Refactor `generate_enhanced_recipe` call to accept multiple `(modification, review, changes)` triples

In `enhanced_recipe_generator.py`:
- Refactor `generate_enhanced_recipe` to accept a list of `(modification, review, change_records)` instead of a single one (it already wraps in a list internally, so this is minor)

Dependencies needed: `scikit-learn`, `numpy`, `openai` (already present for embeddings).

Want me to start implementing, or do you want to adjust anything in this plan first?

## User

go

## Claude

Installed `scikit-learn` and `numpy` via `uv add`.

```tool
[Tool: Bash] uv add scikit-learn numpy
```

Edited `prompts.py` — added `build_batched_prompt` for multi-review extraction.

```tool
[Tool: Edit] prompts.py - added build_batched_prompt function
```

Edited `tweak_extractor.py` — added `embed()`, `cluster()`, `select_representatives()`, replaced `extract_single_modification` with `extract_modifications`.

```tool
[Tool: Edit] tweak_extractor.py - replaced extract_single_modification with embed/cluster/extract_modifications
```

Edited `pipeline.py` — updated `process_single_recipe` to call `extract_modifications`, use `apply_modifications_batch`, pass triples to generator.

```tool
[Tool: Edit] pipeline.py - updated to multi-modification flow
```

Edited `enhanced_recipe_generator.py` — changed `generate_enhanced_recipe` signature to accept `triples: List[Tuple[ModificationObject, Review, List[ChangeRecord]]]`.

```tool
[Tool: Edit] enhanced_recipe_generator.py - updated to accept triples
```

Verified all imports work.

```tool
[Tool: Bash] .venv/bin/python -c "from src.llm_pipeline.pipeline import LLMAnalysisPipeline; print('imports ok')"
```

All changes done. Summary:

**`prompts.py`** — Added `build_batched_prompt` that takes a list of review dicts and produces a single prompt asking for a `{"modifications": [...]}` JSON array, one `ModificationObject` per review.

**`tweak_extractor.py`** — Replaced `extract_single_modification` with:
- `embed()` — calls `text-embedding-3-small` on review texts
- `cluster()` — `AgglomerativeClustering` with cosine distance threshold (default 0.15)
- `select_representatives()` — picks highest-rated review per cluster
- `extract_modifications()` — orchestrates filter → embed → cluster → select → batched LLM call → returns `list[tuple[ModificationObject, Review]]`

**`pipeline.py`** — `process_single_recipe` now calls `extract_modifications`, feeds the list into `apply_modifications_batch` (already existed in `RecipeModifier`), and passes triples to the generator.

**`enhanced_recipe_generator.py`** — `generate_enhanced_recipe` now accepts `triples: List[Tuple[ModificationObject, Review, List[ChangeRecord]]]` instead of a single modification/review/changes.

Want to run it against a recipe to test?
