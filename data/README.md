# Data Directory / 数据目录说明

> **Note:** Data files are NOT included in this repository due to size constraints.  
> **注意：** 由于文件体积较大，数据文件未上传至本仓库。

---

## Files / 文件列表

| File | Size | Description |
|------|------|-------------|
| `train-claims.json` | — | Training set: 1,228 labelled claims with gold evidence IDs |
| `dev-claims.json` | — | Development set: 154 labelled claims with gold evidence IDs |
| `test-claims-unlabelled.json` | — | Test set: 153 claims without labels (for final evaluation) |
| `dev-claims-baseline.json` | — | Baseline prediction file for dev set (format reference) |
| `evidence.json` | ~166 MB | Full evidence corpus: 1,208,827 passages |

---

## Data Format / 数据格式

### Claims (`train-claims.json`, `dev-claims.json`)

```json
{
  "claim-id": {
    "claim_text": "...",
    "claim_label": "SUPPORTS | REFUTES | NOT_ENOUGH_INFO | DISPUTED",
    "evidences": ["evidence-XXXXX", "evidence-XXXXX"]
  }
}
```

### Test Claims (`test-claims-unlabelled.json`)

```json
{
  "claim-id": {
    "claim_text": "..."
  }
}
```

### Evidence (`evidence.json`)

```json
{
  "evidence-XXXXX": "passage text ...",
  ...
}
```

---

## Label Distribution / 标签分布

**Training set (1,228 claims):**

| Label           | Count | Ratio  |
|-----------------|-------|--------|
| SUPPORTS        | 519   | 42.3%  |
| NOT_ENOUGH_INFO | 386   | 31.4%  |
| REFUTES         | 199   | 16.2%  |
| DISPUTED        | 124   | 10.1%  |

**Dev set (154 claims):**

| Label           | Count |
|-----------------|-------|
| SUPPORTS        | 68    |
| NOT_ENOUGH_INFO | 41    |
| REFUTES         | 27    |
| DISPUTED        | 18    |

---

## How to Obtain the Data / 如何获取数据

This dataset is from the **COMP90042 Climate Claim Verification** project (University of Melbourne).  
该数据集来自墨尔本大学 COMP90042 课程气候声明核查项目。

Please contact the course coordinators or refer to the course LMS for access.  
如需获取数据，请联系课程负责人或参考课程 LMS 平台。

---

## Notes / 备注

- `evidence.json` contains 1,208,827 passages (~166 MB) and is the primary bottleneck for retrieval experiments.
- All claim IDs follow the format `claim-{N}`; all evidence IDs follow `evidence-{N}`.
- `dev-claims-baseline.json` serves as a format reference for prediction output — see `eval.py` for the evaluation script.

- `evidence.json` 包含约 120 万条证据段落（约 166 MB），是检索实验的主要性能瓶颈。
- Claim ID 格式为 `claim-{N}`，Evidence ID 格式为 `evidence-{N}`。
- `dev-claims-baseline.json` 为预测输出的格式参考文件，评估脚本见 `eval.py`。
