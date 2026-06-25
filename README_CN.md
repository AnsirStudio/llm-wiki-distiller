# second-me-wiki-distiller

> 构建并维护一个**由 LLM 维护的个人 Markdown 知识库**——不是中立百科，而是一个 **second me（第二个我）的种子**：一个怕忘可查的备忘录、一份可复用的知识资产，以及一个未来模型能用你的知识 / 品味 / 思考方式来推理的表示。
> 一句话：**写入（快速 / 蒸馏）+ 维护（审查）+ 查问（检索）**，四个模式管住全生命周期，并且**按步骤逐步加载资源**——agent 只读它当下真正需要的规则。

🌐 English → [`README.md`](./README.md)

---

## 它是什么

`second-me-wiki-distiller` 是一个**自包含、可独立分享的 skill**（操作手册）。它教 agent 如何把零散素材——笔记、网页剪藏、文章、聊天、社交媒体、书籍、研究、截图、GitHub 项目、对话——**预编译**成结构化、互链、可溯源的 markdown，沉淀为**三层**：

- **`raw/`** —— 原始素材层：原始材料、附件、待处理 inbox。唯一信源，尽量保持原貌。
- **`wiki/`** —— 由 LLM 维护的蒸馏层：**你筛选过**的那一片世界知识，为未来模型复用而编译。**不是**中立百科，**也不是**原文的副本。
- **`self/`** —— 关于你本人的慢变量表示。这是 **second me 的种子**：你是谁、你看重什么、你怎么想。

重点不是重写模型已经会的百科式通用知识，而是保留**你的视角、你的来源路径、训练截止之后的新信息，以及可复用的个人上下文。**

> **第一性原理——这不是中立百科，是「你自己的」。** 它承担三件事：① **备忘录**（怕忘的、好查）；② **知识资产**（你的积累、可复用）；③ **second me**（积累足够后，模型能用你的知识 + 风格 + 思考方式替你做粗判断）。

> **分工（沿用 Karpathy）。** 人只负责**四件事**：挑选来源、引导分析、问对问题、思考这一切意味着什么。**其余全是 AI 的活**——摘要、互链、归档、记账、保持各页一致。你不亲手维护这个库，也不逐条审核；正确性靠**可溯源**（每条事实都链回出处）和**定期 lint**（按计划体检矛盾、过期、断链、孤页）兜底，而不是逐行人审。人会弃坑 wiki，是因为维护成本涨得比价值快——这里维护成本由 AI 扛,所以接近于零。

### 设计速览

- **三层** —— `raw/`（素材）→ `wiki/`（编译后的知识）→ `self/`（second me 种子）。
- **单一根 index。** 用一个 `index.md`（加 `log.md` / `pending.md`）做导航。
- **规则与格式分家。** `references/` 放*方法*（怎么判断、放哪里）；`schemas/` 放*格式*（YAML frontmatter、受控词表、正文结构）。
- **渐进式披露（progressive disclosure）内建。** `SKILL.md` 只负责路由；每个 mode **逐步**读取自己的 references 和 schemas，只在需要那条规则的那一步才加载对应文件，绝不一开始就把整本手册灌进上下文。
- **受控词表 YAML，不再逐条人审。** 页面带 Obsidian / Dataview 友好的 frontmatter（`status`、`confidence`、`depth`、`volatility`…）。`wiki/` 与根页面**不带**审查状态字段——AI 写入即可用，质量靠可溯源的来源和定期 lint，不逐条签字。只有 `self/` 页保留一个轻量 `verified` 打钩（默认 `false`，用户打钩成 `true`），因为关于「你自己」的事实错了代价最大。

## 做什么 / 不做什么

