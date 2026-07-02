# Proposal Schema（捕捉 → 落库的契约）

仅用于并行编排:`capture-only` 模式产出、`integrate` 模式消费。`standalone`(单条/默认)模式不产生 proposal——那时无并发,蒸馏 agent 直接写共享页。

## 位置与生命周期

- 每个 capture-only 子 agent 把自己这一条的 proposal 写到库根目录 `_staging/<source-id>.md`(transient 暂存区)。
- `integrate` 读完全部 `_staging/*.md` 并应用后,**删除已消费的 proposal**。
- `_staging/` 是 transient,可 gitignore。它存在的意义是**断点续传**:崩了重开,扫 `_staging/` 就知道哪些已捕捉、未落库。

## 格式

```markdown
---
source: raw/socialmedia/<作者>/<文件>.md      # 移动后的最终 raw 路径
summary: wiki/summaries/<...>.md              # 本条已写好的 summary 终位（没有则留空）
captured_at: YYYY-MM-DD HH:MM
batch: <批次标识，如 "马克的技术工作坊"，便于 integrate 归并 log>
---

## 知识贡献

# 每个落点一条；target 用带路径内链；action=update|create；type=页面类型。
- target: [[wiki/entities/gemini]]
  action: update
  type: entity
  贡献: Gemini 2.5 Pro 在 4 模型编程横评中唯一全通过（2025-04，创作者实测）。
  证据: 见本条 summary

- target: [[wiki/methods/本地部署LLM-ollama-openwebui]]
  action: create
  type: method
  贡献: <要点骨架，integrate 据此建页>
  证据: 见本条 summary

## 已蒸馏作品登记（仅创作者 entity）

- entity: [[wiki/entities/马克的技术工作坊]]
  行: | 2025-04-15 | [[wiki/summaries/马克的技术工作坊-2025-MCP终极指南]] | MCP 入门教程 |

## 数据缺口（过门槛才写，0-3 条）

- <无 / 具体，关联页>

## log 素材

- 一句话:本条做了什么（移了哪个 raw、写了哪篇 summary、提议动了哪些页）——给 integrate 归并成一条用。
```

## 给 integrate 的约定

- proposal 是**压缩结果**:integrate 按它落库时不必回读原文/summary 全文,读 proposal 即可。
- `贡献` 写的是"该往目标页补什么知识",不是改动指令的措辞;integrate 负责真正 read-modify-write 那一页时把它融进去(不是往后追加)。
- 多条 proposal 指向同一 `target` → integrate 合并成对该页的**一次**更新。
