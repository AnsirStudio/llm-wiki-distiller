# Self Schema

适用于 `self/` 五件套。所有页面继承 `schemas/common.md`，`type` 使用 self 类型。

## Types

`self-observations`, `self-identity`, `self-preferences`, `self-now`, `self-profile`

## verified 字段（仅 self 页）

self 是 second me 的种子，关于「你自己」的事实一旦写错会污染下游所有推断，所以这一层保留一个轻量的人工确认开关——但只是打钩，不是逐条强制核。

在 self 页 frontmatter 里加一行：

```yaml
verified: false
```

- 受控词表：`false`（默认，AI 草稿）、`true`（用户已确认）。
- AI 新建或改动 self 页时一律写/保持 `verified: false`，**绝不**自己改成 `true`。
- 只有用户亲自确认后把它打钩成 `verified: true`。
- 不强制——你大可让大部分条目长期停在 `false`，只给真正在意的核心身份事实打钩。
- wiki/ 与 root 页不带这个字段。

## observations.md

```markdown
# 观察

## 事实流

- `YYYY-MM-DD` | 明确/强推断/弱信号 | 观察内容。证据：`来源`。

## 时间线

- `YYYY-MM-DD`：更新说明。
```

规则：只追加，不重写旧观察。

## identity.md

```markdown
# 身份

## 稳定身份

## 价值观与原则

## 思维方式与决策依据

## 待确认问题

## 时间线
```

只收稳定、可复用、对 second me 有帮助的内容。

## preferences.md

```markdown
# 偏好

## 沟通

## 工具与工作流

## 学习

## 审美与风格

## 明确的不要

## 时间线
```

每条偏好说明适用场景和证据。

## now.md

```markdown
# 当前

## 当前项目

## 活跃问题

## 近期目标

## 曾经在忙

## 时间线
```

旧事项移到历史区，不删除。

## profile.md

控制在 500 token 以内。

```markdown
# 个人画像

用户是……

用户重视……

用户偏好……

截至 YYYY-MM-DD，当前关注……

使用这个 profile 时要注意……

## 时间线
```

只有 identity/preferences/now 有实质更新，或审查模式触发时，才重新生成。
