# Search Ranking Quality Report

**Slick Statistics** is the public record of how [Slick](https://slicksearchhq.com) search ranking behaves over time. This repository holds frozen SERP benchmarks and the analysis that goes with them. **Benchmark files and this report are refreshed whenever a major ranking release ships**, so each file is a dated snapshot you can compare against earlier ones.

[Slick](https://slicksearchhq.com) is a private search engine: it does not track users or rely on personal data. That design choice removes the signals most commercial engines use for localization and personalization, so **geo and “near me” style queries stay weak by default**. The production corpus is relatively modest (**on the order of two million documents**). Even at that scale, ranking aims to stay competitive on head queries and usable on many long tails.

The sections below focus on the **latest** snapshot. Comparisons with **launch** and **intermediate** appear where they clarify what changed. Future releases may add more snapshots; this report will not always compare every file on disk.

---

## How ranking evolved (product snapshot)

**Launch** used a single-stage score. Straightforward informational cases were acceptable, but navigational queries and many long tails suffered. Brand homepages often lost to off-target URLs on the same registrable domain, and some how-to prompts pulled loosely related articles.

**Intermediate** layered targeted heuristics and intent rules. Major-brand navigation improved; definition-style, who-is, and ecosystem-page handling gained structure. Accuracy rose for several well-known domains, yet encyclopedia-style mismatches on definitional prompts and brittle token overlap remained visible.

**Latest** combines **dual recall** (lexical plus dense retrieval) **with fusion**, then **cross-encoder reranking**, then a **normalized heuristic blend** feeding **`finalScore`**. Dense similarity no longer feeds the main heuristic sum directly; semantics show up chiefly through recall and reranking. That shift produced clearer gains on both head and many long-tail prompts described below.

---

## Benchmark files

Each text file is a complete run: every query in that snapshot, with ranked results and scores. Query strings appear in the `=== QUERY: … ===` headers (the **75-query suite** in `ranking_benchmark_latest.txt` is the current reference set).

| File | Era | Queries | Top ranks | Score fields |
|------|-----|---------|-----------|--------------|
| `ranking_benchmark_launch.txt` | Launch | 15 | ≤10 | `unifiedScore` |
| `ranking_benchmark_intermediate.txt` | Intermediate | 91 | ≤100 (see header) | `finalScore`, reranker / heuristic channels |
| `ranking_benchmark_latest.txt` | **Latest** | **75** | **≤10** | `finalScore`, reranker / heuristic channels |

**Adding snapshots over time.** A major ranking update typically adds or replaces `ranking_benchmark_latest.txt` and updates this report. Older files stay in the repo for history. The next release might compare only **intermediate → latest**, only **latest → a new snapshot**, or several pairs at once, depending on what changed. Not every report will walk through all three eras.

---

## Latest benchmark at a glance

The **latest** run uses **75** queries balanced across navigational, informational, transactional, commercial, news-oriented, local, and long-tail buckets, **≤10** results each, **zero** empty SERPs.

**Headline metrics**

| Observation | Value |
|-------------|------:|
| Mean position-1 `finalScore` | **0.90** |
| Position-1 navigational hits on registrable official hosts (21 nav queries) | **10 / 21 (48%)** |
| Queries with cross-encoder score exactly **0** at position 1 | **0** |
| Position-1 URL moves vs **intermediate** on the **20** overlapping queries | **16 / 20** |

Strong navigational winners often sit at the top of the displayed score range with equally strong passage-level agreement in the reranker channel. Weak navigational cases (`facebook`, `twitter`) stay far below that band (position-1 blends roughly **0.22** and **0.35**), which drags the navigational intent average down even when other brands look excellent at rank 1.

**Intent mix (latest suite)**

| Bucket | Count |
|--------|------:|
| Navigational | 21 |
| Informational | 20 |
| Transactional | 19 |
| Question-shaped | 13 |
| Local | 2 |

---

## Representative outcomes (aligned with live ranking)

These rows summarize behavior users see today versus earlier frozen captures.

### Navigational

| Query | Result |
|-------|--------|
| `apple` | Launch-era captures surfaced product or site-search URLs; **latest** consistently places the main homepage at rank 1. |
| `stackoverflow` | Intermediate ranked dataset mirrors above the official property; **latest** ranks the official site first. |
| `facebook` | Still prefers a locale host (`en-gb.facebook.com`) over apex; better than encyclopedic wrong-article failures in intermediate, but canonical apex remains unresolved. |
| `amazon` | Improved from influencer shop paths to the main storefront root. |

### Informational and question-shaped

| Query | Result |
|-------|--------|
| `what is artificial intelligence` | Previously surfaced a researcher biography; now favors a direct definitional article. |
| `what is machine learning` | Same pattern: away from mismatched encyclopedia entries toward explanation-focused pages. |
| `how does the internet work` | Better than clearly wrong articles in older captures, but can still favor IoT-oriented guides over core network-stack explainers. |

### Transactional

| Query | Result |
|-------|--------|
| `download vscode` | Returns the correct download URL; blended **`finalScore`** at rank 1 stays lower than expected despite destination correctness. |
| `best smartphones 2025` | Review roundups rank materially higher than in launch-era runs where most candidates carried negligible score mass. |

---

## Why latest wins overall

Hybrid retrieval plus reranking trims several recurring failure families:

- **Navigational reliability** for major brands (Amazon, Microsoft, LinkedIn, Stack Overflow, GitHub, and similar) versus shop paths, mirrors, or tertiary subsites.
- **Definitional queries** see fewer biography or category-page mismatches when users wanted plain explanations.
- **Long-tail recall** benefits from dense channels fused with lexical recall before reranking.
- **Empty SERPs** do not appear in the latest suite (contrast with brittle launch-era slices).
- **Cross-encoder scoring** tightens top-result consistency across intents; rank-1 never drops to zero reranker signal in this benchmark.

Top results feel **more stable** across query types, especially where official homes surface consistently, **without** assuming a web-scale index.

---

## How position-1 scores behave (latest suite)

**Saturation at the top.** Multiple URLs can pin reranker scores near the ceiling while titles diverge in scope. Broad technical questions illustrate this: several “internet”-adjacent guides all look locally strong even when only some match the user’s implied framing (core networking versus adjacent consumer-tech narratives).

**Thin tail at the low end.** Only **one** query lands position 1 with `finalScore` strictly below **0.15**: `download vscode`, correct URL but weak blend. `facebook` and `twitter` show **weak** blends yet stay above that floor, signalling host ambiguity rather than wrong entity.

**Observable channels.** Intermediate and latest files list `finalScore`, `crossEncoderScore`, and `heuristicScore` per hit, along with titles, URLs, and snippets. Position 1 keeps positive `crossEncoderScore` everywhere in the latest run, unlike launch-era captures where lexical spikes alone decided winners.

---

## SERP-level notes (latest suite)

### Brand and property navigation

On queries shared with intermediate, official roots often reclaimed rank 1 after shops, dataset mirrors, login surfaces, or tertiary paths had won: Amazon apex versus influencer paths, Microsoft corporate entry versus encyclopedic learn pages, LinkedIn root versus login, Stack Overflow official versus off-site mirrors.

### Definition and explainer queries

Lexical-heavy biography inflation on “what is …” prompts is largely gone in favor of definitional journalism and structured explainers. Mechanism queries (`how does the internet work`) can still drift to IoT explainers when reranker saturation hides topical mismatch.

### Transactional and commercial

Aside from the VS Code blend anomaly, transactional prompts usually land credible destinations. Comparison prompts (`react vs vue`, cloud provider triplets) remain recall-bound: rank 1 reflects whichever comparison-style pages entered the candidate pool.

### Local

Only **two** queries carry a local tag in this classification snapshot. Expectations stay low without geo or user signals: `gyms in san francisco` may reward geographic overlap through a national retailer locator rather than gym verticals.

### Long-tail hobbies

Project-oriented URLs with instructional headings score well (for example woodworking prompts), unlike launch captures where commerce-adjacent token overlap zeroed most of the SERP.

---

## Position-1 domain concentration

In the latest file, repeat apex domains at rank 1 skew toward encyclopedic and trade publishers (Britannica, TechTarget, Wired, Guardian, Business Insider, Investopedia, YouTube, and similar). That reflects **query mix** (explainers and news-like intents), not a claim about global production traffic.

---

## Intermediate vs latest (overlap)

Twenty queries appear in both files; **sixteen** change position-1 URL. This section is one pairwise comparison; future reports may highlight different pairs or skip overlap tables entirely.

**Clear upward moves (illustrative)**

| Query | Intermediate position 1 | Latest position 1 |
|-------|------------------------|-------------------|
| `amazon` | Influencer shop path | Brand apex |
| `stackoverflow` | Off-site dataset mirror | Developer Q&A home |
| `microsoft` | Encyclopedia learn article | Corporate root / locale entry |
| `linkedin` | Login surface | Root property |
| `what is artificial intelligence` | Wrong encyclopedia biography | Consumer definitional article |
| `what is machine learning` | Off-topic encyclopedia article | Trade publication definition |
| `what is python programming` | Wikibooks chapter | Structured definition article |
| `history of world war 2` | Category-style encyclopedia page | Curated historical overview |
| `climate change explained` | Adjacent encyclopedia topic | Explainer lecture content |

**Mixed outcomes**

| Query | Assessment |
|-------|------------|
| `facebook` | Drops encyclopedic noise; locale apex vs registrable apex still debated |
| `twitter` | Drops encyclopedic noise; embed-focused subdomain may still disappoint |
| `how does the internet work` | Removes worst mismatches; IoT skew may persist at rank 1 |
| `who is elon musk` | Credible news analysis; neutral biography pages remain uncommon |
| `quantum mechanics explained` | Still vulnerable when recall feeds scientist biographies |

**Stable across captures:** `apple`, `google`, `youtube`, `reddit` stayed on strong official or canonical URLs.

---

## Earlier snapshots (brief)

**Launch (15 queries):** single-stage blend; junk paths on brands, product URLs beating homes, “best of” prompts with almost no scored mass.

**Intermediate (91 queries):** wider query list than latest; same score columns as latest, with more results per query in file headers. Useful when comparing URL-level changes into the current 75-query suite.

---

## Excerpted signals from `ranking_benchmark_latest.txt`

Scores rounded for readability.

**Navigational, strong**

```
finalScore: 1
crossEncoderScore: 1
heuristicScore: 1
URL: https://apple.com/
```

**Navigational, mixed host**

```
finalScore: 0.22
crossEncoderScore: 1.0
heuristicScore: 0.98
URL: https://en-gb.facebook.com/
```

**Navigational, corrected vs intermediate**

```
finalScore: 1
crossEncoderScore: 1
URL: https://stackoverflow.com/
```

**Question, topical drift with saturated reranker**

```
#1 finalScore: 1.0   crossEncoderScore: 1.0
   Title: What Is the Internet of Things? A WIRED Guide
#5 finalScore: 0.99  crossEncoderScore: 0.99
   Title: What is IoT? A Beginner's Guide
```

**Commercial roundup**

```
#1 finalScore: 1.0   crossEncoderScore: 1.0
   URL: techradar.com/.../best-phone...
```

**Comparison query under recall stress**

```
#1 finalScore: 0.77   crossEncoderScore: 1.0
   URL: css-tricks.com/author/...
```

**Local geo without vertical precision**

```
#1 finalScore: 0.95   crossEncoderScore: 0.97
   URL: rei.com/stores/san-francisco.html
```

**Transactional, correct destination / weak blend**

```
#1 finalScore: 0.10
   URL: https://code.visualstudio.com/download
```

**Long-tail hobby**

```
#1 finalScore: 0.91
   Title: ... beginner-friendly outdoor woodworking projects ...
```

---

## Output formats (benchmark dumps)

**Launch** (`unifiedScore` era)

```
=== QUERY: {query} ===
Score (unifiedScore): {float}
Title: ...
URL: ...
```

**Intermediate and latest**

```
=== QUERY: {query} ===
Total results: N
Page: p / P
Page Size: K
Classification: {intent label}
============================================================

--- RESULT i ---
finalScore: {0–1 scale}
crossEncoderScore: {0–1 scale}
heuristicScore: {0–1 scale}
Title: ...
URL: ...
Snippet: ...
```

Sublinks may appear for some results when present in the index.

---

## Remaining gaps

Issues line up with both ranking mechanics and product constraints:

1. **Canonical hosts:** locale or tooling subdomains can beat apex URLs (`facebook`, `twitter`).
2. **Explainer drift:** umbrella phrases (“how does the internet work”) invite adjacent topics when every candidate scores highly.
3. **Who-is:** standard biography destinations rarely win; news analysis often tops instead.
4. **Local:** weak without geo or behavioral signals (expected for a privacy-first engine).
5. **Comparisons:** dedicated head-to-head articles are sparse in recall (`react vs vue`, cloud triplets).
6. **Transactional blends:** correct destinations sometimes receive lower blended scores than human raters expect (`download vscode`).
7. **Suite alignment:** intermediate lists **91** queries versus **75** in latest; sixteen intermediate-only prompts are outside the latest matrix until suites converge.

---

## Appendix: query presence matrix (selected)

| Query | Launch | Intermediate | Latest |
|-------|:------:|:------------:|:------:|
| facebook | ✓ | ✓ | ✓ |
| google | ✓ | ✓ | ✓ |
| apple | ✓ | ✓ | ✓ |
| how does the internet work | | ✓ | ✓ |
| best smartphones 2025 | ✓ | | ✓ |
| react vs vue | | | ✓ |
| gyms in san francisco | | | ✓ |
| what is artificial intelligence | | ✓ | ✓ |
| stackoverflow | | ✓ | ✓ |
| amazon | | ✓ | ✓ |

---

*Latest benchmark: May 2026 (75 queries, up to 10 results per query).*
