# 落库模式（integrate）

仅由编排者在并行蒸馏时调用:所有 `capture-only` 子 agent 回来后,跑**一次**,把它们的 proposal 统一落库。单条/`standalone` 蒸馏不需要它(那时蒸馏 agent 已自己直接写)。

核心:**唯一写手、串行**。所有对共享页(entity/concept/method/index/log)的写入都在这一步、由这一个流程完成,所以不存在并发覆盖、不存在"读完又被改"。

## Step 1：收集 proposal

读取库根目录 `_staging/` 下全部 `*.md`(每个是一条已捕捉素材的 proposal,格式见 `schemas/proposal.md`)。只读 proposal,**不回读 summary 全文/原文**——proposal 已是压缩结果。

## Step 2：按目标页归并（reduce）

把所有 proposal 的"知识贡献"按 `target` 分组——reduce 的 key 是目标页:

- 同一个 entity/concept/method 被多条贡献 → **合并成对这一页的一次更新**,不逐条改 N 次。
- 同一个"待新建"页被多条提议 → **建一页**,融合各条贡献,不重复建。
- 创作者 entity 的"已蒸馏作品登记"按 entity 分组,**一次性把多行按日期排序补进表格**;**不写时间线/版本里程碑**(见 `schemas/wiki.md`)。真有里程碑(首入库/方向转变/量级跨越)就在 `快照`/`内容定位` 一句话带过。

## Step 3：写共享页（唯一写手，逐页 read-modify-write）

读具体 schema(`schemas/common.md` + `schemas/wiki.md`):

- **真正读旧页再改**,把合并后的贡献融进去,不是往页尾追加新段(这是旧版"两个时间线/重复段"的根源)。
- **页面已有遗留的「编辑日志」式 `## 时间线` / `## 版本里程碑` 段时,绝不往里追加新条目**——跟着旧结构走是实测踩过的坑;把贡献融进正文对应小节,遗留段留给 review 机械层清理。
- 先更新已有页,再创建新页,新旧互链。
- 冲突保留双方来源和日期,不静默覆盖。

## Step 4：更新 index.md

新建的 concept/entity/topic/method/tip/github/comparison/quote 各补一行(summary 不进 index)。机械操作,可脚本生成/校验(见 `schemas/root.md` 与 `modes/review.md` Step 4)。

## Step 5：写一条合并 log + git

读 `schemas/root.md` 的 log 格式,把本批 proposal 的"log 素材"**归并成一条**(压缩、不是拼接):先写"本批"总况(几条 / raw 移了几条 / summaries / 重复 / 空),再列**新建页**和**更新页**——**每页一行、页名用 `[[wiki/...]]` 可点、跟一句话概括**;pending 新增也列。**时间戳用本机本地时间**(见 `schemas/root.md` 时间戳规则,别用沙箱 UTC),标题保留 `## [YYYY-MM-DD HH:MM] ingest · … — <标题>`。写完 `git add -A && git commit`(一批一个快照,不 push)。

## Step 6：pending + 清理

- **先清 pending 打勾条目**:写新缺口前,用 blind 脚本删掉 `pending.md` 里所有 `- [x]` 已勾选条目(含其下缩进子说明)——纯机械、不等 review。这样用户在上一批 pending 里勾掉的,这一批落库时就清了,pending 不会越堆越长。用计数/diff 验证(`grep -c '\[x\]'`),不必读全文。
- 各 proposal 里过门槛的数据缺口 → 追加进 `pending.md`(带 `- [ ]`,数量克制)。
- **删除已消费的 `_staging/*.md`**——它们已落库,使命完成(断点信息已进 log)。

## 输出

批次报告:落库了几条 proposal、更新/新建了哪些页、合并成几条 log、哪些进 pending、有无冲突或低置信度。
