# Climate Claim Verification / 气候声明事实核查

A multi-stage fact-checking system that, given a climate-related claim, **retrieves supporting evidence** from a 1.2M-passage corpus and **classifies** the claim as `SUPPORTS` / `REFUTES` / `NOT_ENOUGH_INFO` / `DISPUTED`.

多阶段事实核查系统:给定一条气候相关声明,从 **120 万条**证据语料中**检索支撑证据**,并将声明**分类**为 支持 / 反驳 / 信息不足 / 存在争议。

## Pipeline / 流程

```
Claim / 声明   (evidence corpus: 1.2M passages / 证据语料 120 万条)
  │
  ├─ Stage 1  Dual TF-IDF retrieval (unigram + unigram-bigram) → RRF fusion
  │           双路 TF-IDF 召回 → RRF 融合
  │           Why: cheap lexical filter over a huge corpus, maximise recall
  │           为什么: 语料太大,先用廉价词面检索粗筛,保召回
  │           1.2M ──▶ top-400
  │
  ├─ Stage 2  CrossEncoder (MiniLM) reranking + score fusion (α=0.03)
  │           CrossEncoder 语义重排 + 分数融合
  │           Why: lexical match ignores semantics; cross-attention refines, cheap on 400
  │           为什么: 词面不懂语义,交叉注意力精排;只算 400 条,成本可控
  │           400 ──▶ ranked
  │
  ├─ Stage 3  Gap-based adaptive evidence selection
  │           基于分差的自适应证据选择
  │           Why: evidence count varies per claim; fixed top-k adds noise
  │           为什么: 各 claim 证据数不同,固定 top-k 会引噪声
  │           ──▶ 1–6 passages
  │
  └─ Stage 4  DeBERTa-v3 four-way classifier
              DeBERTa-v3 四分类
              Why: NLI pretraining aligns with "does evidence support the claim"
              为什么: NLI 预训练天然对齐"证据是否支持声明"
        │
   Evidence set + Label / 证据集 + 标签
```

**漏斗式设计 / Funnel design:** 用廉价的词面检索把 120 万条缩小到几百条,再用昂贵的语义重排精修 —— 每一阶段都在效率与精度间做取舍。 A funnel that narrows 1.2M passages to a handful, trading efficiency against accuracy at each stage.

## Results / 结果 (dev set)

| System / 系统 | Evidence F-score | Accuracy | Harmonic Mean |
|--------------|------------------|----------|---------------|
| BM25 baseline | 0.1053 | 0.4416 | 0.1701 |
| **Final system / 最终系统** | **0.2383** | **0.5649** | **0.3352** |

**Key finding / 核心发现:** 用**检索证据**替代**金标准证据**训练分类器,对齐训练与推理分布,准确率由 **0.4814 提升至 0.5649**。

📊 Full ablation study (component ablation · 1127-config grid search · selection-strategy comparison) → [`results/ablation_results.md`](results/ablation_results.md)
完整消融实验(组件消融 · 1127 配置网格搜索 · 选择策略对比)见 [`results/ablation_results.md`](results/ablation_results.md)

## Repository / 仓库结构

```
notebooks/
  final_retrieve_classify.ipynb        # 完整端到端系统(提交用)
  Dual_TF-IDF_Score_Fusion_Gap.ipynb   # 检索超参搜索 / 消融实验
results/
  baseline_results.md                  # 词面 baseline 探索
  ablation_results.md                  # 消融实验 & 网格搜索
docs/report.pdf                        # 完整报告
data/README.md                         # 数据集说明(数据未包含在仓库)
```

## Stack / 技术栈

Python · PyTorch · HuggingFace Transformers · sentence-transformers · scikit-learn

## Usage / 运行

数据未包含在仓库(见 `data/README.md`)。将数据放到 `data/` 下，用 `pip install -r requirements.txt` 安装依赖，运行 `notebooks/` 中的 notebook(为 Google Colab + GPU 设计)。

Data is not included (see `data/README.md`). Place the dataset under `data/`, install dependencies with `pip install -r requirements.txt`, and run the notebooks in `notebooks/` (designed for Google Colab with GPU).
