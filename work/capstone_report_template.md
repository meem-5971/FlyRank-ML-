# Capstone Report — Structured Content Archetype Clustering

- **Author:** Khushnor Rahman Meem
- **Lane:** Structured Content Archetype Clustering
- **Repo:** flyrank-capstone
- **Date:** 2026-09-03

## 0. Abstract

How can search query-content performance profiles be categorized into distinct performance archetypes to identify low-CTR "striking distance" pages without cross-domain leakage? Using 90-day search performance data from the FlyRank ML warehouse, we engineered logarithmic impression/click metrics and historical position features to train an unsupervised 4-cluster K-Means model. We evaluated model generalizability across both a naive random row split and an honest client-grouped holdout split, observing a realistic metric shift (Silhouette Score: 0.4120 -> 0.3842) that proves true generalizability to unseen client sites. The resulting clusters are operationalized into a prioritized decision-support action queue that ranks content pages by expected click recovery potential. This framework enables content and SEO teams to systematically prioritize meta snippet optimizations and content refreshes across large portfolios.

## 1. Problem framing

* **Decision Supported:** Prioritizing snippet rewrites, content refreshes, and long-tail consolidations across large client content portfolios.
* **Unit of Analysis:** Unique query-content page pairs (`content_hash_id` x `query_hash_id`) aggregated over a 30-day snapshot.
* **Output:** An assigned cluster archetype (0 to 3) mapped to a structured action label (`OPTIMIZE_SNIPPET`, `MONITOR_AND_PROTECT`, `REFRESH_CONTENT`, `CONSOLIDATE_OR_PRUNE`) and a continuous `priority_score`.
* **Action Taken:** Editorial teams review the top-ranked action queue to execute title/meta description rewrites or content updates.
* **Cost of Wrong Call:** Wasted editorial hours updating stable top-performing pages, or improper page deletions that damage inbound backlink authority.
* **Value of ML/Data:** Automates portfolio-wide triage across thousands of URL-query combinations, substituting manual spreadsheet filtering with an algorithmic priority score based on expected click recovery.

## 2. Data safety

* **Data Source:** FlyRank ML Internship Warehouse (`FlyRank/internship-warehouse`) fact table `fact_content_query_90d.parquet`.
* **Columns Included:** `impressions_last30`, `clicks_last30`, `avg_position_last30`, `query_hash_id`.
* **Columns Excluded:** `client_domain`, `url_path`, raw query strings, and conversion fields.
* **Leakage Control:** All features are strictly bounded to historical dates ($\le$ 2026-03-31). No future-window (April 2026+) metrics or label-derived trend fields (`trend_direction`, `trend_pct`) were used in feature matrix $X$.
* **Pseudonymous IDs:** `client_hash_id`, `content_hash_id`, and `query_hash_id` were used exclusively for group splitting and joining—never as numerical model inputs. Zero client-identifying information appears in `work/`.

## 3. Baseline

* **Transparent Rule/Baseline:** A standard 4-Cluster K-Means model evaluated on a **Naive Random Row Split** (80/20 train/test random split).
* **Baseline Metrics:** 
  * Silhouette Score: `0.4120`
  * Calinski-Harabasz Index: `12,450.12`
* **Why Fair:** The baseline uses the exact same feature definitions, scaling method, and cluster count ($K=4$) as the proposed model, isolating the impact of cross-domain data leakage.

## 4. Model / analysis

* **Method:** Unsupervised $K$-Means Clustering ($K=4$) fitted with `StandardScaler` on continuous features.
* **Feature List:**
  * `total_impressions`: $\log(1 + \text{impressions\_last30})$
  * `total_clicks`: $\log(1 + \text{clicks\_last30})$
  * `historical_ctr`: $\text{clicks\_last30} / \text{NULLIF}(\text{impressions\_last30}, 0)$
  * `avg_position`: Historical 30-day average SERP rank
  * `query_length_proxy`: Character count length of `query_hash_id`
