# 实例示范：九幕走完一次立项

> 场景：你已有一个素材库项目，现在要加"审片批注"。
> 下面每一幕都标注了 **谁在说话 / 调了哪个技能 / 为什么这么做**。

---

## 第 0 幕｜你只说一句话

> **你**：我想给我的素材库加个审片批注功能

---

## 第 1 幕｜路由

**【AI】** 调 `project-flow`，输出：

> 这是 **② 已有仓库里的新功能**。
> 路线：`grill-with-docs` → 判断单/多会话 → 分流。
> 开始盘问，我一次给一整轮。

**【AI】** 调 `grill-with-docs`。

> 💡 注意它**没有**开始读代码。规则第一条：需求模糊就不动手。

---

## 第 2 幕｜第一轮盘问

```
❓ Q1 - 批注的载体：绑在"时间码区间"上，还是绑在"整段素材"上？
       精度要到帧还是毫秒？

➡️ 建议：时间码区间 + 帧精度。审片场景里 "其实说的是 00:03:12 到 00:03:45"
   很常见，秒级精度会在评议时引发分歧。

---

❓ Q2 - 参与者身份：外部客户能不能看？看完能不能批注？

➡️ 建议：先只做内部。外链分享涉及权限、水印、访问审计，会让范围翻三倍。

---

❓ Q3 - 批注的角色语义：一条批注是"问题"还是"意见"？
       要不要状态机（待处理/已改/已确认）？

➡️ 建议：先只有一条状态：未解决/已解决。审片真正的痛点是"漏掉没改的"。

---

❓ Q4 - 批注和版本的关系：新版本上传后，旧批注跟着走还是作废？

➡️ 建议：跟着时间码走，标注"此批注基于 v2"。作废会让评议历史断代。
```

**【你】**：

> Q1 同意，但我要**毫秒**不是帧——我们素材有 60fps 也有 25fps，帧不准。
> Q2 同意内部。
> Q3 这条我不同意。我们甲方会看，他提的一律是"问题"，我们自己提的是"意见"——两种要分开。
> Q4 同意。

> 💡 **这就是"它在起作用"的信号：你反驳了两条。**
> 如果全盘"同意"，这场盘问就是白开——"连答 40 个 agreed 得到 AI 写的 plan"
> 正是这个失败模式。

---

## 第 3 幕｜当场落纸

**【AI】** 立刻写 `GLOSSARY.md`（不是最后一次性生成）：

```markdown
# 素材库

集中管理视频素材与审片流程。

## Language

**Note**:
A single remark pinned to a time range of an Asset.
Pinned at millisecond precision, because source footage ranges 25–60 fps.
_Avoid_: comment, feedback, annotation, mark

**Issue**:
A Note raised by an external reviewer. Distinguished from an internal Opinion.
_Avoid_: bug, ticket, problem

**Opinion**:
A Note raised by an internal team member.
_Avoid_: feedback, suggestion

**Resolution**:
The state of a Note: unresolved or resolved. Only two states.
_Avoid_: status, workflow state, open/closed
```

三个设计点：

1. `_Avoid_` 把 `comment` / `feedback` / `annotation` 全禁了 —— 以后代码里不会出现四个名字指一个东西
2. **`Issue` 和 `Opinion` 分开了** —— 这是 Q3 反驳的直接产物
3. 毫秒精度写进了定义 —— 这是 Q1 的产物

**【AI】** 判断要不要写 ADR：Q4"批注跟时间码走"通过三闸门
（难逆转 ✅ / 不看背景会惊讶 ✅ / 真实权衡 ✅）→ 写 `docs/adr/0007-note-follows-timecode.md`。

Q1 的"毫秒"只有一句话、改起来也便宜 → **只进术语表，不写 ADR**。

---

## 第 4 幕｜岔路：出现"必须看到才能答"的问题

**【AI】**：

```
❓ Q5 - 批注在播放器时间轴上的呈现：标记条、悬浮气泡、还是右侧列表联动高亮？

➡️ 我无法用文字回答——它取决于"看画面时眼睛愿不愿意离开画面"。
   这是 ungrillable 的问题，我不该继续问。

建议：停一下，用 prototype 出三种排布，你看一眼再定。
```

**【AI】** `handoff`（出去）→ 新会话 → `prototype` → `handoff`（回来）

**【回来】**：

> **你**：第二种。列表在右侧，点了高亮画面，但气泡不要盖住画面中心。

> 💡 一条决策，两分钟，比再问十轮值。

---

## 第 5 幕｜前沿空了，停下等你（G1）

**【AI】**：

```
前沿已空。已定的决策：
1. 批注绑毫秒级时间码区间
2. 只做内部 + 外部 reviewer 两种角色
3. Issue / Opinion 语义分开，状态只有 未解决/已解决
4. 批注跟时间码走，标注所基于的版本
5. 时间轴排布取原型第二种

我们达成共识了吗？确认后我再往下走。
```

**【你】**：达成共识，往下。

> 💡 **这就是 G1。** AI 不能替你宣布。

---

## 第 6 幕｜分流判断

**【AI】**：

> 这活儿能不能一个上下文窗口做完？
> 不能。批注时间码模型 + 播放器联动 UI + 两种角色权限 + 版本迁移，至少 4 个独立层。
> → 走 `to-spec` → `to-tickets`。

