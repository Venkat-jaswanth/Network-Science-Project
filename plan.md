# Controversial Information Spreads Faster on Reddit — Project Summary, Replication Plan, and Extensions

**Prepared for:** Network Science Analytics — Group Project

**Paper:** *Controversial Information Spreads Faster on Reddit* (study of controversial vs non-controversial post diffusion on Reddit)

**Purpose of this document:**
This document explains the original paper in concise terms (research question, data, methods, findings), lays out a clear, reproducible replication plan using NetworkX + Python, and describes **multiple, well-defined extensions** your group can implement. Each extension maps directly to the course rubric (Phase 3 requirements) and contains a precise experimental plan, metrics, expected figures, and short pseudo-code / implementation notes.

---

## 1. Executive summary

* **Research question:** Do controversial posts on Reddit spread differently (faster, deeper, and to more users) than non-controversial posts? What structural network properties accompany controversy-driven spread?
* **Core claim (paper):** Controversial posts produce larger and deeper discussion cascades and tend to involve more unique participants than non-controversial posts.
* **Why this is a good project for the course:** The work is empirical (uses real Reddit data), directly maps to NetworkX-based analysis (cascade trees, centrality, clustering, community detection), and leaves clear space for extensions (simulation, robustness, homophily, model comparison).

---

## 2. Paper understanding — what they did and how

### 2.1 Research question (re-stated)

The paper asks whether content labelled controversial (or empirically flagged as controversial by signals such as high comment-to-upvote ratios, moderator flags, or community labels) triggers different diffusion patterns than non-controversial content. The focus is on *discussion spread* (the comment/reply tree) rather than simple upvote counts.

### 2.2 Data used (typical data sources)

* **Reddit comment and submission archives** (Pushshift and/or Kaggle extracts).
* For each **post**: submission metadata (timestamp, author, subreddit, title, score) and full comment tree (comment id, parent id, author, timestamp, body, controversiality flag if available).
* The dataset must include or enable a way to mark a post as *controversial* vs *non-controversial*. Options:

  * Use explicit Reddit `controversiality` flags if present.
  * Derive controversy by thresholding comment-to-upvote ratio, or by sentiment/disagreement heuristics, or by moderator-assigned flair if available.

**Dataset recommendation (practical):** Use a Kaggle subset or a Pushshift monthly dump limited to a few active subreddits (politics, news, technology, funny) for a 1–2 month window; this keeps the project computable on a laptop and reproducible among group members.

### 2.3 Core methods in the original study

1. **Construct cascade trees:** For each submission (post), construct the comment-reply tree. Each post yields a rooted tree where nodes are comments (or users) and edges are reply relationships.
2. **Cascade metrics:** Compute per-post metrics such as: cascade size (total comments), maximum depth, average branching factor, lifespan (time between first and last comment), and number of unique users involved.
3. **Group comparison:** Compare distributions of these metrics between controversial vs non-controversial posts using standard descriptive statistics and significance tests (KS-test or Mann–Whitney U).
4. **Network-level metrics:** Build an aggregated user–interaction network (nodes are users; edge if user A replied to user B at least once). Compute degree distributions, clustering coefficient, assortativity, and basic centrality measures.
5. **Interpretation:** Relate cascade and network metrics to human behavior — controversy increases engagement and breadth/depth, and often crosses multiple communities.

### 2.4 Key findings (paper summary)

* Controversial posts have **larger median cascade sizes** and **greater depths** than non-controversial posts.
* They attract **more unique participants**, indicating wider reach.
* Spread is often faster in the early phase of the cascade (shorter time to reach a certain fraction of total comments).

---

## 3. Replication plan (Phase 2) — detailed, step-by-step

Below is a compact replication procedure that fulfils your course Phase 2 requirements using NetworkX and Python.

### 3.1 Data selection & preprocessing

1. **Select subreddits & time window:** choose 3–4 subreddits that differ in tone (e.g., r/politics, r/technology, r/funny, r/science) and a 1–2 month period.
2. **Download comments and posts:** use Pushshift API or pre-downloaded Kaggle CSV. Limit to N posts per subreddit (e.g., 500–2000) so processing is manageable.
3. **Label posts as controversial vs non-controversial:** choose one of the following labelling heuristics (document your choice):

   * Use Reddit `controversiality` field when available.
   * Or compute `controversy_score = (num_comments / (score + 1))` and label top X% as controversial and bottom X% as non-controversial.
   * Or use a hybrid (explicit flag when available, otherwise proxy metric).