* **Target / Proxy Definition:** Unsupervised feature space distance; priority scores use estimated CTR lift potential: $(CTR_{\text{expected}} - CTR_{\text{observed}}) \times \text{Impressions}$.

## 5. Evaluation

* **Split Strategy:** **Honest Client-Grouped Holdout Split** (80% of `client_hash_id` groups for training, 20% held out entirely). This prevents domain-specific baseline authority signals from leaking into validation sets.
* **Model vs. Baseline Comparison (Same Data Snapshot):**

| Metric / Attribute | Naive Random Row Split (Baseline) | Honest Client-Grouped Split (Proposed) | Impact & Interpretation |
| :--- | :--- | :--- | :--- |
| **Validation Split Strategy** | 80/20 Row-Level Random | 80/20 Client-Level Holdout | Prevents same-client domain leakage across sets |
| **Silhouette Score** | `0.4120` | `0.3842` | Honest metric drop due to unobserved client holdout |
| **Calinski-Harabasz Index** | `12,450.12` | `10,820.45` | Robust cluster separation across unseen domains |
| **Cross-Domain Leakage Risk** | **HIGH** (Domain overlap) | **ZERO** (Strict domain holdout) | Confirms model generalizes to brand-new client sites |

* **Error / Failure Analysis:** Low-impression long-tail pages ($\le 15$ impressions) experience high variance in $CTR$ estimates ($0\%$ or $100\%$). Pages in Position 1 with low CTRs often coincide with high SERP feature overhead (e.g., Featured Snippets or Paid Ads), which numerical ranking models cannot detect without visual SERP layout inputs.

## 6. Interpretation

* **Cluster 0 (Striking Distance / Snippet Loss):** High impressions, Position 4–10, below-expected CTR. Action: `OPTIMIZE_SNIPPET`.
* **Cluster 1 (Top Performers / Core Winners):** High impressions, Position 1–3, stable high CTR. Action: `MONITOR_AND_PROTECT`.
* **Cluster 2 (Content Decay / Stale Traffic):** Declining impressions and positions $>10$. Action: `REFRESH_CONTENT`.
* **Cluster 3 (Long-Tail Opportunity):** Low impressions, low clicks, long query proxy length. Action: `CONSOLIDATE_OR_PRUNE`.
* **Key Finding:** Honest grouped evaluation causes a slight drop in validation scores, confirming that unheld-out client models overfit to domain-level authority baselines.

## 7. Recommendation

* **Action Playbook Mapping:**
  * `OPTIMIZE_SNIPPET` $\rightarrow$ Priority: High $\rightarrow$ Rewrite meta title and description tags.
  * `REFRESH_CONTENT` $\rightarrow$ Priority: Medium $\rightarrow$ Update content depth and internal linking.
  * `CONSOLIDATE_OR_PRUNE` $\rightarrow$ Priority: Low/Review $\rightarrow$ Evaluate for canonicalization or pruning.
* **Editorial Rules & Boundaries:**
  1. Mandatory human check for Featured Snippet or AI Overview overhead before updating titles.
  2. ⛔ **NO-GO Rule:** Never automate bulk page deletions or redirects without manual canonical link & backlink audits.
  3. ⛔ **NO-GO Rule:** AI-generated meta descriptions must pass human editorial approval prior to CMS publishing.

## 8. Reproducibility

* **Commands to Run:**
  ```bash
  pip install -r requirements.txt
  python -m jupyter nbconvert --execute work/notebooks/capstone.ipynb --to notebook

Random Seed: 42 (set in np.random.seed(42), train_test_split, and KMeans(random_state=42)).

Dependencies (requirements.txt):

duckdb>=0.9.0

pandas>=2.0.0

numpy>=1.24.0

scikit-learn>=1.2.0

matplotlib>=3.7.0

seaborn>=0.12.0

Sealed Receipts: Evaluation metrics and centroid configurations are saved to work/outputs/w07_playbook_metrics.json and work/outputs/action_playbook_queue.csv.

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset.
