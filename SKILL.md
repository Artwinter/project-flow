---
name: project-flow
description: 立项流程总控。当用户说"我有个想法""我想做个新项目""想做个小工具/小产品""要不要做 X""从零开始做点什么""这个功能还没想清楚""新功能怎么规划""开工""开坑""立项"，或开始一个全新项目、全新功能而需求还模糊时，先用本技能给出路线再进入对应节点。绝不在需求模糊时直接写代码。
---

# 立项流程总控（project-flow）

本技能是 [mattpocock/skills](https://github.com/mattpocock/skills) 那条
`grill → spec → tickets → implement` 链的路由层。

**用户记不住细节，你负责记。** 用户只需要说一句话，你负责把他送到正确的节点。

## 硬规则

1. 需求模糊（说不清做什么、给谁用、什么算完成）→ **绝不直接写代码**。
2. **一次只推进一个节点**，节点之间停下来等用户。
3. 三个 gate 不许跳过：
   - **G1** grill 结束：frontier 为空 **且用户亲口说"达成共识"**
   - **G2** spec 内部：**接缝(seams)先给用户确认**，再动笔写
   - **G3** 每张票之间：**换新会话 / 清上下文**

## 第一步：只问一句话

> 「这是 ①全新项目、②已有仓库里的新功能、还是 ③只是个想法还没决定要不要做？」

不要问清单。三选一，用户回一个数字就能继续。

## 第二步：给路由

| 情况 | 第一步 | 说明 |
|---|---|---|
| ③ 只是想法，还没打算动手 / 不在任何项目里 | `grill-me` | 无状态、不写文件 |
| ② 已有仓库，为新功能做改动 | `grill-with-docs` | 会写 GLOSSARY.md / ADR |
| ① 全新项目 | 先 `grill-me` 定要不要做 → 建仓库 → `grill-with-docs` → `setup-matt-pocock-skills` | 有仓库之后一律用 grill-with-docs |

**二选一，不要串着跑。** 两者共用同一套 `grilling` 访谈内核；有仓库可写时
grill-with-docs 严格优于 grill-me。

## 第三步：判断分流

- 出现「必须看到才能答」的问题（手感 / UI / 状态机 / 布局）→ 用 `prototype` 回答，来回用 `handoff` 交接
- 整体**一个上下文窗口做得完** → 直接 `implement`，**跳过 spec**
- **跨多个会话** → `to-spec` → `to-tickets` → 逐票 `implement`

判断依据只有一个：**"这活儿塞不进一个上下文窗口吗？"** 塞得进就别上 spec。

## 节点速查

| 节点 | skill | 出口门 |
|---|---|---|
| 盘问到共识 | `grill-with-docs` / `grill-me` / `grilling` | 前沿空 + 用户确认 |
| 一次性配置 | `setup-matt-pocock-skills` | tracker / 标签 / docs 位置三项配完 |
| 记录成 spec | `to-spec` | 接缝先确认才写 |
| 切成票 | `to-tickets` | 垂直切片 + 阻塞边 + 用户核对编号 |
| 实现单票 | `implement` | 接缝上 TDD + code-review + commit |
| 审查 diff | `code-review` | Standards + Spec 双轴 |
| 一次性原型 | `prototype` | 只回答一个具体设计问题 |
| 复盘 | `retro` | 改的是环境，不是代码 |

## 上下文卫生（最容易翻车的地方）

- `grill` → `to-spec` → `to-tickets`：**同一个会话跑完，不清空、不 compact**
- 每张票 `implement`：**换新会话**，票是自包含的
- spec 发布后：**摘掉 `ready-for-agent` 标签**，否则自动 agent 会把整份 spec 一把梭
- 接近 150k tokens 就在最近的阶段边界 compact，不要在退化状态硬撑

## 反模式

- 先 grill-me 再 grill-with-docs（重复，二选一）
- 单会话的小改动也上 to-spec
- grill 还没谈完就 to-spec（会把"未决定"固化成"已决定"）
- `to-spec` 之后清空上下文再 `to-tickets`（大 spec 被 tracker 截断）
- 把 `to-tickets` 产出的票丢给 triage（triage 只处理非你创建的原始 issue）

## 用户怎么说 → 你怎么做

| 用户说 | 你做什么 |
|---|---|
| "我有个想法…" / "开坑" / "立项" / "新项目" | 问上面那一句，然后给路由 |
| "盘问我" / "压力测试这个方案" / "grill me" | 直接 `grilling` |
| "这个还没想清楚" | 判断在不在仓库里 → 对应的 grill |
| "定下来了，开始做" | 判断单会话还是多会话 → `implement` 或 `to-spec` |
| "这个先放放" | 把已定决策落进 GLOSSARY/ADR，然后停 |

## 与本机其他技能的关系

- `brainstorming`：它用于**发散创意**（把选项摊开）；本技能用于**收口立项**
  （把想法变成能施工的决策）。用户说"开新项目 / 新功能 / 我有个想法"时，优先本技能。
- `using-superpowers`：它的"先调技能再回答"与本技能不冲突——本技能就是那个该先调的东西之一。

## 适配不同 harness

| Harness | 怎么调下游技能 |
|---|---|
| Claude Code | `Skill(skill: "grilling")` |
| DeepSeek Harness | `skill(name: "grilling")` |
| 其他 / 无技能机制 | 直接读 `<skills-dir>/<name>/SKILL.md` 并按内容执行 |

**注意**：上游 mattpocock/skills 的四个技能带 `disable-model-invocation: true`。
在部分 harness（例如 DSH）里这是**硬禁用**——模型完全调不动，路由表会失效。
安装时按 `docs/install.md` 说明处理。

**不要凭记忆复述规则。** 需要细节时去读对应技能的 SKILL.md 原文。
