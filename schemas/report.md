# Review Report Schema

review 模式每轮的产出物 + 全库唯一的 review 游标。**report 落盘是 review 的最后一步**：先做完所有检查修复、写完 log，最后写 report——report 存在 = 本轮完成；中途崩了就没有 report，游标自然停在上一轮，不会出现"游标推了、活没干完"的脏状态。

## 位置与命名

- 每轮报告：`review/YYYY-MM/YYYY-MM-DD-HHMM.md`（按月建子文件夹；文件名带时分，一天多跑不撞名，且按名排序即按时间排序）。
- 汇总报告（可选，月/季/年）：同文件夹下 `rollup-YYYY-MM.md`（季/年放对应结束月，`rollup-2026-Q3.md` / `rollup-2026.md`）。前缀 `rollup-` 使其不参与游标查找。
- **游标查找** = 取最新一篇常规报告的 frontmatter：`ls review/*/20*.md | sort | tail -1`（glob `20*.md` 天然排除 rollup）。库里还没有任何 report 时，问用户从哪个时间点开始。

## Frontmatter

```yaml
---
type: review-report          # rollup 用 review-rollup
reviewed_at: "2026-07-02 06:30"   # 本轮完成时间 = 下一轮的游标起点
window_from: "2026-07-01 06:30"   # = 上一篇 report 的 reviewed_at；本轮覆盖的变更区间下界
scope: "游标后变更 + 全库机械层"   # 本轮切片描述
pages_checked: 0
issues_found: 0
fixed: 0
needs_decision: 0
flags_open: []               # 本轮未处理完的旗标，滚动携带，下一轮判断层的输入
metrics:                     # 库指标快照（机械统计，供 rollup 聚合出趋势）
  wiki_pages: 0
  index_lines: 0
  log_lines: 0
  inbox_backlog: 0
  pending_open: 0
---
```

数字字段和 `metrics` 是给汇总用的：**rollup = 只读本期各 report 的 frontmatter + 挑正文要点，不回读库本身**——成本与库大小无关。

## 正文结构

1. **发现**（按严重程度，每条一行，涉及页用 `[[wiki/...]]` 可点）
2. **已处理**（低风险修复，做了什么）
3. **需要决定**（合并/删除/冲突等待用户拍板的，同步进 `pending.md`）
4. **趋势/异常**（对比上一篇 metrics 值得注意的变化；没有就写"无"）
5. **下一步**（建议下个切片）

## 硬规则

- report 只追加不修改；写错了在下一篇更正并注明。
- `log.md` 的 review 条目降为**一行**，指向本篇 report（`详见 [[review/YYYY-MM/...]]`）——详情只住 report，一处不两写。
- report 不进 `index.md`（它不是知识页）；`review/` 目录不参与 index 同步、孤儿页扫描等机械检查。
