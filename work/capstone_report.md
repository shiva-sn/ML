# Capstone Report — Structured Content Archetype Clustering

- **Author:** Shiva
- **Lane:** Structured Content Archetype Clustering
- **Repo:** https://github.com/shiva-sn/ML
- **Date:** 2026-09-03

## 0. Abstract

This project asks what recurring performance archetypes exist across a large content inventory and how those patterns can support content-review decisions. The analysis uses an anonymized, content-level dataset with a rolling 90-day performance window and eight observed numeric features covering search demand, content size, freshness, visibility, and engagement. An unsupervised K-Means analysis with K=3 was built after treating zero search position as missing, median-imputing missing values, log-transforming skewed volume fields, and RobustScaling the feature matrix. The final full-population reference result has a silhouette score of 0.8414, with three interpretable observed archetypes: Low-Visibility Established Content, Stale Underperforming Content, and High-CTR Efficient Niche Content. The output is a decision-support and prioritization aid for human review, not a causal model and not a forecast of future search performance.

## 1. Problem framing

The decision is **which types of content should a content/SEO team review first, and what kind of review may be appropriate**.

The unit of analysis is **one row per content item** at the analysis snapshot. The primary output is an **archetype/cluster assignment**, followed by a ranked review queue in W07. A human editor uses that output to prioritize inspection and decide whether to improve, rewrite, protect, monitor, or route an item to manual review.

The cost of a wrong call is practical: unnecessary edits can consume editorial resources or damage useful content, while missed review opportunities can leave stale or low-visibility content unresolved. ML helps because the inventory contains multiple interacting observed signals that are difficult to reduce to one fixed threshold without losing structure.

## 2. Data safety

The analysis uses an anonymized structured content dataset and content-level aggregates used by W05. The core feature vector is:

`search_volume`, `word_count`, `content_age_days`, `days_since_update`, `impressions_90d`, `ctr_90d`, `avg_position_90d`, `engagement_rate`.

Deliberately excluded from clustering:

- `client_id` / `client_hash_id` and `content_id` / `content_hash_id` — identifiers for grouping or joining only, never model features.
- Query-breadth fields — excluded because they can encode data-coverage differences rather than substantive content archetypes.
- `trend_direction` / `trend_pct` and other future/trend/label-like fields — excluded to avoid using later outcomes in the snapshot-defined clustering vector.
- Provider/model metadata and client names, domains, and URLs — unnecessary for archetype definition and inappropriate for public-facing outputs.

`avg_position_90d = 0` is interpreted as **no position data**, then treated as missing before imputation.

The project keeps client-identifying information out of paper-facing outputs and uses only aggregate or hashed/public-safe identifiers where necessary.

## 3. Baseline

For this clustering lane, the meaningful baseline is not a supervised accuracy rule. The baseline is the **simpler volume/size feature representation** considered before the final eight-feature clustering design, together with a transparent threshold-based review mindset.

The final analysis does not claim that K=3 is superior to a hidden ground truth. Instead, it compares candidate cluster solutions using internal separation and cluster balance, while explicitly excluding leakage-prone fields.

The headline quantitative result is the final W05 full-population silhouette reference of **0.8414** for K=3. Because this is an internal clustering metric, it should not be read as accuracy, lift, or business impact.

## 4. Model / analysis

The method is **unsupervised K-Means clustering with K=3**. This fits the lane because there is no ground-truth archetype label to predict; the goal is to discover groups of content items with similar observed profiles.

Preprocessing:

1. Convert `avg_position_90d = 0` to missing.
2. Median-impute numeric missing values.
3. Apply `log1p` to `search_volume`, `word_count`, and `impressions_90d` to reduce skew.
4. Apply `RobustScaler` before K-Means.

The target/proxy definition is: **there is no prediction target; cluster assignment is the unsupervised output**.

The exact eight-feature vector is the same vector audited in W03 and used downstream by W05. IDs, query breadth, future/trend fields, and other metadata are intentionally outside the clustering matrix.

## 5. Evaluation

The primary model-selection and reporting reference is the W05 full-population clustering result with **K=3** and **silhouette = 0.8414**.

