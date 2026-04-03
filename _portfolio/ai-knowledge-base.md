---
title: "Academic Literature Q&A Agent"
excerpt: "A full-stack AI knowledge base system for research groups, integrating RAG, short-term & long-term memory, and Function Calling.<br/>"
collection: portfolio
---

## 科研文献智能问答系统

**个人独立项目** · 2025

### 项目简介

面向科研课题组设计并实现的**全栈 AI 知识库问答系统**，支持上传 PDF / Word 学术文献，进行深度多轮对话式检索。主要解决以下痛点：

| 痛点 | 本系统方案 |
|------|-----------|
| 论文数量多，检索效率低 | PDF 自动解析 → 语义分块 → 混合检索 |
| LLM 不了解组内论文细节 | RAG 将相关段落动态注入上下文 |
| 多轮对话容易遗忘前文 | 滑动窗口 + 摘要压缩短期记忆 |
| 需要查结构化信息（作者/年份/数据集）| Function Calling 查询 SQLite |
| 用户研究习惯需要持久化 | ChromaDB 向量化长期记忆 |

### 技术架构

```
前端 (HTML)  →  FastAPI 后端  →  Chat / RAG / Memory / Function Service
                                       ↓
                         LLM Caller (DeepSeek-V3, 限流+重试+Token 预算)
                                       ↓
                   ChromaDB (向量检索)  +  SQLite (结构化查询)
                                       ↓
                   BAAI/bge-small-zh-v1.5 (本地离线 Embedding)
```

### 核心技术模块

**① 混合检索（RAG）**
- **语义向量检索**：ChromaDB + `BAAI/bge-small-zh-v1.5`，余弦相似度匹配语义近似段落
- **BM25 关键词检索**：`rank_bm25` + `jieba` 中文分词，精确匹配方法名、论文标题等专有名词
- **RRF 融合排序**：两路结果合并排序，兼顾语义与精确召回
- **Query Rewriting**：结合短期记忆上下文，将含省略/指代的问题改写为完整疑问句
- **语义分块**：余弦相似度驱动的动态合并，避免固定长度截断导致语义断裂

**② 记忆系统**
- **短期记忆**：内存 `deque` 滑动窗口（保留最近 N 轮）；超过 token 上限时，对早期对话调用 LLM 做摘要压缩
- **长期记忆**：用户主动录入的研究偏好，向量化存入 ChromaDB，对话时语义相似度召回 Top-k 相关记忆

**③ Function Calling**
- 自定义 `search_papers`、`search_experiments` 等工具函数
- LLM 自动判断是否需要调用，按作者、年份、数据集、方法名等维度结构化查询 SQLite

**④ 工程能力**
- **FastAPI** RESTful 后端，4 个路由模块（chat / document / memory / usage）
- 限流器 + 指数退避重试 + Token 预算监控，防止 API 超额
- 纯 HTML 前端，无框架依赖，聊天面板 + 文献管理 + 记忆管理 + Token 用量看板

### 技术栈

`Python` · `FastAPI` · `PyTorch` · `ChromaDB` · `SQLite / SQLAlchemy` · `BM25 / jieba` · `DeepSeek-V3` · `HuggingFace` · `pypdf` · `BAAI/bge-small-zh-v1.5`

### 代码仓库

> [github.com/MengyuanTon/ai-knowledge-base](https://github.com/MengyuanTon/ai-knowledge-base)
