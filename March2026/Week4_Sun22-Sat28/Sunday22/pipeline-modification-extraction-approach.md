# Pipeline Modification Extraction Approach
**Date:** March 22, 2026
**Time:** Session start
---

## User

/document we are solving a problem where @src/llm_pipeline/pipeline.py processes only one randomly selected review and extracts only one modification from it. I want to share with you naive solutions we will not be using and why they do not work, but then I will share my two solution approaches. I want you to weigh the two solution approaches based on the following questions: Are we certain that the system parses out ALL the intended modifications? Does the system scale beyond the 5 examples? Are there poor assumptions embedded in the current implementation? Pick one best option with direct reasoning for why it is better at solving those questions.
Naive Approaches:
There are two different naive solutions to this problem:
First,  just loop extract_modification for each review that has has_modification=True.
This adds latency because there is an LLM call for each review, and we would burn tokens resending the initial recipe every single time just to change one review.
The cost for token is way too high for this approach
Second, we can keep only one call to the llm, but instead of one recipe modification being sent we send all of the reviews in one prompt.
GPT-3.5-turbo is way too small to handle this. If the number of reviews scales anywhere in the double digits, the context window will become far too bloated.
Third, we could use an LLM as a judge to filter the reviews before sending the modifications to the LLM to enhance the recipe. I would send ALL modification review texts in one call to GPT-3.5-turbo to the group, then send one batched extraction call with the group summaries + recipe context.
This approach will not work because we are relying on 3.5 (older model) to make accurate groupings and not hallucinate different results than the JSON.
This increases the cost of the operation as we are sending ALL reviews to a model (can be 10, 100, 10000) then sending the groups again to another model. What if the grouping model sends 20 groupings to the main model as a batch?

Solution Approaches:
We could run a small embedding model like text-embedding-3-small to embed all the modification review texts then cluster these embeddings with cosine similarity. The challenge here would be then picking the review in each cluster to be the representative to attribute credit to the author (we could just choose the highest rated review). From here it is similar to the second naive approach where we send each selected modification as one batched llm call. This shouldnt have context bloat as the clustering eliminates a lot of duplicated reviews
This approach adds deterministic embeddings, and clustering

Alternatively, I noticed that scraper_v2.py has has modification-detection regex patterns   Line 80: tweak_patterns = [  r"I (added|used|substituted|replaced|made with|changed)",
What if we can use these same regex patterns as extractors? We would then normalize the matching sentences (lowercase, strip quantities to placeholders, stem ingredients) and group by hash fingerprint. For each unique fingerprint group, pick the best review and batch all of them into a single LLM call for structured extraction.
This removes the embedding dependency.
Regex is brittle, btu we are already using it for detection anyways.
Normalization tends to be lossy.

## Claude

Documenting.

**The embedding + clustering approach is the better option.** Here's my reasoning against your three questions:

### 1. Are we certain the system parses out ALL intended modifications?

**Embeddings: yes.** Every review with `has_modification=True` gets embedded. Semantic similarity catches modifications regardless of how they're phrased — "my grandmother's trick is to fold in cream cheese" clusters alongside "I added cream cheese." Nothing is filtered by syntactic form.

**Regex: no.** Those 5 patterns were built for *detection* (binary signal: "does this review mention a tweak?"), not *extraction*. Detection tolerates false negatives; extraction cannot. A review saying "the secret is browning the butter first" has no match against `r"I (added|used|substituted|...)"`. You'd silently lose modifications expressed outside the pattern vocabulary.

### 2. Does the system scale beyond the 5 examples?

**Embeddings scale in coverage.** `text-embedding-3-small` costs ~$0.02/1M tokens and handles arbitrary language. Clustering is fast. The real bottleneck — the final LLM call — gets controlled because clustering collapses duplicates. 200 reviews saying "I added vanilla" become 1 cluster.

**Regex scales computationally but not linguistically.** Different cuisines, review styles, or non-English loanwords will produce modifications the patterns miss. You'd accumulate regex maintenance debt as the corpus grows.

### 3. Are there poor assumptions?

**Regex carries two dangerous ones:**
- That detection patterns = extraction patterns (they serve different purposes)
- That normalization (stemming, quantity placeholders) won't merge genuinely distinct modifications ("added cinnamon" vs "added cardamom" could collide after aggressive stemming)

**Embeddings assume** cosine similarity cleanly separates distinct modifications. This is well-supported for short, concrete cooking statements. The "pick highest-rated review as representative" heuristic is straightforward and auditable.

### Bottom line

The regex approach inherits the scraper's detection vocabulary and repurposes it for extraction — a category error. Embeddings treat every modification review as semantic content, cluster by meaning, and deduplicate before hitting the LLM. That directly solves the "all modifications" question without context bloat and without brittle pattern lists.

```tool
[Tool: Read] file_path: /Users/jperr/Documents/Casper-Assessment/src/llm_pipeline/pipeline.py
```

```tool
[Tool: Agent] subagent_type: Explore, prompt: Find and read scraper_v2.py tweak_patterns and tweak_extractor.py extract_single_modification...
```

## User

can you modify my solution approach for the cosine similarity to be formalised as a proper solution approach claim. keep it as one paragraph

## Claude

Here's the formalized claim:

**Solution Approach: Embedding-Based Clustering with Batched Extraction.** All reviews where `has_modification=True` are embedded using `text-embedding-3-small`, then clustered by cosine similarity to collapse semantically duplicate modifications into distinct groups. Within each cluster, the highest-rated review is selected as the representative, preserving attribution. The representative review texts are then batched into a single LLM call to `gpt-3.5-turbo` for structured modification extraction, where each representative produces one `ModificationObject`. This ensures every unique modification surfaces (no random sampling), scales sublinearly with review count (clustering absorbs duplicates before the LLM sees anything), and confines the expensive LLM call to a bounded set of deduplicated representatives rather than raw review volume.

## User

I am done documenting.

