# Wiki 规则

`wiki/` 是用户筛选过、蒸馏过、面向未来模型使用的世界知识。它不是中立百科，也不是 raw 原文仓库。

## 目录

```text
wiki/
  concept/
  entity/
  topic/
  summary/
  method/
  tip/
  github/
  comparison/
  quotes/
  scrap/
```

## 先判断

| type | 放哪里 | 判断条件 |
| --- | --- | --- |
| `concept` | `wiki/concept/` | 可复用的想法、概念、术语、框架、心智模型。回答“这是什么？” |
| `entity` | `wiki/entity/` | 人、公司、工具、模型、产品、平台、组织。 |
| `topic` | `wiki/topic/` | 正在被讨论的社会话题、争议、热点、爆火帖子引发的讨论现象。 |
| `summary` | `wiki/summary/` | 对单个重要 raw 来源的一对一总结。 |
| `method` | `wiki/method/` | 有步骤、有判断节点、可复用的流程或方法。回答“怎么做？” |
| `tip` | `wiki/tip/` | 一场景一做法，足够小、足够具体。 |
| `github` | `wiki/github/` | 可定位到 GitHub 仓库的项目。 |
| `comparison` | `wiki/comparison/` | 2 个以上对象、概念、工具或方法的横向分析。 |
| `quote` | `wiki/quotes/` | 用户觉得好的句子、表达、俚语、金句。暂不做独立 quotes 层。 |
| `scrap` | `wiki/scrap/` | 已经蒸馏但仍碎、不足以成为正式条目的内容。 |

## 关键边界

- `concept` vs `method`：如果重点是定义和理解，是 concept；如果有可执行步骤，是 method。
- `method` vs `tip`：method 可复用且有步骤；tip 是一个小场景里的一个动作。
- `topic` vs `comparison`：topic 是外部世界正在讨论什么；comparison 是你或 agent 做的结构化对比。
- `summary` vs raw：raw 是原文；summary 是 raw 的前引速览，供以后搜索知识库时先快速判断是否需要回看原文。
- `summary` vs 其他 wiki 页：summary 不是核心知识页，也不是蒸馏终点；concept/entity/method/tip 等才承载可复用知识。不要把 summary 内容换个标题重复成 concept。
- `quote`：只收真正击中用户、值得复用或保留的表达。长篇摘录仍应回到 raw 或 summary。
- `scrap`：不是垃圾桶，而是“暂时还不够成型”。审查模式定期看是否能升级。

## 蒸馏入库

- 蒸馏是把来源里的知识拆出来，汇入现有 wiki 网络；不是为单篇文章机械生成页面。
- 先从来源识别候选：概念、人物/公司/工具、方法、tip、quote、GitHub/工具项目、topic、comparison、self 信号。
- 对每个候选先搜索现有 wiki 的标题、别名、相近概念、相关主体和来源 URL。能承接到旧页时，优先更新旧页：补充新事实、例子、边界、来源、时间线和互链。
- 只有现有页面不能承接、且候选未来可能单独搜索或复用时，才新建页面。
- 控制数量：普通来源通常只写 0-3 个高价值更新/新页；密集来源或用户要求深挖时才更多。零碎小点留在 summary 或 log，不强行拆页。
- 素材简介、正文或视频说明里的配套资源、提示词、模板、工具链接、下载链接、项目仓库等，要在 summary 正文中写 `## 配套资源` 或同义章节；不要只放进 YAML `original_url` 或末尾 `## 来源`。

## 详略度

- `2025-01-01` 以前的通用知识轻写，强调用户为什么关心。
- `2025-01-01` 之后的新模型、新工具、新事件、新讨论可以写得更详细，但仍以未来复用价值为准。
- 用户明确说“轻量记录”时，只写 stub。
- 用户明确说“深度调查”时，增加多源核实和更完整分析。

## GitHub 项目

默认写：

- 仓库 URL；
- owner；
- 项目做什么；
- README 主张；
- license、语言、活跃度等明显元信息；
- 用户为什么保存它。

只有用户要求深度调查、代码审查、安全判断或实现提取时，才读取源码和关键文件。

## 关系与冲突

- 内部链接用 Obsidian 风格：`[[wiki/concept/example]]`。
- 不要让重要内部链接指向不存在页面，除非明确标为待创建。
- 冲突不调和，保留双方来源和日期。
- 过时内容不删除，标记状态并写时间线。

## 新类型规则

遇到放不进现有类型的内容：

1. 先尝试映射到现有类型。
2. 单例就近归类或放 `scrap`。
3. 同类情况反复出现，再建议新增 type/folder。
4. 新增前更新 `references/wiki.md`、`schemas/wiki.md` 和 index 规则。