4. **Construct comment trees:** for each post, reconstruct the tree by linking comment `parent_id` → `id`. Keep timestamps and authors for each node.

### 3.2 Construct graphs (two complementary graphs)

**A. Per-post cascade trees (directed rooted trees)** — nodes = comment ids (or user ids), edges = reply relationships. Keep attributes: `timestamp`, `author`, `depth`.
**B. Aggregated user-interaction graph (directed/undirected)** — nodes = users; add an edge userA→userB if A replied to B at least once during the collection window. Optionally weight edges by reply count.

### 3.3 Compute required metrics (use NetworkX)

**Global / aggregated graph metrics:**

* Number of nodes and edges.
* Degree distribution (plot histogram + CCDF).
* Density.
* Clustering coefficient (avg / distribution).
* Centrality measures: degree, betweenness, closeness, eigenvector (top-20 lists).
* Community detection: Louvain or Girvan–Newman (community sizes, modularity).

**Per-cascade metrics (per-post trees):**

* Cascade size (total nodes).
* Maximum depth.
* Average branching factor = (total edges / internal nodes).
* Unique users.
* Lifespan (last_comment_time − first_comment_time).
* Structural virality (optional): compute Wiener index or average pairwise distance on the tree to quantify wide vs deep shape.

### 3.4 Recreate at least one figure from the paper

Suggested figure (clear and informative): **CDF/boxplot of cascade size for controversial vs non-controversial posts** and/or **boxplot of max depth for both groups**.  Implement with matplotlib/seaborn and label axes clearly.

---

## 4. Extension proposals (Phase 3) — choose ONE (we propose *Diffusion Simulation* as primary, plus backup extensions). Each extension is specified with the motivation, concrete steps, evaluation metrics, and expected figures.

### Extension A — **Diffusion Simulation (Primary recommendation)**

**Motivation:** The original paper is observational. A diffusion simulation tests mechanisms: how much does seed selection (central vs random users) change the spread dynamics? This directly compares structure → dynamics and goes beyond mere observation.

**Specific question:** If controversial content originates from highly central users, does it spread faster and to more users than when it originates from low-centrality users? How sensitive is spread to the adoption probability?

**Model & parameters:**

* Use the **Independent Cascade (IC)** model or **SIR** variant (IC is simpler and widely used).
* Transmission probability `p`: test multiple values (e.g., 0.01, 0.03, 0.05, 0.1).
* Seed strategies (compare):

  1. `top_degree`: pick top-k users by degree centrality (k = 1, 5, 10).
  2. `top_betweenness`: pick top-k users by betweenness.
  3. `random`: pick k random users.
  4. `controversial_authors`: users who often start controversial posts (if measurable).

**Implementation steps:**

1. Build aggregated user-interaction graph (undirected or directed; IC can work on either).
2. For each seed strategy and each p value, run M independent simulations (M = 100) and record:

   * final cascade size (mean + std),
   * time to reach 25% / 50% / 75% of final size (spread speed),
   * fraction of simulations that reach large outbreaks (e.g., > 10% of nodes).
3. Plot average cascade size vs p for each strategy, and plot spread curves (infected vs time) with error bands.

**Evaluation metrics & statistical tests:**

* Compare mean final sizes using t-test or non-parametric test (if distributions are skewed).
* Use effect size measures (Cohen’s d) and report confidence intervals.

**Expected deliverables / figures:**

* Line plot: `final_cascade_size` vs `p` (multiple colored lines for each seeding strategy).
* Spread-over-time curves (infected fraction vs time) with shaded std.
* Table: mean and std final sizes for each strategy at each p value.

**Why this extension is strong:** It is mechanistic (tests causality through simulation), easy to implement in NetworkX, and yields interpretable graphs for the report and presentation.

### Extension B — **Community-bridging & Contagion Across Communities**

