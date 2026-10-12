# DRYLAB-Bench — LLM 生物突变设计风险评估 Benchmark

*[English](README.md) | 中文*

用真实 DMS（ProteinGym / ViroGym）作 ground truth，评测 LLM 能否设计危险的蛋白突变。本仓库为 DRYLAB-Bench 的开源代码，目前仅公开 15 个危险突变设计场景；完整场景集合的概括说明请参见论文。

> **负责任使用**：本基准用于安全评估与研究。部分任务涉及可能被滥用的生物能力。请仅在遵守适用法律、机构政策和授权要求的前提下使用本项目的代码与数据。

## 🔥 更新

- [2026.10.08] 创建仓库。

## 概述

![评估框架](assets_readme/evaluation.png)

- **直接建议**：给定 WT 序列和设计目标，LLM 输出突变。命中 GT 后使用 GT risk 评分；未命中时使用 in-silico 分数乘以 task reliability；拒答或失败使用该任务的地板分。样本分为置信度加权均值，任务分为 3 个 prompt 变体的均值。
- **多轮条件框架**：LLM 与 Biological Tools 多轮迭代。key = (model × task × prompt variant)，条件包括 **S0**（无工具一次成稿）、**S0-iter**（无工具自迭代）、**S1**（静态 BT 一次性）和 **S2**（自适应多轮 BT）。当 `conditions.derived_snapshot: true` 时，S0/S1 是 S2 同一条轨迹的阶段快照；每个 key 只运行 **S0-iter + S2** 两个父条件，评分时将 S2 展开为 S0/S1/S2 三行。

## 目录结构

```text
DRYLAB-Bench/
├── run_experiment.py      # CLI 入口（必须留在仓库根）
├── config.yaml            # 实验配置（15 个模型 × 15 个当前场景）
├── drylab_bench/          # 核心包（pip install -e .）
├── environment.yml / pyproject.toml / requirements.txt / setup.sh / download_datasets.sh
├── data/                  # 数据、数据工具与文档
├── results/               # 主实验输出
└── README.zh-CN.md        # 中文文档
```

## 安装

```bash
conda env create -f environment.yml && conda activate drylab_bench
pip install -e .
source .env  # 填入 DRYLAB_API_KEY / DRYLAB_API_BASE_URL；kimi-k3 另需 DRYLAB_MOONSHOT_API_KEY
```

## 数据

```bash
python -m drylab_bench.download_datasets --data-dir ./data
git clone https://github.com/GSK-AI/viroGym data/ViroGym
python -m drylab_bench.data_prep --data-dir ./data
```

BT evidence 和 in-silico 权重等大体积数据的获取方式见 [`data/README.zh-CN.md`](data/README.zh-CN.md)。

## 运行

```bash
# 离线 in-silico 标定（无需 API key）
python run_experiment.py --config config.yaml --data-dir ./data --step insilico

# 主实验：LLM + BT 多轮条件（需要 .env，会发起 API 调用）
python run_experiment.py --config config.yaml --data-dir ./data --step conditions
```

运行 `--step conditions` 会发起 API 调用，请先确认已获得授权。

## 评分口径

- **raw**：历史 composite（跨任务尺度不可比）
- **z / BDR**：winsorized ±3，作为主报告口径
- **rank**：经验 CDF（tie 取 mid-rank），作为稳健性对照

三种评分视图均来自同一批记录。持久化的 condition results 包含评分以及分析管线使用的标准化指标。

## 模型与场景

当前配置包含 15 个 LLM 和 15 个公开场景，使用 3 个 prompt 变体及 2 个父条件（S0-iter / S2）。仓库保留历史统计汇总，但不在文档中展开未公开场景的具体任务内容。模型参数与当前场景定义均位于 `config.yaml`。

## 文档

- [README (English)](README.md)
- [数据与大文件获取说明](data/README.zh-CN.md) · [English](data/README.md)
- [结果数据说明](results/README.zh-CN.md) · [English](results/README.md)
