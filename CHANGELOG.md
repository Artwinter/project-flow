# Changelog

## [1.0.0] - 2026-11-05

首个公开版本。

### 新增

- `project-flow` 技能本体：约 100 行的立项流程路由层
- 三个门禁（G1 共识确认 / G2 接缝确认 / G3 每票清上下文）
- 反模式清单与触发词映射
- `docs/walkthrough.md`：九幕完整实例（从"我有个想法"到 PR）
- `docs/design-notes.md`：设计取舍与已知局限
- `docs/install.md`：Claude Code / DSH / 手动 三种安装方式
- `examples/AGENTS.md-snippet.md`：提高触发命中率的兜底片段

### 依赖

- [mattpocock/skills](https://github.com/mattpocock/skills) 的
  grilling / grill-me / grill-with-docs / domain-modeling / setup-matt-pocock-skills /
  to-spec / to-tickets / implement / tdd / code-review / prototype / retro

### 已知问题

- DSH 需手动删除下游技能的 `disable-model-invocation: true` 才能使用（见 install.md）
- 未做下游技能存在性检查
