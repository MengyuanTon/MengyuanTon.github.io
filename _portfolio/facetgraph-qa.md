---
title: "FacetGraph-QA: Automated Training Data Generation for Medical LLMs"
excerpt: "An automated pipeline that generates high-quality multi-turn QA datasets for fine-tuning medical domain LLMs, using knowledge graph traversal and a novel Facet multi-narrative framework.<br/>"
collection: portfolio
---

## FacetGraph-QA：医药领域大模型训练数据自动化生成系统

**个人独立项目** · 2025.12 – 2026.03  
[GitHub 代码仓库](https://github.com/MengyuanTon/FacetGraph-QA)

---

### 项目背景

医药垂直领域大模型的监督微调（SFT）与推理链训练，高度依赖**高质量、多样性的领域标注数据**。然而人工标注成本极高，且难以覆盖复杂的关联推理场景。本系统旨在以自动化流水线替代人工标注，低成本构造兼具**事实精度、推理深度与视角多样性**的医药 QA 训练数据集。

---

### 核心架构：五步多 Agent Pipeline

```
[Step 1] KG API 随机游走 → 获取2跳邻域实体（全局去重）
         ↓
[Step 2] question_creator → 基于实体上下文生成深度领域问题
         ↓
[Step 3] facet_planner → 为问题规划 N 个独立完整叙事框架（Facets）
         ↓
[Step 4] facet_reducer → 弹性数量控制（<2时补充 / ≥8时筛选至8）
         ↓
[Step 5-a] FacetGraph QA Agent × N（并发）→ 每个 Facet 独立生成完整答案
           · 强制 <think> 推理链 + 溯源引用
           · 5步 Evidence-First 工作流
         ↓
[Step 5-b] facet_redundancy_detector → LLM 语义去重（结构性判断）
         ↓
[Step 6] Multi-Answer Synthesis Agent → 多视角综合为结构化最终答案
         ↓
输出 { question, planners[], history[], summary }
```

多轮循环（max_rounds=5），每轮 `history[]` 携带前轮 Q&A，支持上下文记忆保持能力的训练。

---

### 核心创新

#### ① Facet 多叙事框架机制（最核心）

**Facet ≠ 子问题拆解**。对同一个问题，系统规划多个独立完整的*叙事重心*，每个 Facet 都能引导出针对该问题的一篇完整回答：

| 问题示例 | Facet 视角 |
|---------|-----------|
| SLC19A1 基因型与培美曲塞总生存率的关联 | 临床证据综述 |
| | 药理学机制解析（转运蛋白功能） |
| | 预后生物标志物价值评估 |
| | 研究方法论局限性分析 |

每个 Facet 单独调用 Agent 生成携带 `<think>` 推理链的完整答案，**一次生成同时兼容普通 SFT 与 CoT 推理两种训练格式**，数据利用率翻倍。

#### ② Evidence-First Prompt 工程

FacetGraph QA Agent 使用精密设计的 Prompt，强制每个关键结论附带溯源引用 `[证据Rx：来源=refs:《文档名》]`，并设置 6 条质量红线（无证据支撑、片面覆盖、工具痕迹残留等任意触发则视为失败），确保生成数据具备**可追溯的证据链**。

#### ③ KG 驱动无限问题生成

从外部医药知识图谱（`ai.yzint.cn`）以 `hopCount=2` 随机游走取实体，驱动问题生成，保证：
- 问题涉及跨实体的**复杂关联推理**，而非简单事实问答
- 全局维护 `used_entity_ids` 集合，避免重复实体
- 理论上**无限不重复**采样

#### ④ LLM 语义去重替代 Embedding 相似度

冗余检测器让 LLM 从*推理路径、证据类型、组织结构*三维度判断两个 Facet 是否本质相同，能识别"换了说法但逻辑相同"的语义重复——这是 embedding 相似度做不到的。

---

### 输出数据格式

```json
{
  "question": "SLC19A1基因型如何影响培美曲塞疗效？",
  "planners": [
    {
      "facet": "临床证据综述",
      "think": "<think><facet=临床证据综述>Step A: ...\nStep B: ...\n[证据R1: 来源=refs:《PMID12345》]...</facet></think>",
      "answer": "基于多项临床研究，SLC19A1 84C>T 多态性..."
    },
    ...
  ],
  "history": [...],
  "summary": "综合四个视角的结构化最终回答..."
}
```

---

### 技术栈

`Python` · `Qwen 3.5-plus` · `医药知识图谱 API` · `OpenAI 兼容接口` · `requests` · `指数退避重试`

---

### 与普通 RAG/QA 系统的本质区别

| 维度 | 普通 RAG/QA | FacetGraph-QA |
|------|------------|---------------|
| **系统目标** | 直接服务终端用户 | 生产 LLM 训练数据 |
| **问题来源** | 用户手动输入 | KG 随机游走自动生成 |
| **回答结构** | 单一回答 | N 个独立完整视角 + 综合答案 |
| **推理过程** | 隐含于模型内部 | 显式 `<think>` 标签暴露证据推理链 |
| **质量控制** | 无或简单相关性过滤 | LLM 语义去重 + 6 条质量红线 + 强制证据引用 |
| **训练价值** | 低（缺乏推理链） | 高（CoT + 多视角 + 证据链完整） |