- ✅ **快速** —— 不碰 inbox 的直接写入：新增、修改、归档、删除一个 `wiki/`/`self/` 条目，或把一个小知识点快速记入库。
- ✅ **蒸馏（写）** —— 处理 `raw/inbox/`（及其他 raw 素材），每条过 raw 规范化 → wiki 候选 → self 信号 → quote，写可溯源的页。
- ✅ **审查（维护）** —— 扫现有库：矛盾、lint、时效、断链、孤儿、去重、`scrap`/`dropzone` 聚类，以及 **self 升维**（把 log 里潜伏的注意力信号聚合成稳定结论）。
- ✅ **检索（问）** —— 直接读已编译好的页面回答「我是不是记过 X」「库里有没有 Y」，引用来源页；不写、不动 `log.md`、不编。
- ❌ **不感知编排** —— 不附带 `AGENTS.md`/`CLAUDE.md`，也不知道谁调度它；多 agent 批量是更高层的事。

## 模式门（永远是第一步）

每次调用**先定模式**，按提示词语义判定，不需要用户说出模式名。共有**四个模式**——快速、蒸馏、审查、检索——覆盖从「快速补一条」到「全自动批处理」再到「维护现有库」再到「查问已有内容」。选择逻辑见 `SKILL.md`；模式门之前，先有一道轻量的 **Step 0 根目录检查**，确认当前工作目录确实是知识库（需命中 ≥2 类信号：`raw/`、`wiki/`、`self/`、根目录文件）才动手。

## 渐进式披露（核心设计）

`SKILL.md` 刻意写薄：核心想法 + 根目录检查 + 模式门 + 少数共享硬规则 + 资源地图。它**只负责路由**。详细方法在 `modes/`、`references/`、`schemas/` 里，**惰性加载**：

```
SKILL.md  →  modes/<模式>.md  →  （一步一步）references/* 然后 schemas/*
```

每个 mode 是带编号的步骤序列，每一步都写明*在这一步*该读哪个规则文件——例如蒸馏只在认领素材时读 `references/raw.md`，只在出现 wiki 候选时读 `references/wiki.md`，只在真正写入那一刻才读具体的 `schemas/*.md`。agent 从不在开头读完整本手册，上下文保持精简、决策保持局部。

## 弱信号机制

为「self 为核心」专门设计的一个机制。**关于「你」的明显信号直接进 `self/`；但还有一类*弱*信号——你反复选择去蒸馏的东西，本身就是关于你注意力和兴趣的微弱证据。** 若每条都写进 `self/`，会把它灌爆、还和 log 重复，所以它们被**留在 `log.md` 里**（log 带轻量 `topic` 标签）潜伏着；由**审查模式**日后按 `topic × 时间窗`聚合成结论、升级进身份层——把一堆例行动作变成一条真正关于「你」的观察，零重复、不灌爆。

## 核心规则（速览）

1. **溯源不编造** —— 每条事实链回来源；缺失就留空，别用 `未知` 填满。同一事实全库单一出处。
2. **仅 self 轻量确认** —— `self/` 页带一个 `verified` 打钩（AI 一律保持 `false`，只有用户打钩成 `true`）；`wiki/`/根页面无审查门，写入即信任，靠来源 + lint 兜底。
3. **写前搜索** —— 按来源 / 相似标题 / 别名 / URL / 核心句子去重。
4. **绝对日期** —— 持久化事实绝不只写「昨天 / 今天 / 最近」，必带 `YYYY-MM-DD`。
5. **矛盾保留** —— 冲突时双方说法、来源、日期都留，标记冲突，绝不静默覆盖。
6. **不删只迭代** —— 归档 / 取代 / 弃用，保留历史；物理删除前必须确认。
7. **别过度推断** —— 证据薄的性格 / 偏好判断标 `confidence: low` 或先进 `pending.md`；不从弱证据推断受保护特征、医疗 / 法律 / 财务状况。
8. **wiki 是编译后的记忆，不是原文仓库** —— 保留 raw 路径，不把完整原文搬进 `wiki/`。
9. **沿时间线校准详略** —— `2025-01-01` 以前的知识默认轻写（除非它对你重要）；越新的知识（补训练截止后的 gap）越值得写细，但以*有用*而非完整为准。

