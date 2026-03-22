# Fix Relative Path in Test Pipeline
**Date:** Sunday, March 22, 2026
**Time:** 11:31 AM PDT
---

## User

/document in @src/test_pipeline.py line 43 the recipe is hardcoded which is causing a pathing error. in @src/llm_pipeline/enhanced_recipe_generator.py line 243 onward the path is correctly initialized. the error likely exists due to another directory being inserted in front of the hardcoded dir path

## Claude

Documenting.

Looking at the issue: in `src/test_pipeline.py` line 43, the recipe path is hardcoded as `"../data/recipe_10813_best-chocolate-chip-cookies.json"`. This relative path assumes the script is run from `src/`, but if run from the project root or another directory, the path breaks.

In `src/llm_pipeline/enhanced_recipe_generator.py` line 243, `os.makedirs(os.path.dirname(output_path), exist_ok=True)` correctly handles directory creation from whatever path is passed in — so that's not the problem.

The likely issue: when the pipeline runs, the working directory may not be `src/`, causing `../data/` to resolve incorrectly. The fix would be to compute the path relative to the script's location rather than relying on the working directory.

## User

exactly, because i am running this from root compute path relative to scripts location

## Claude

Fixing `src/test_pipeline.py` to compute paths relative to the script's location.

```tool
[Tool: Read] src/test_pipeline.py
```

```tool
[Tool: Edit] src/test_pipeline.py — use Path(__file__).resolve().parent to compute script directory, then build recipe path relative to that
```

Fixed both paths in `test_pipeline.py`. Both `test_single_recipe()` and `test_all_recipes()` now resolve `../data` relative to the script's location using `Path(__file__).resolve().parent`, so it works regardless of where you run it from.

## User

I am done documenting