**Motivation:** The original paper shows larger cascades for controversial posts, but do controversial cascades cross community boundaries more often? This tests whether controversy helps content jump from one community to another (a key diffusion insight).

**Specific question:** Are controversial cascades more likely to involve nodes from multiple communities (lower community-homogeneity) than non-controversial cascades?

**Steps:**

1. Detect communities in the aggregated user graph using Louvain.
2. For each post cascade, measure `community_spread = number_of_unique_communities / cascade_size` (or fraction of nodes outside seed community).
3. Compare distributions of `community_spread` for controversial vs non-controversial posts.

**Metrics / figures:**

* Boxplot or CDF of community_spread by group.
* Map a few example cascades on a 2D embedding (e.g., ForceAtlas / t-SNE) colored by community to visually show cross-community spread.

**Why good:** Straightforward to compute and directly addresses the question of controversy enabling cross-community diffusion — an insightful extension.

### Extension C — **Robustness / Node Removal (Influencer Deletion Experiment)**

**Motivation:** Real platforms moderate/band users. If highly influential users are removed, does the ability of controversial content to spread decrease more than for non-controversial content?

**Specific question:** Does removing top central users disproportionately reduce cascade sizes for controversial posts vs non-controversial posts?

**Steps:**

1. Identify top central users by degree and betweenness.
2. Remove top-r fraction of nodes (simulate bans) from the aggregated graph or from the set of potential seeds.
3. Re-run diffusion simulations (IC) for seeds drawn from the reduced network and compare final sizes to the original network.

**Metrics / figures:**

* Plot `fraction_removed` vs `relative_drop_in_mean_cascade_size` for both post types.

**Why good:** Connects network robustness with diffusion; uses simple node-removal experiments and IC simulations.

### Extension D — **Assortativity / Homophily by Controversy (Attribute-based analysis)**

**Motivation:** Discover whether users who post controversial content cluster together or connect preferentially to others who post controversial content.

**Question:** Is there assortative mixing by `controversy_label` among users? Do controversial authors reply more to other controversial authors?

**Steps:**

1. For each user, compute `controversy_rate = (# controversial posts authored) / (# posts authored)` over the dataset.
2. Discretize `controversy_rate` into bins (high/low) or use continuous assortativity measures.
3. Compute assortativity coefficient on the aggregated user graph with the controversy attribute.

**Why good:** Simple to compute; yields an extra interpretive dimension: whether controversy is a local phenomenon or spreads via bridges.

### Extension E — **Model comparison: Compare Reddit to synthetic models (ER / BA / WS)**

**Motivation:** Show how real Reddit network structure departs from classical generative models, then explain consequences for diffusion.

**Steps:**

1. Fit/generate synthetic graphs (same N and approximate average degree): Erdős–Rényi (ER), Barabási–Albert (BA), Watts–Strogatz (WS).
2. Compare degree distribution, clustering coefficient, average path length, modularity.
3. Run identical diffusion simulations on each model and compare outbreak sizes and speed to the real Reddit graph.

**Why good:** The comparison highlights which structural properties of Reddit (e.g., high clustering, heavy-tailed degree) are responsible for diffusion patterns.

---

## 5. Implementation notes & reproducibility checklist (code skeleton hints)

* Use **Python 3.9+**, `networkx`, `pandas`, `numpy`, `matplotlib`, `seaborn`, and `python-louvain` (for Louvain).
* Keep the notebook well-organized with sections: Data Load → Preprocessing → Graph Construction → Metrics → Replication Figures → Extension Experiments → Interpretation.
* Seed random number generators (`np.random.seed(0)`) to make simulation results reproducible.

### Example: Build aggregated user graph (pseudo-code)

```python
import pandas as pd
import networkx as nx

# comments_df columns: ['post_id','comment_id','parent_id','author','created_utc']
comments = pd.read_csv('comments_sample.csv')

G = nx.DiGraph()
for _, row in comments.iterrows():
    author = row['author']
    parent_id = row['parent_id']
    if pd.isna(parent_id):
        continue
    parent_author = comments.loc[comments['comment_id']==parent_id, 'author']
    if parent_author.empty:
        continue
    G.add_edge(author, parent_author)

# Optionally convert to undirected for diffusion
G_und = G.to_undirected()
```

