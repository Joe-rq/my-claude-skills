---
name: source-tracing
description: >
  从高熵的二手内容中回溯到低熵、权威、原始的信息源。
  通过结构化步骤帮助用户验证消息真伪、理解原始语境，
  建立独立的信息判断与溯源能力。
---

# 信息溯源

从二手传播内容中回溯到权威原始信息源，通过 6 步结构化流程完成信息去噪和验证。

## 输入参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `claim` | text | 是 | 一条未经验证的信息或高热度说法 |
| `context_hint` | text | 否 | 时间、机构、人名或事件关键词等上下文线索 |
| `verification_goal` | text | 否 | 溯源目标，如"找到原视频"、"查看完整讲话稿" |

## 输出

| 字段 | 类型 | 说明 |
|------|------|------|
| `source_trace_log` | JSON | 包含源头链接、时间、机构、主要论点、上下文摘要的结构化记录 |
| `trust_index` | number (0-1) | 源头权威性 + 内容完整性评分 |
| `entropy_delta` | number | 信息熵下降幅度（表示信息被"去噪"的程度） |

## 溯源流程（6 步）

### 1. 去包装 (Depackaging)

识别该信息的传播链条，查找最初的发布主体（如官方账号、原作者、机构源）。

### 2. 源头定位 (Source Location)

搜索并访问权威渠道（政府官网、机构公告、官方 YouTube 频道、学术数据库）。

### 3. 原文解读 (Primary Analysis)

阅读或观看原始内容，记录关键论点、关键词与发言语境。

### 4. 语境还原 (Context Reconstruction)

分析原文语气与时空语境，避免被剪辑或误导。

### 5. 多源对照 (Cross Verification)

对比两个以上独立来源，确认一致性。

### 6. 结构存档 (Structured Archiving)

输出溯源记录，包含原始链接、日期、机构、主要观点、对照结果。

## 示例

**输入：**
```
claim: "SEC 将全面禁止算法稳定币"
context_hint: "2025年3月，Gary Gensler 演讲"
verification_goal: "找到完整演讲视频并核对原文"
```

**输出：**
```json
{
  "source_trace_log": {
    "source_url": "https://youtube.com/watch?v=xxxxxx",
    "source_date": "2025-03-17",
    "speaker": "Gary Gensler",
    "institution": "U.S. SEC",
    "summary": "讨论稳定币监管框架与透明度原则，未提及全面禁止。"
  },
  "trust_index": 0.95,
  "entropy_delta": 0.82
}
```

## Primitive IR 映射

| Primitive 类型 | 对应概念 |
|----------------|---------|
| Entity | SEC |
| Event | Public Speech |
| Resource | Official Video / Transcript |
| Action | Source Tracing |
| Policy | Transparency & Regulation |
| Ledger | Source Verification Log |

## 依赖工具

- **web.search** — 用于定位权威信息源和官方媒体频道
- **summarizer** — 提取原文要点并生成结构化摘要

## 推荐链式调用

Source Tracing → Fact Comparison → Context Analysis → Structural Summary
