# Climate Claim Verification — Baseline Retrieval Experiments
**气候声明核查系统 — 检索基线实验报告**

---

## Project Overview / 项目概述

Automated fact-checking system for climate-related claims against a large-scale evidence corpus.  
针对气候相关声明构建的自动事实核查系统，核心任务为在大规模证据语料库中检索支撑证据并进行标签分类。

**Dataset Statistics / 数据集规模**

| Split | Claims | Evidence Passages |
|-------|--------|-------------------|
| Train | 1,228  | 1,208,827         |
| Dev   | 154    | —                 |
| Test  | 153    | —                 |

**Label Distribution (Train) / 训练集标签分布**

| Label           | Count |
|-----------------|-------|
| SUPPORTS        | 519   |
| NOT_ENOUGH_INFO | 386   |
| REFUTES         | 199   |
| DISPUTED        | 124   |

**Majority baseline accuracy:** 44.16% (predicting SUPPORTS for all)

**Evaluation Metric:** Harmonic Mean of Evidence F-score × Classification Accuracy  
评估指标为证据 F-score 与分类准确率的调和均值。

---

## Baseline Results Summary / 基线结果汇总

| Method                         | top-k | Evidence F-score | Accuracy | **Harmonic Mean** | Runtime   |
|-------------------------------|-------|------------------|----------|-------------------|-----------|
| Dummy baseline                 | 1     | 0.0000           | 0.4416   | 0.0000            | —         |
| TF-IDF                         | 1     | 0.0978           | 0.4416   | 0.1602            | —         |
| TF-IDF                         | 3     | 0.0959           | 0.4416   | 0.1576            | —         |
| TF-IDF                         | 5     | 0.0883           | 0.4416   | 0.1471            | 140.22s   |
| TF-IDF                         | 10    | 0.0724           | 0.4416   | 0.1244            | —         |
| TF-IDF top-100 + BM25 rerank   | 5     | 0.0704           | 0.4416   | 0.1214            | 74.72s    |
| Full BM25 (label = NEI)        | 5     | 0.0673           | 0.2662   | 0.1075            | 1,778.66s |
| **Full BM25 optimized**        | **5** | **0.1053**       | **0.4416** | **0.1701**      | ~1,802s   |

**Best baseline: Full BM25 optimized — Harmonic Mean 0.1701**

---

## Methods & Analysis / 方法与分析

### 1. Dummy Baseline
- Predicted majority label (`SUPPORTS`) and a fixed evidence passage for all claims.
- Verified output format and evaluation pipeline; evidence F-score = 0 as expected.

### 2. TF-IDF Retrieval
- Built sparse TF-IDF matrix (shape: 1,208,827 × 531,771) over full evidence corpus.
- Matched each claim via cosine similarity; top-k evidence selected.
- Evidence F-score peaked at top-1 (0.0978) and degraded with larger k, indicating higher k introduced more false positives and hurt precision.
- **Failure mode:** surface-level keyword bias — a claim mentioning *"3 per cent"* retrieved unrelated passages sharing only the token *"per cent"*, missing the actual topic (CO₂ emissions, Australia, climate).

### 3. Full BM25 Retrieval
- Indexed all 1.2M passages with BM25Okapi; used regex tokenizer for cleaner term matching.
- Achieved the **best retrieval F-score among lexical methods (0.1053)**.
- BM25's term-frequency saturation and document-length normalization outperformed raw TF-IDF.
- **Success case:** for the claim *"South Australia has the most expensive electricity in the world"*, BM25 top-2 results were exactly the two gold evidence passages.
- **Limitation:** full corpus scoring required ~30 min per dev set evaluation.

### 4. TF-IDF + BM25 Two-Stage Reranking
- Stage 1: TF-IDF retrieved top-100 candidates from full corpus (fast).
- Stage 2: BM25 reranked within top-100 candidates only.
- Runtime dropped to **74.72s**, but F-score (0.0704) fell below either method alone.
- Root cause: local BM25 IDF computed on 100-candidate subset was unstable; if gold evidence missed TF-IDF top-100, it was unrecoverable.

---

## Key Takeaways / 核心结论

- Lexical retrieval (TF-IDF / BM25) is a necessary but insufficient baseline for fact-checking; surface-level token overlap cannot capture paraphrases or indirect support.
- BM25 achieved the best lexical retrieval quality but at prohibitive runtime cost (~30 min for 154 claims).
- Two-stage pipelines can dramatically cut runtime but require careful calibration of candidate set size and reranking strategy.
- These results motivate moving to **dense semantic retrieval** (e.g., bi-encoder / cross-encoder) for meaningful improvements.

**词面检索是必要但不充分的基线。BM25 在词面匹配方法中表现最优（F-score 0.1053），但受限于语义缺失和运行时间开销，为后续引入稠密语义检索提供了量化基准。**

---

## Runtime vs. Quality Trade-off / 效率与质量权衡

| Method                        | Runtime   | Harmonic Mean | Note                     |
|------------------------------|-----------|---------------|--------------------------|
| Full BM25 optimized           | ~1,802s   | **0.1701**    | Best quality, slowest    |
| Optimized TF-IDF sparse       | 140.22s   | 0.1486        | Good balance             |
| TF-IDF top-100 + BM25 rerank  | 74.72s    | 0.1214        | Fastest, quality drops   |