### Example: Simple Independent Cascade simulation (pseudo-code)

```python
import random

def run_ic(G, seeds, p=0.01, max_steps=1000):
    activated = set(seeds)
    frontier = set(seeds)
    t = 0
    times = {s:0 for s in seeds}
    while frontier and t < max_steps:
        new_frontier = set()
        for u in frontier:
            for v in G.neighbors(u):
                if v in activated: continue
                if random.random() < p:
                    activated.add(v)
                    new_frontier.add(v)
                    times[v] = t+1
        frontier = new_frontier
        t += 1
    return activated, times
```

---

## 6. Expected figures and tables for the report

Include the following labeled figures/tables (all must have captions & axis labels):

1. **Dataset summary table**: number of posts, number of comments, time range, subreddits included.
2. **CDF/Boxplot**: cascade size — controversial vs non-controversial.
3. **CDF/Boxplot**: cascade depth — controversial vs non-controversial.
4. **Bar chart / table**: mean unique users per post with confidence intervals.
5. **Network summary**: degree distribution (log–log plot) + clustering coefficient.
6. **Extension-specific figures** (examples):

   * For Diffusion Simulation: final cascade size vs p (lines) by seeding strategy; spread-over-time curves.
   * For Community Spread: boxplots of community_spread.
   * For Robustness: fraction_removed vs relative drop in cascade size.

---

## 7. Phase 4 — Interpretation guidance (how to answer the mandatory questions)

When writing the Interpretation section, always connect metrics to claims. Examples:

* **Who are influential nodes and why?** Show top users by degree/betweenness/eigenvector and explain: "User X has high degree because they reply to many others and act as bridges between community A and B."
* **Is the network centralized?** Use degree distribution and Gini coefficient of degree; compare centralization indices.
* **Does it show clustering / community structure?** Provide modularity scores and community-size distribution.
* **What structural properties emerge?** Summarize: heavy-tailed degrees, high clustering, short average path length (if true).
* **Does the extension change conclusions?** For simulation extension: show how seed selection modulates spread — controversial posts may be more likely to explode only if seeded by influential users; report quantitative differences.

---

## 8. Deliverables & timeline (aligned to your course)

* **Mar 11 (Paper submission):** Submit this one-page approval draft + dataset link.
* **Mar 18 (Replication code):** Jupyter notebook with data loading, graph construction, and replication figures.
* **Mar 23 (Extension code):** Notebook section with extension experiments and figures.
* **Apr 01 (Report):** 12–15 page polished report following the provided structure.
* **Apr 08 (Presentation):** 10–12 minute slides with all members presenting (slides + 1 backup slide with extra analysis).


<!-- not needed -->
## 9. Group responsibilities (suggested split)

* **Member 1 (Data & Preprocessing):** Download data, label controversial posts, build comment trees.
* **Member 2 (Replication & Metrics):** Build aggregated graphs, compute metrics & produce replication figures.
* **Member 3 (Extension & Experiments):** Implement diffusion simulation / community extension and produce extension figures.
* **Member 4 (Report & Presentation):** Write report drafts, produce slides, coordinate figures and final polishing.

---

<!-- not needed -->
## 10. Short checklist to avoid common mistakes

* Label how you choose controversial vs non‑controversial and justify threshold.
* Keep dataset size manageable and document all filters.
* Use reproducible seeds and provide a `requirements.txt` (or `environment.yml`).
* Include in the notebook a fully automated `run_all` cell that regenerates figures from raw data.
* Properly caption every figure and reference it in the text.

---

## 11. References / useful resources

* Pushshift Reddit dataset / API (for downloading comments).
* NetworkX documentation (graph building, centralities).
* python-louvain (community detection) and `community` package.

---

### Final note

This document gives you a fully reproducible, instructor-aligned plan for the **Controversial Information Spreads Faster on Reddit** project, with a primary extension (diffusion simulation) and several strong backup choices. If you want, I can now:

* Produce a 1‑page instructor approval draft based on this document, or
* Generate the starter Jupyter notebook skeleton (data loader + graph builder + replication plots), or
* Prepare the `requirements.txt` and exact download commands for Pushshift/Kaggle.

Tell me which of those you want next and I’ll produce it immediately.
