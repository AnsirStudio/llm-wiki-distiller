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
review_status: ai-draft
---
```

## 受控词表

- `type`：见 `schemas/wiki.md`、`schemas/self.md`、`schemas/root.md`。
- `status`：`current`, `needs-review`, `superseded`, `deprecated`, `archived`。
- `confidence`：`high`, `medium`, `low`, `uncertain`。
- `depth`：`light`, `standard`, `deep`。
- `volatility`：`evergreen`, `volatile`。
- `review_status`：`ai-draft`, `user-reviewed`。

## 字段说明

- `title`：页面标题。
- `tldr`：一句话摘要，供 index 扫描。
- `original_url`：只放核心来源，用于快速读取和证据定位；通常 1-3 条，复杂页面最多 5 条。优先放直接支撑页面主要结论的 raw 路径、summary 页、原始 URL 或对话来源。不要把搜索时看过的资料、背景链接、相关但非核心的来源全部塞进 YAML。
- `related`：内部相关页面。
- `valid_as_of`：仅 volatile 页面必填。
- `review_by`：仅 volatile 页面建议填写，通常 1-6 个月后。
- `review_status`：AI 新建或改动后保持 `ai-draft`；只有用户确认后改 `user-reviewed`。

## 日期

所有持久化日期用 `YYYY-MM-DD`。不要只写“今天”“昨天”“最近”。

## original_url 护栏

- YAML `original_url` 是快速索引，不是完整 bibliography。
- wiki/self 页面不要使用 `sources` 字段；需要来源索引时统一写 `original_url`。
- 只收能支撑页面主结论、身份判断、关键事实或 self 信号的核心来源。
- 搜索补充、背景资料、延伸阅读、同主题但未直接支撑结论的链接，放正文末尾 `## 来源`，并用简短说明标注用途。
- 如果来源很多，正文 `## 来源` 可以分为“核心来源”和“补充资料”；YAML `original_url` 仍只保留核心来源。
