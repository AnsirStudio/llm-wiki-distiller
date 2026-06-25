# 通用 Schema

所有 wiki/self/root 页面尽量使用稳定 YAML，方便 agent、Obsidian 和 Dataview 查询。

## 通用 Frontmatter

```yaml
---
title: ""
tldr: ""
type: concept
status: current
created: 2026-06-23
updated: 2026-06-23
tags: []
aliases: []
original_url: []
related: []
confidence: medium
depth: standard
volatility: evergreen
valid_as_of:
review_by:
---
```

> wiki/ 与 root 页面不带审查状态字段——AI 写入即可用，质量靠溯源、`## 来源` 引用和审查模式的定期 lint 兜底，不逐条人工核。仅 `self/` 页保留一个轻量 `verified` 打钩，见 `schemas/self.md`。

## 受控词表

- `type`：见 `schemas/wiki.md`、`schemas/self.md`、`schemas/root.md`。
- `status`：`current`, `needs-review`, `superseded`, `deprecated`, `archived`。
- `confidence`：`high`, `medium`, `low`, `uncertain`。
- `depth`：`light`, `standard`, `deep`。
- `volatility`：`evergreen`, `volatile`。

## 字段说明

- `title`：页面标题。
- `tldr`：一句话摘要，供 index 扫描。
- `tags`：粗粒度分类标签，用于按领域/身份筛选，不是关键词桶。只放该页面**稳定不变**的领域/身份维度（例如：创作者、抖音、AI工具评测、独立开发），**最多 8 个**；超过 8 个说明粒度太细，该精简。具体工具名、概念名、单次提到的术语**不进 tags**——它们应该用 `related` 互链到对应 concept/entity 页面，检索模式做的是全文检索而不是只读 tags，省略它们不影响以后搜索。更新页面时不要因为新一条来源带来新名词就追加 tag，先问"这是不是这个主体本身的稳定维度"。
- `original_url`：只放核心来源，用于快速读取和证据定位；通常 1-3 条，复杂页面最多 5 条。优先放直接支撑页面主要结论的 raw 路径、summary 页、原始 URL 或对话来源。不要把搜索时看过的资料、背景链接、相关但非核心的来源全部塞进 YAML。
- `related`：内部相关页面。
- `valid_as_of`：仅 volatile 页面必填。
- `review_by`：仅 volatile 页面建议填写，通常 1-6 个月后。

## 日期

所有持久化日期用 `YYYY-MM-DD`。不要只写“今天”“昨天”“最近”。

## original_url 护栏

- YAML `original_url` 是快速索引，不是完整 bibliography。
- wiki/self 页面不要使用 `sources` 字段；需要来源索引时统一写 `original_url`。
- 只收能支撑页面主结论、身份判断、关键事实或 self 信号的核心来源。
- 搜索补充、背景资料、延伸阅读、同主题但未直接支撑结论的链接，放正文末尾 `## 来源`，并用简短说明标注用途。
- 如果来源很多，正文 `## 来源` 可以分为“核心来源”和“补充资料”；YAML `original_url` 仍只保留核心来源。
- **创作者/博主类 entity（每次更新对应一条新视频/帖子）**：`original_url` 只放该创作者的主页类链接（平台主页、官网、GitHub 主页等），**不放任何单条作品/视频/帖子链接**。单条来源链接只属于对应 `wiki/summary/` 页面自己的 `original_url`；entity 页面靠正文“已蒸馏作品”表格链接到这些 summary 页，不需要在自己 YAML 里重复收链接。
