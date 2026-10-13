---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Relevance"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems"
tools: ["Elasticsearch"]
---

# Search Relevance

**L5 Expert** · [↑ Search Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Relevance is a ranking pipeline — analyse text, retrieve candidates, score them (BM25 + business signals + optionally vectors), re-rank the top few, and then *measure* whether the changes helped with offline judgments and online experiments.

### 1. The big picture
Retrieval answers "which documents match?"; relevance answers "in what order should the user see them?". A modern search stack has layers: lexical retrieval with BM25, structured boosts (freshness, popularity, stock), semantic retrieval with embeddings, and a re-ranker (learning-to-rank or a cross-encoder). Each layer trades quality for cost, which is why cheap recall-oriented stages come first and expensive precision-oriented stages only see the top 50–200 results.

The hard part is not the query DSL — it is making ranking changes safely: every tweak helps some queries and hurts others, so you need judgment sets, metrics and controlled experiments.

### 2. Core concepts and vocabulary
- **TF / IDF** — term frequency in the document; inverse document frequency across the corpus (rare terms count more).
- **BM25** — the default scoring function; saturating TF and length normalisation with parameters `k1` and `b`.
- **Boost** — a multiplier on a clause or field (`title^3`).
- **`function_score`** — wraps a query and modifies the score using functions (field value factor, decay, script, weight, random).
- **Rescore** — run a more expensive query only over the top `window_size` results from the first pass.
- **Analyzer / synonyms / stemming** — normalisation that decides which words match.
- **Dense vector / kNN** — embedding vectors and approximate nearest-neighbour search (HNSW).
- **Hybrid search** — combining lexical and vector retrieval.
- **RRF (reciprocal rank fusion)** — merges ranked lists by rank, not score.
- **Judgment list** — queries with graded relevance labels.
- **NDCG, MRR, precision@k, recall@k** — ranking metrics.
- **Interleaving / A-B test** — online comparison of ranking variants.

### 3. How it works, step by step
1. **Normalise the query** with the same analyzer chain used at index time (lowercasing, tokenisation, stemming, synonyms, spell correction).
2. **Retrieve candidates**: a `bool` query with `must` (scoring) and `filter` (non-scoring, cacheable) clauses; optionally a kNN query for semantic recall.
3. **Score** each candidate with BM25 per field; combine fields with `multi_match` types (`best_fields`, `cross_fields`, `most_fields`) or `dis_max`.
4. **Apply business signals** with `function_score`: boost fresh, popular or in-stock items; demote out-of-stock; promote sponsored items in a *controlled* way.
5. **Rescore** the top N with a heavier model (phrase proximity, LTR, cross-encoder).
6. **Return results with explanations on demand** (`explain: true`) for debugging.
7. **Evaluate** on a judgment set (offline), then on traffic (online) before shipping.

BM25 in a line: `score(q,d) = Σ_t IDF(t) · tf·(k1+1) / (tf + k1·(1−b+b·len(d)/avglen))`, defaults `k1 = 1.2`, `b = 0.75`.

### 4. Worked example
```json
GET products/_search
{
  "query": {
    "function_score": {
      "query": { "bool": {
        "must":   [ { "multi_match": { "query": "running shoes",
                        "fields": ["title^3","brand^2","description"], "type": "best_fields" } } ],
        "filter": [ { "term": { "in_stock": true } } ] } },
      "functions": [
        { "field_value_factor": { "field": "popularity", "modifier": "log1p", "missing": 0 } },
        { "gauss": { "released": { "origin": "now", "scale": "90d", "decay": 0.5 } } }
      ],
      "score_mode": "multiply",
      "boost_mode": "multiply"
    }
  },
  "rescore": { "window_size": 100,
    "query": { "rescore_query": { "match_phrase": { "title": { "query": "running shoes", "slop": 2 } } },
               "query_weight": 1.0, "rescore_query_weight": 1.5 } },
  "explain": true
}
```
Reading it: the filter removes out-of-stock items without affecting score; `log1p(popularity)` prevents a viral item from dominating; the Gaussian decay halves the weight of items 90 days old; the rescore rewards exact phrase proximity for only the top 100. In the `explain` output look at each clause's contribution to see which signal is dominating.

Hybrid with RRF (conceptually): run BM25 → ranks `r_b(d)`, run kNN → ranks `r_v(d)`, then `RRF(d) = Σ 1/(k + r(d))` with `k = 60`.

### 5. Under the hood
- **Why not just add BM25 and vector scores?** The scales differ (BM25 is unbounded and corpus-dependent, cosine similarity is 0–1) and shift per query; any fixed weight is brittle. RRF only uses ranks, so it's scale-free and robust; alternatives are min-max normalisation, learned weights or a re-ranker.
- **kNN with HNSW** builds a layered graph per segment; recall/latency are tuned by `m`, `ef_construction`, `num_candidates`. Filtering with kNN can be pre-filter (accurate, slower) or post-filter (fast, may return too few).
- **Shard-local statistics.** IDF is computed per shard; on small indexes it can skew scores. `dfs_query_then_fetch` computes global statistics for an extra round trip.
- **`b` mis-set symptom**: if long documents always win → `b` too low; if short stubs outrank substantive documents → `b` too high. Titles (short, uniform) usually want low `b`; long documents want the default or higher.
- **Learning to rank** uses features (BM25 per field, popularity, recency, click models) and a gradient-boosted model, applied in the rescore stage. It needs clean labels or click data with position-bias correction.
- **Embeddings lifecycle**: changing the model changes every vector — you must re-embed and reindex, keep a version in the field name or index, and be able to run old and new side by side.
- **Evaluation metrics**: precision@k (how many of the top k are relevant), recall@k, MRR (position of first relevant), NDCG@k (graded relevance with log position discount, normalised against the ideal ordering).

### 6. Performance and design implications
- Put restrictive, cacheable conditions in `filter`; they don't score and are cached as bitsets.
- Keep rescoring windows small (50–200).
- Prefer a few well-understood signals with bounded modifiers (`log1p`, `sqrt`, `saturation`) over uncapped multipliers.
- Keep business rules explicit and auditable: promoted items in a separate stage, with labels for sponsored results.
- Tune with data: collect query logs, label a sample (including long-tail and zero-result queries), watch zero-result rate, click-through rate on top 3, reformulation rate, and conversion.
- Experiment platforms need unit-of-randomisation rules, collision control for overlapping tests, and sample-size/power planning; interleaving needs fewer users than A/B for ranking comparisons.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `explain: true` / `_explain` API | Score breakdown per clause | Debug a ranking surprise |
| `function_score` | Add popularity, recency, rules | Business ranking |
| `rescore` | Expensive second pass on top N | Phrase, LTR, cross-encoder |
| `multi_match` types | Field-combination strategy | Multi-field queries |
| `similarity` settings (`k1`, `b`) | BM25 tuning | Corpus with unusual length distribution |
| `search_type=dfs_query_then_fetch` | Global IDF | Small/skewed shards |
| `knn` query / `rank` RRF | Vector and hybrid retrieval | Semantic search |
| Ranking evaluation API (`_rank_eval`) | Offline metrics on judgments | Regression test relevance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| BM25 only | Queries are keyword-ish, vocabulary is controlled |
| Hybrid lexical + vectors | Natural-language queries, vocabulary mismatch |
| Self-hosted embedding model | Data cannot leave the boundary; you can run GPUs |
| Managed embedding API | Small team, variable volume, willing to accept vendor/versioning risk |
| LTR | You have enough labelled/click data and an offline pipeline |
| Hard business rules in the query | Rules change often → configuration, not code |

### 9. Common mistakes
- Tuning on a handful of queries → regressions elsewhere → judgment set + `_rank_eval`.
- Uncapped popularity boost → a few items always on top → log/saturating modifiers.
- Adding raw BM25 and cosine scores → unstable ranking → RRF or learned fusion.
- Forgetting to re-embed after a model change → mixed vector spaces → versioned vector fields and reindex.
- Treating click-through as truth → position bias → interleaving, debiasing, or randomised exploration.
- Pushing relevance config globally at once → global regression → staged rollout and experiments.

### 10. 🎯 Interview kit
**30-second answer:** Relevance is layered: analyse and match, score with BM25, add bounded business signals with `function_score`, rescore the top N with something heavier, and evaluate with NDCG on a judgment set and online with A/B or interleaving. For hybrid search I fuse lexical and vector rankings with reciprocal rank fusion because scores aren't comparable.

**Structure a longer answer:** retrieval vs ranking → BM25 and parameters → business signals → semantic/hybrid → evaluation → operational safety.

**Likely follow-ups:**
- *Why RRF?* → rank-based, scale-free, robust, needs no tuning (k≈60).
- *How to layer sponsored/out-of-stock?* → filter or demote with function_score; promoted results injected in a separate labelled slot.
- *What is `b`?* → document-length normalisation.
- *How to run 30 teams' experiments safely?* → traffic layers, namespaces, guard metrics, power analysis.

**Signals interviewers look for:** metrics-driven mindset, awareness of score scales, data-pipeline thinking for embeddings, experimentation rigor.

### 11. Check yourself
- [ ] Write the BM25 formula and describe `k1` and `b`.
- [ ] Build a `function_score` query with a bounded popularity signal and freshness decay.
- [ ] Compute RRF for two ranked lists by hand.
- [ ] Explain NDCG@10 in words.
- [ ] Plan a re-embedding migration with zero downtime.
<!-- explainer:end -->

## Implementations (tools)

- [Elasticsearch](<../../../../TOOLS/Elasticsearch/_index.md>)

## Practice questions

- [Deep] [A search returns relevant docs but ranked terribly. Walk me through how you'd debug scoring — explain output, boosting, function_score, and constant_score — to fix relevance without breaking recall.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24467_A_search_returns_relevant_docs_but_ranked_terribly_Walk_me_t.md>)
- [Deep] [Explain how you'd A/B test two relevance configs safely in production. How do you split traffic, log per-query features, and detect a regression on a specific query segment before full rollout?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24533_Explain_how_you_d_A_B_test_two_relevance_configs_safely_in_p.md>)
- [Expert] [Design a search-relevance evaluation harness. How do you measure NDCG/MRR offline, run A/B tests online, and prevent a relevance tuning change from silently regressing tail queries?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24476_Design_a_search-relevance_evaluation_harness_How_do_you_meas.md>)
- [Expert] [Design the reference architecture for a search platform that must serve lexical, semantic, and hybrid retrieval as first-class products to any team. What abstractions do you expose so teams don't need to understand HNSW or BM25 internals?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24362_Design_the_reference_architecture_for_a_search_platform_that.md>)
- [Expert] [Two teams both want to own 'search relevance' — the platform team and a product ML team. As principal, resolve this org-design conflict. Where do the responsibility boundaries and interfaces sit, and how do you prevent duplicated ranking stacks?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24376_Two_teams_both_want_to_own_search_relevance_the_platform_tea.md>)
- [Expert] [Relevance quality varies wildly across your 30 tenant search apps and there's no shared definition of 'good.' As principal, how do you establish org-wide relevance standards and a shared judgment infrastructure without stifling domain-specific tuning?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24394_Relevance_quality_varies_wildly_across_your_30_tenant_search.md>)
- [Expert] [You're standardizing how the org does zero-downtime reindexing as a platform capability, since every team hand-rolls it dangerously. Design the reindex-as-a-service: alias orchestration, dual-write windows, validation gates, and automatic rollback. What are the correctness invariants you enforce?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24410_You_re_standardizing_how_the_org_does_zero-downtime_reindexi.md>)
- [Expert] [You must decide the ownership boundary between your search platform and the ML infra platform for feature stores that feed learning-to-rank. Where does data ownership, freshness SLA, and model-serving responsibility sit, and how do you avoid a fragile cross-team dependency?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Q-24419_You_must_decide_the_ownership_boundary_between_your_search_p.md>)

## Related concepts

- [Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Re-ranking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>) *(Retrieval and RAG)*
- [Quality Attribute Trade-off Scenarios](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>) *(Trade off Analysis)*
- [Index Strategy and Governance at Scale](<../../../Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>) *(Data Platform Strategy)*
- [Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*

## References

- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Q
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Consequences
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Democratizing Data Quality › Prioritize Long-Term Growth and Sustainability Versus Short-Term Gains
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Democratizing Data Quality › Case Study: Toast's Journey to Finding the Right Structure for Their Data Team
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Democratizing Data Quality › Prioritize building a diverse data team from day one
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — T
- *Designing Data-Intensive Applications* — Martin Kleppmann — M
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › What Will We Want?
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › Final Thoughts on Scoring
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Conclusion
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Creating Foresight › Hypothesize
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Creating Foresight › Conclusion
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — People in the Loop › Conclusion

