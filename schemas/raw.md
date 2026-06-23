# Raw Schema

raw 文件不强制全部使用 YAML。剪藏工具、外部整理材料、AI 生成资料可能自带 metadata；很小的片段也可以没有 frontmatter。需要 agent 新建或规范化 raw markdown 时，使用本文件。

## 最小 Frontmatter

```yaml
---
title: ""
raw_type: article
created: 2026-06-23
source_date:
author:
creator:
platform:
original_url:
attachment: []
status: unprocessed
---
```

## 受控值

- `raw_type`：`article`, `book`, `chat`, `design`, `idea`, `research`, `video`, `work`, `socialmedia`, `dropzone`, `attachment-pointer`, `unknown`。
- `status`：`unprocessed`, `claimed`, `distilled`, `duplicate`, `archived`。

## 字段说明

- `title`：来源标题。未知时用文件名推断，不要编造。
- `source_date`：素材发布或产生日期；不知道就留空。
- `created`：进入知识库的日期。
- `author`：作者、创作者、对话参与者或机构；不知道就留空。
- `creator`：社媒创作者名；`raw_type: socialmedia` 时必填，且必须等于 `raw/socialmedia/{创作者名}/` 的父文件夹名。
- `platform`：YouTube、Douyin、X、博客、PDF、微信等。
- `original_url`：原始链接；没有就留空。
- `attachment`：附件路径数组，例如 `["raw/attachment/report.pdf"]`。
- `status`：处理状态。

## Markdown 正文

`raw_type: socialmedia` 的文件路径必须是 `raw/socialmedia/{创作者名}/{platform}-{年}-{主题}.md`，按人归档，不按平台归档；同一创作者的多平台内容放在同一个创作者文件夹下。

新建 raw 指针文件时用这个最小结构：

```markdown
# 标题

## 来源

- 原始链接：
- 作者/平台：
- 日期：
- 附件：![[raw/attachment/file.pdf]]

## 内容

这里放原文、摘录、OCR、转录、或对附件的最小描述。

## 处理记录

- `YYYY-MM-DD`：进入 inbox / 创建附件指针 / 已蒸馏。
```

## 非 Markdown 附件指针

PDF、图片、视频、PPT、音频等文件放入 `raw/attachment/` 后，必须创建一个 markdown 指针文件。指针文件的 `raw_type` 可使用原始语义类型，例如 PDF 文章用 `article`，截图用 `design` 或 `dropzone`；无法判断时用 `attachment-pointer`。

不要把附件路径只放在 `wiki/` 页面里。先有 raw 指针，再从 wiki summary 或条目链接回 raw 指针。

## 宽松原则

- 已存在 raw 文件有自己的 frontmatter 时，不强行重写。
- 小片段可以只有标题和正文。
- 缺失字段留空，不用 `未知` 填满。
- 蒸馏结果不要写回 raw；写到 `wiki/` 或 `self/`。
