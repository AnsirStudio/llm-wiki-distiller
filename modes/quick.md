# 快速模式

用于不经过 `raw/inbox/` 的直接写入：新增、修改、归档、删除 wiki/self 条目，或把一个小知识点快速记入库。

## Step 1：判定对象与操作

不需要先读 schema。只根据用户请求判断：

1. 可能落点：`wiki/`、`self/`，或两者都要。
2. 操作：新增、修改、归档、删除。
3. 是否真的不需要 raw/inbox。

如果发现其实需要处理完整 raw 素材、长文、聊天记录、PDF、视频转录或 inbox 文件，停止快速模式，转 `modes/distill.md`。

## Step 2：读取落点规则

按需要读取：

- 涉及知识、概念、实体、方法、tip、GitHub、quote、scrap 时，读 `references/wiki.md`。
- 涉及用户身份、偏好、当前事项、个人画像时，读 `references/self.md`。

然后判断具体页面类型或 self 文件。

## Step 3：写前检查

不需要额外 schema，先做搜索：

- 搜索已有页面、别名、同义词、来源 URL、核心句子。
- 如果是当前事实或快变对象，先核对最新来源。
- 如果是用户本人信息，区分明确事实、强推断、弱信号。
- 如果是删除请求，确认是否可以归档替代；物理删除前问用户。

## Step 4：读取 schema 并写入

到写入时再读取：

- 通用字段：`schemas/common.md`
- wiki 页面：`schemas/wiki.md`
- self 页面：`schemas/self.md`
- `index.md`、`log.md`、`pending.md`：`schemas/root.md`

写入规则：

- 新增：按 schema 创建页面，补来源、关联、时间线。
- 修改：保留历史，追加新证据或版本说明，不静默覆盖旧结论。
- 归档：改状态并在时间线说明原因。
- 删除：用户确认后再删，并清理入链、index 和 log。

## Step 5：收尾检查

- YAML 字段完整，受控词表合法。
- `updated` 日期已更新。
- 重要页面有 `## 时间线`。
- `index.md` 已同步。
- `log.md` 已记录。
- 未确认、缺口或后续动作已写入 `pending.md`。

## 输出

简短告诉用户改了哪些文件、哪些内容仍需确认。不要输出长报告。