**【AI】** 调 `to-spec`（**同一个会话，不清空**）。

### 6.1 先给接缝（G2）

```
打算在这 1 个接缝上测试：

  NotesService — 对外入口：
    addNote(assetId, range, body, authorRole)
    listNotes(assetId, version)

  理由：复用现有素材库的 service 测试基座；这是最高层；
        整个改动只需要这一个接缝。
  不打算在时间码解析、播放器组件、ORM 层各开接缝。

这个接缝符合你的预期吗？
```

**【你】**：符合，但我想单独测一下"跨版本迁移"那个边界。

**【AI】**：好，那就 **2 个接缝**：`NotesService`（主）+ `NoteMigration`（版本迁移边界）。

### 6.2 写 spec（节选）

```markdown
## Problem Statement
审片意见散落在微信和口头，改没改漏没漏无法追溯，客户和内部的意见混在一起。

## Solution
在素材播放器上按毫秒时间码区间打 Note；外部 reviewer 的 Note 记为 Issue，
内部记为 Opinion；Note 跟时间码走并标注所基于的版本。

## User Stories
1. As a 审片人, I want to 在播放到某帧时一键打点, so that 我不用记时间码
2. As a 导演, I want to 看到某条 Note 对应画面的前后 3 秒, so that 我知道改什么
3. As a 客户, I want to 我的意见和内部意见分开显示, so that 我不用分辨哪条是我提的
4. As a 制片, I want to 看到未解决 Note 的数量, so that 我知道能不能过审
5. As a 制片, I want to 新版本上传后旧 Note 仍在, so that 跨版本评议不断代
...（这一节要极长）

## Implementation Decisions
- 新增 Note 模块：timeRange(startMs, endMs) + body + authorRole + resolution
- 角色两种：internal / external；external 的 Note 记为 Issue
- 版本迁移：Note 携带 basedOnVersion，迁移时按时间码重挂，不复制
- 状态只有 unresolved / resolved，不做工作流
- 播放器联动走订阅式高亮，不改现有播放器内核
（不写文件路径、不写代码片段）

## Testing Decisions
- 好测试：只断言 NotesService 返回的 Note 集合与字段，不断言内部调用
- 接缝：NotesService（主）、NoteMigration（跨版本边界）
- prior art：现有 AssetService 的集成测试写法

## Out of Scope
- 外链分享与访问审计
- 批注回复的线程
- 批注导出 PDF
- 移动端

## Further Notes
时间轴排布取原型第二种（右侧列表联动高亮，气泡避开画面中心）。
```

---

## 第 7 幕｜切票（先核对，不擅自发布）

**【AI】** 调 `to-tickets`，**先给清单**：

```
1. Note 数据模型与时间码区间
   Blocked by: —
   Delivers: 能存取一条 Note，尚未有 UI

2. NotesService 入口 + 角色语义（Issue/Opinion）
   Blocked by: 1
   Delivers: 两种角色的 Note 能分开存取（主接缝）

3. 播放器上的打点与高亮（原型第二种排布）
   Blocked by: 2
   Delivers: 边看边打点，端到端可演示

4. 跨版本迁移（第二接缝）
   Blocked by: 1
   Delivers: 上传 v3 后旧 Note 仍在且挂对位置

5. 未解决计数与列表筛选
   Blocked by: 3
   Delivers: 制片能一眼看到还差几条
```

**【你】**：3 和 5 是不是可以并？我要先能演示给客户看。

**【AI】**：拆成两票是因为 5 依赖 3 的列表组件。如果要的是"能演示"，3 就够了。

**【你】**：那就这样。

**【AI】** 发布 → 打 `ready-for-agent` → **立刻摘掉父 spec 的标签**。

> ⚠️ 这一步最容易被忘。忘了，自动 agent 会把整份 spec 一次干完。

---

## 第 8 幕｜逐票实现

```
新会话 → /implement 1 → 绿 + code-review + commit → 清
新会话 → /implement 2 → 绿 + code-review + commit → 清
新会话 → /implement 3 → 绿 + code-review + commit → 清
...
```

每票固定五步：读票定接缝 → 在**事先约定的接缝**上 TDD → 频繁 typecheck →
跑一次全量 → code-review + commit。

> 💡 `implement` **不会重开计划**，不会访谈你，不会提议换个做法。
> 这是它和"跟我说 build this"的根本区别。

---

## 第 9 幕｜收尾

```
/pr      → PR body（最小可视化 + before/after 证据 + 单向/双向门）
/retro   → 回看这次，建议改的是环境不是代码
```

---

## 附：四种体量对照

| 你的情况 | 路线 | 耗时量级 |
|---|---|---|
| 改个 bug / 加个小设置 | 不用 project-flow，直接做 | 分钟 |
| 一个新功能，一个窗口做得完 | project-flow → grill-with-docs → implement | 1–2 小时 |
| 跨 3 个以上模块 / 要分几次做 | → grill-with-docs → to-spec → to-tickets → 逐票 implement | 1–3 天 |
| 连要不要做都没定 | → grill-me（结论可能是"不做"） | 30 分钟 |
