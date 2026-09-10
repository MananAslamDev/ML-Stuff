# Predicting Content Refresh Opportunities: A Machine Learning Approach to Search Visibility

**Abstract**
Digital agencies frequently struggle to identify which underperforming web pages are worth the editorial cost of a rewrite. This research proposes a machine learning approach to predict content refresh opportunities by identifying pages with high search visibility but disproportionately low traffic. Utilizing a Random Forest Classifier trained on daily search and AI referral signals, the model isolates pages that mathematically *should* be generating sessions. Evaluated under a rigorous grouped-client split, the model outperformed rigid baseline heuristic rules in precision, significantly reducing false-positive recommendations. The resulting output is a ranked decision-support queue that allows editorial teams to efficiently target title and meta-data optimizations. 

**Introduction & Problem Statement**
In search engine optimization, visibility does not always equal traffic. When a page generates thousands of impressions but zero clicks, it represents a missed opportunity—often caused by poorly optimized titles or meta descriptions. However, manually analyzing millions of daily rows to find these anomalies is impossible for human editors. The decision this work supports is **prioritization**: how can an editorial team confidently decide which 10 pages to rewrite today to capture the most otherwise-lost traffic?

**Data**
This analysis was conducted using the `fact_content_daily_performance` table from the FlyRank internship data warehouse. 
* **Time Window:** The mid-panel month (March 2026) was selected to prevent non-random end-of-panel data anomalies. 
* **Exclusions:** Rows with missing (NULL) Search Console impressions were excluded, as a page cannot be evaluated for Click-Through-Rate (CTR) optimization if it is not being served in search results at all. Client names and identifiable URLs were hashed to preserve privacy.

**Methodology**
We framed this as a classification problem. The model's objective is to predict whether a page is receiving traffic (`ga4_sessions > 0`), using search visibility features (`gsc_impressions`, `ai_gemini`, `ai_copilot`). 
* **Baseline:** Our baseline was a hardcoded heuristic rule (e.g., Impressions > 10 = Expect Traffic). 
* **Model:** A Random Forest Classifier (max_depth=5, n_estimators=100) was chosen to capture non-linear relationships.
* **Validation Design:** To prevent identity leakage (the model memorizing which clients naturally get more traffic), we utilized a `GroupShuffleSplit` grouped by `client_hash_id`. This ensured the model was evaluated purely on its ability to generalize to completely unseen clients. 

**Results**
The Random Forest model successfully learned the directional relationship between search visibility and traffic expectation. Under a random split (which allows data leakage), the model showed artificially high precision. Under the honest, grouped-client split, the precision normalized but still outperformed the baseline heuristic. By interpreting the model's errors (predicting traffic where actual sessions were 0), we generated our list of high-confidence CTR-optimization targets. 

**Limitations & Honest Framing**
This model provides **decision-support**, not guaranteed outcomes. The relationships identified between search impressions, AI visibility, and traffic are **observational** and **directional**. The model cannot prove causality, nor does it guarantee that rewriting a flagged page will force a search engine to rank it higher. 

**Ranked Recommendations (Action Playbook)**
Based on the model's output, we recommend the following workflow for editorial teams:
1. **Extract the Top 20:** Pull the highest-probability false positives (high expected traffic, zero measured sessions).
2. **Human Review:** Verify that the page did not suffer a tracking pixel failure. 
3. **CTR Rewrite:** Update the title and meta description to better align with user search intent.

**Reproducibility & Acknowledgments**
The complete code is available in the `work/notebooks/` folder of this project repository. 
*Built on the FlyRank ML Internship dataset (https://flyrank.ai)*
