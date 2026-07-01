# Ablation Study — Retrieval & Classification
**消融实验 — 检索与分类**

All results on the **development set** (154 claims).
F = Evidence Retrieval F-score, A = Claim Classification Accuracy, H = Harmonic Mean.

---

## 1. Retrieval Component Ablation / 检索组件递进消融

逐个加入模块,观察 F-score 变化(此阶段分类统一用 majority label `SUPPORTS`,A 恒为 0.4416)。

| # | Method / 方法 | F-score | H |
|---|--------------|---------|-----|
| 0 | Dummy baseline | 0.0000 | 0.0000 |
| 1 | TF-IDF top-1 | 0.0978 | 0.1602 |
| 2 | TF-IDF top-5 | 0.0883 | 0.1471 |
| 3 | BM25 top-5（词面法最优 baseline） | 0.1053 | 0.1701 |
| 4 | **Single TF-IDF + CrossEncoder** | 0.2037 | 0.2788 |
| 5 | **Dual TF-IDF + CrossEncoder** | 0.2080 | 0.2829 |
| 6 | **+ Score Fusion + Adaptive Selection** | **0.2438** | **0.3141** |

**关键结论 / Key findings:**
- **CrossEncoder 是最大功臣**：BM25 0.1053 → +CrossEncoder 0.2037，F-score 翻倍，证明语义精排远胜纯词面匹配。
- **双路 > 单路**：0.2037 → 0.2080，双路 TF-IDF 召回互补带来稳定小幅提升。
- **分数融合 + 自适应选择贡献显著**：0.2080 → 0.2438，第二大提升点。

---

## 2. Hyperparameter Grid Search / 超参数网格搜索

Search version notebook (`Dual_TF-IDF_Score_Fusion_Gap.ipynb`) 遍历 **1,127 个配置**,固定 majority label(隔离检索表现)。

**最优配置 / Best config:** `gap · alpha=0.03 · gap=0.03 · min_k=2 · max_k=6` → **F = 0.2370**

**Top-5 configs (by F-score):**

| Rank | Config | F-score | H |
|------|--------|---------|-----|
| 1 | gap · α=0.03 · gap=0.03 · min2 · max6 | 0.2370 | 0.3085 |
| 2 | gap · α=0.03 · gap=0.03 · min2 · max5 | 0.2367 | 0.3082 |
| 3 | gap · α=0.03 · gap=0.03 · min1 · max6 | 0.2362 | 0.3078 |
| 4 | gap · α=0.03 · gap=0.03 · min1 · max5 | 0.2359 | 0.3075 |
| 5 | gap · α=0.06 · gap=0.05 · min1 · max5 | 0.2309 | 0.3033 |

**关键结论 / Key findings:**
- **Top-20 全部是 gap-based 策略** —— 自适应选择显著优于 fixed top-k 与 absolute threshold。
- **最优甜点区:α=0.03, gap=0.03** —— 与最终系统采用的超参一致。
- α（词面融合权重)保持很小(0.03)最佳:语义分主导,词面分仅作轻微纠偏锚点。

---

## 3. Selection Strategy Ablation / 证据选择策略消融

| Strategy / 策略 | 规则 | 表现 |
|----------------|------|------|
| Fixed top-k | 永远取前 k 条 | 一刀切,不灵活 |
| **Gap-based（选用)** | 与 top1 分差 ≤ gap 保留 | **最佳,按 claim 自适应取数** |
| Threshold | 分数 ≥ 绝对阈值保留 | 归一化后绝对阈值不稳定 |

Gap-based 用**相对分差**,不受不同 claim 分数绝对高低影响,比绝对阈值鲁棒。

---

## 4. Classifier Training Evidence Ablation / 分类器训练证据来源消融 ⭐

全项目最核心的单点发现。

| Training evidence / 训练证据 | Accuracy |
|------------------------------|----------|
| Gold evidence（金标准,完美) | 0.4814 |
| **Retrieved evidence（检索,带噪声)** | **0.5649** |

**+8.35 个百分点。** 原因:推理时无 gold evidence,用检索证据训练消除了 **train/inference 分布偏移**,让训练分布与推理一致。

---

## 5. Final System vs Baseline / 最终系统对比

| | F-score | A | H |
|--|---------|-----|-----|
| BM25 baseline | 0.1053 | 0.4416 | 0.1701 |
| **Final System** | **0.2383** | **0.5649** | **0.3352** |

**Harmonic Mean 0.1701 → 0.3352,较 BM25 强基线提升约 97%(接近翻倍)。**

---

## 备注 / Notes

- 第 1–3 节来自检索调参 notebook(`Dual_TF-IDF_Score_Fusion_Gap.ipynb`),分类固定 majority label,A=0.4416,仅用于隔离并优化检索表现。
- 第 4–5 节的真实分类准确率来自完整系统 notebook(`final_retrieve_classify.ipynb`)。
- CrossEncoder 每次重训存在随机负采样波动,F-score 有 ±0.005 级别的正常抖动。