## 它搭建的目录（幂等自建，首次运行先判断在哪建）

缺什么补什么，已存在跳过；判断「已存在」看 ≥2 类信号（`raw/`、`wiki/`、`self/`、或根目录文件命中 ≥2 个），不是撞上一个同名文件就认。命中不够、用户又没指过库路径 → 先停下来问，不会自己瞎猜目录。

```
<项目根>/
├── index.md  log.md  pending.md           # 单一根导航 + 只追加日志 + 待办项
├── raw/    # 唯一信源，尽量保持原貌
│   ├── inbox/        # 待处理入口（inbox/clipping/ 放 clipper 自动保存内容）
│   ├── attachment/   # 二进制（PDF/图片/视频/PPT/音频），每个都被一个 raw .md 指针引用
│   └── articles/ books/ chats/ design/ ideas/ research/ videos/ work/ socialmedia/ dropzone/
├── wiki/   # concept/ entity/ topic/ summary/ method/ tip/ github/ comparison/ quotes/ scrap/
└── self/   # observations.md identity.md preferences.md now.md profile.md（无单独 index，五文件即入口）
```

> 内部链接用 Obsidian 风格、带文件夹前缀：`[[wiki/concept/example]]`、`[[self/identity]]`。非 markdown 素材绝不孤立存在——放进 `raw/attachment/`，并配一个最小 markdown 指针文件，由 wiki 页引用。

## 怎么用

把素材丢进 `raw/inbox/`，再告诉 agent 你想怎么处理（「把这些蒸进库」「review 下数据库」「记一下这个人的主页」「我之前是不是记过 X」）。它会先做根目录检查、按语义定模式、只加载这一步需要的规则、必要时把条目从 inbox 认领出来、每条过三层、写入页面，并更新 `index.md` / `log.md` / `pending.md`；检索类问题则直接读库作答，不落盘任何改动。完整方法见 `SKILL.md`。

> 多说一句：这个 skill 会跟着作者自己的实际使用持续进化。如果某个流程 / prompt / 护栏跟你的习惯不合拍，欢迎直接 fork 改成适合自己的版本——后续更新方向主要跟着作者自己的使用场景走，未必每次都贴合所有人。

## 文件

| 文件 | 作用 |
|------|------|
| `SKILL.md` | 入口（刻意写薄）：核心想法 + Step 0 根目录检查 + 模式门 + 共享硬规则 + 资源地图。只负责路由。 |
| `modes/{quick,distill,review,search}.md` | 四模式逐步行为规范，每步惰性加载 references/schemas。 |
| `references/raw.md` | raw/ 方法：类目、附件、指针文件、去重。 |
| `references/wiki.md` | wiki/ 方法：页面类型判断、边界、详略度、GitHub 深读、冲突。 |
| `references/self.md` | **核心** —— self 五件套、证据等级、抽取维度、second me 画像、敏感信息。 |
| `schemas/common.md` | 通用 YAML frontmatter + 受控词表 + 日期规则。 |
| `schemas/raw.md` | raw 指针格式、来源 metadata、附件引用。 |
| `schemas/wiki.md` | wiki 各类型页面的 frontmatter 与正文结构。 |
| `schemas/self.md` | self 五件套 schema。 |
| `schemas/root.md` | `index.md` / `log.md` / `pending.md` 格式。 |

## 渊源与致谢

架构借鉴：

- **Andrej Karpathy「LLM Wiki」**（gist）：<https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f> —— 把知识预编译进互链 markdown，原始素材不可变 + LLM 维护层 + 规则 schema。
- **honcho 项目**（Plastic Labs 开源个人记忆层）：<https://github.com/plastic-labs/honcho> —— 后台蒸馏个人原子事实、保留矛盾、新值取代旧值、证据薄就弃权。

## 独立性

本 skill **自包含、可独立使用**——不依赖任何外部编排文件、也不依赖任何其它 skill。把整个 `second-me-wiki-distiller/` 目录拷走即可复用或分享。
