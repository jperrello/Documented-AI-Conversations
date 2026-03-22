# Recipe Enhancement Pipeline Showcase HTML
**Date:** March 22, 2026
**Time:** Current session
---

## User

/document our goal is to showcase my new pipeline that showcases recipe modifications. we will create a simple html page that will take a recipe from @data/enhanced/ and show the original recipe. then below there will be a cards that a user can click to reveal modified recipes. each card will have a title so the user has a rough idea of its modification before clicking. each card should include the modifications, reasoning, and the author of the representative claim. navigate @src/llm_pipeline/prompts.py @src/llm_pipeline/tweak_extractor.py @src/llm_pipeline/pipeline.py and @src/llm_pipeline/enhanced_recipe_generator.py to see the pipeline in action

## Claude

Documenting.

Let me examine the enhanced recipe data structure to understand what we're working with.

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/data/enhanced/enhanced_10813_best-chocolate-chip-cookies.json
```

I've reviewed the pipeline files and the enhanced data. The structure is clear:

- Each enhanced JSON has: original recipe fields (`title`, `ingredients`, `instructions`, `description`), plus `modifications_applied` — an array where each entry has `source_review` (with `text`, `reviewer`, `rating`), `modification_type`, `reasoning`, and `changes_made` (with `type`, `from_text`, `to_text`, `operation`).

Let me check one more enhanced file for variation, then build the HTML.

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/data/enhanced/enhanced_77935_creamy-sweet-potato-with-ginge.json
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/data/enhanced/enhanced_19117_spicy-apple-cake.json
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/data/enhanced/enhanced_144299_nikujaga-(japanese-style-meat-.json
```

```tool
[Tool: Read] /Users/jperr/Documents/Casper-Assessment/data/recipe_10813_best-chocolate-chip-cookies.json (first 30 lines for original recipe structure)
```

Now I have everything I need. Built the HTML showcase page at `showcase.html` — a self-contained single file with all 4 enhanced recipes embedded. Features:

- Recipe selector buttons at top to switch between recipes
- Dark summary bar showing modification count, total changes, servings, and change types
- Original recipe displayed with ingredients and instructions in a two-column grid
- Expandable modification cards below, each showing:
  - Card title (the reasoning text) with a colored badge for modification type
  - On click: reasoning, change details (strikethrough originals with green replacements), full reviewer quote, and star rating

```tool
[Tool: Write] /Users/jperr/Documents/Casper-Assessment/showcase.html (full HTML page, ~500 lines)
```

Opened the page in browser and used Playwright to test:

## User

i want you to use playwright to test formatting and clicking

## Claude

Started local HTTP server and navigated Playwright to the page.

```tool
[Tool: Playwright navigate] http://localhost:8787/showcase.html
```

```tool
[Tool: Playwright screenshot] showcase-initial.png - full page initial state
```

Page renders correctly with all 4 recipe selector buttons, summary bar, original recipe in two-column layout, and 4 collapsed modification cards.

Clicked first modification card ("Adjusting sugar and flour quantities"):

```tool
[Tool: Playwright click] First modification card
```

```tool
[Tool: Playwright screenshot] showcase-card-open.png - card expanded
```

Card expands showing: strikethrough original values with arrows to new values, full review quote with emoji, and 5-star rating. Chevron rotates on open.

Switched to "Creamy Sweet Potato With Ginger Soup":

```tool
[Tool: Playwright click] Creamy Sweet Potato With Ginger Soup button
```

Recipe view updates — shows 5 modifications with diverse badge types (Quantity Adjustment, Technique Change, Addition, Ingredient Swap). Clicked first soup card:

```tool
[Tool: Playwright click] First soup modification card
```

```tool
[Tool: Playwright screenshot] showcase-soup-expanded.png - soup recipe with expanded card
```

Everything works as expected:

- **Recipe selector** switches between all 4 recipes, updating the summary bar, original recipe, and modification cards
- **Modification cards** toggle open/closed on click with the chevron rotating
- **Expanded cards** show: reasoning (italic), change details (strikethrough original with green replacement or green "+ added" items), the full reviewer quote, and star rating
- **Badge colors** differentiate modification types (blue for quantity, green for addition, orange for technique, pink for ingredient swap)

The page is at `showcase.html` in the project root.

## User

I am done documenting

## Claude

Finalizing and pushing the conversation document.