The validation framing in W06 is deliberately cautious about generalization. Where client-grouped validation is used, preprocessing is fitted on development clients and held-out clients are assigned using development-fitted centroids. This tests transfer to unseen client groups; it is **not** a time-forward test and does not establish future performance.

Error analysis for clustering is qualitative rather than classification-style: ambiguous assignments, material missingness, and rare clusters are routed to human review. The rare high-efficiency cluster is particularly important to verify because a very small group can be unstable or overly narrow as a portfolio rule.

There is no supervised accuracy, precision@K, or AUC claim in this capstone because the final lane is unsupervised. The earlier Week 1 refresh-ranking exercise is a separate supervised workflow and should not be used as the performance metric for this clustering capstone.

## 6. Interpretation

The final clustering yields three observed archetypes:

### Cluster 0 — Low-Visibility Established Content

These items are interpreted as established content with relatively weak observed search visibility. The supported action is **Improve**: review the page for targeted optimization opportunities rather than assuming the page should be replaced.

### Cluster 1 — Stale Underperforming Content

These items show the combination of weaker observed performance and freshness/staleness signals. The supported action is **Rewrite**: inspect whether substantial content or freshness revision is appropriate.

### Cluster 2 — High-CTR Efficient Niche Content

These items show comparatively efficient observed click performance and are a very small cluster. The supported action is **Protect**, with an explicit manual-review gate because the pattern is rare.

The most important surprise is not a single feature importance; it is that useful archetypes emerge from a **combination** of content, freshness, visibility, and engagement signals rather than a one-variable rule. The earlier Week 1 exploration also showed that simple search-volume intuition is weak in the supplied sample: the observed correlation between `search_volume` and `impressions_90d` was approximately **0.001**. This supports treating search demand as one signal among several, not as a stand-alone proxy for page traffic.

The analysis is metric-based, not semantic: article text was not used to define the clusters.

## 7. Recommendation

The W07 playbook converts the archetypes into a human-review workflow:

| Archetype | Action | Practical use |
|---|---|---|
| Stale Underperforming Content | **Rewrite** | Review first for substantive freshness/content changes. |
| Low-Visibility Established Content | **Improve** | Inspect for targeted optimization opportunities. |
| High-CTR Efficient Niche Content | **Protect** | Avoid unnecessary edits; manually verify the rare pattern. |
| Ambiguous / materially incomplete cases | **Review** | Diagnose the underlying data/content before acting. |

A FlyRank editor could use the queue tomorrow by starting with the highest-priority rows, checking the underlying page context, and then accepting, changing, or rejecting the suggested action. The model does not publish, rewrite, delete, or redirect content automatically.

Confidence is strongest at the level of **observed structural patterns** and weaker for any claim about future business outcomes. No recommendation should be interpreted as proof that the chosen action will improve rankings, traffic, or engagement.

## 8. Reproducibility

The notebooks are designed to run from the repository with a relative `work/` layout. The core sequence is:

```bash
git clone https://github.com/shiva-sn/ML.git
cd ML
pip install -r requirements.txt
```

Then run the analysis notebooks in project order, with W03 defining the data/feature boundaries, W05 producing the clustering output, W06 auditing validation and model claims, W07 producing the action queue, and `capstone.ipynb` consolidating the paper-facing record.

The central feature-preparation choices are deterministic in definition: median imputation, `log1p` on the three skewed volume/count fields, and RobustScaler. K-Means is reported at **K=3**. The repository contains the notebooks and output conventions needed to regenerate the analysis rather than relying on an undocumented manual step.

For the clustering capstone, the most important reproducibility artifacts are the W03 feature/leakage definitions, the W05 clustered content output, the W07 action queue/summary, and this report. A fresh rerun should be used to confirm that reported counts and summary statistics still match the current repository outputs.

## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset**.

Data source / internship warehouse credit: **FlyRank** — https://flyrank.ai

The project reports only anonymized, hashed, aggregate, or otherwise public-safe analysis outputs.

---

### Claims checklist

This report uses **observed, measured, directional, and decision-support** language. It does not claim causal effects, does not claim to predict Google's algorithm, and does not use client-identifying details. The silhouette value is reported as an internal clustering metric rather than accuracy or business impact.
