# project-flow

> **把"我有个想法"变成"一路做到 PR"的一张路由卡。**
>
> 一个约 100 行的 agent skill，给 [mattpocock/skills](https://github.com/mattpocock/skills)
> 的 `grill → spec → tickets → implement` 链提供入口与门禁。

---

## 它解决的问题

绝大多数 AI 编码翻车不是模型不够聪明，而是**它把"还没想清楚"当成了"已经想清楚"**。

**没有 project-flow：**

```
你：我想给我的素材库加个审片批注功能
AI：好的！我来帮你实现。我先看看你的项目结构……
    （30 秒后开始写 Model、Controller、前端组件）
你：等等，我还没说要审批流
AI：（已经写了 600 行）
```

**有 project-flow：**

```
你：我想给我的素材库加个审片批注功能
AI：这是「已有仓库里的新功能」。路线：grill-with-docs → 判断单/多会话 → 分流。
    先盘问，我一次给一整轮。

    ❓ Q1 - 批注绑在时间码区间上，还是绑在整段素材上？
    ➡️ 建议：时间码区间 + 帧精度……
```

差别不在"模型更聪明"，在于**多了一道强制工序**。

---

## 它是什么 / 不是什么

| 是 | 不是 |
|---|---|
| 一张路由卡：现在进哪个节点、出口是什么 | 知识库：不装 spec 模板、不装访谈技巧 |
| 三个 gate 的执行者 | 决策者：不替用户拍板 |
| 触发层：让流程在该出现的时候出现 | 流程本身：真正干活的是下游技能 |

**故意做小**（约 100 行 / 5 KB）：它每次对话都可能被加载，属于"入口卡"。
把 `to-spec` 的七节模板塞进来，等于每轮付几千 token 读用不上的规则。
细节属于下游技能，这里只留判断。

---

## 快速开始

装好之后，你只需要说一句话：

```
开坑
```

AI 会问你：

> 「这是 ①全新项目、②已有仓库里的新功能、还是 ③只是个想法还没决定要不要做？」

回一个数字，流程就跑起来了。

**三个口令**：`开坑` / `立项` / `盘问我`（最后一个直接进访谈，不谈路由）。

---

## 路由表

| 你的情况 | 走哪 |
|---|---|
| ③ 只是想法，还没打算动手 | `grill-me` |
| ② 已有仓库里的新功能 | `grill-with-docs` |
| ① 全新项目 | `grill-me` 定要不要做 → 建仓库 → `grill-with-docs` → `setup-matt-pocock-skills` |

然后按体量分流：

```
                    ┌─ 有"必须看到才能答"的问题？→ prototype（用 handoff 来回）
                    │
判定：塞得进一个上下文窗口吗？
                    │
      ┌─────────────┴──────────────┐
   塞得进                        跨多个会话
      │                             │
  /implement              /to-spec → /to-tickets → /implement × N
      │                             │
      └─────────────┬───────────────┘
                    │
              /code-review → /pr → /retro
```

---

## 三个 gate

| Gate | 防的是什么 | 不设会怎样 |
|---|---|---|
| **G1** 谈完才往下走 | 用户没确认就进 spec | spec 把"未决定"固化成"已决定"，后面全歪 |
| **G2** 接缝先确认 | 测试边界没谈就写 spec | `tdd` 在错误边界写测试，`code-review` 把没约定的接缝当缺陷 |
| **G3** 每票清上下文 | 上一票污染下一票 | 越往后越糊涂，最后说不清哪些是这次决定的 |

G1 的措辞是 **"用户亲口说"** —— 来自 grilling 原文：
`Do not act on it until the user confirms you have reached a shared understanding.`
AI 不能替用户宣布达成共识。

---

## 上下文卫生

最有价值、也最容易被忽略的一节：

| 区间 | 策略 |
|---|---|
| grill → to-spec → to-tickets | **一个窗口跑到底**，不清、不 compact |
| 每张票 implement | **换新窗口**，票是自包含的 |
| spec 发布后 | **摘掉 `ready-for-agent` 标签** |
| 接近 150k tokens | 在最近的阶段边界 compact，别硬撑 |

---

## 安装

### Claude Code

```bash
git clone https://github.com/mattpocock/skills.git /tmp/mp-skills
cp -r /tmp/mp-skills/skills/engineering/{grill-with-docs,domain-modeling,to-spec,to-tickets,implement,tdd,code-review,prototype,retro,setup-matt-pocock-skills} ~/.claude/skills/
cp -r /tmp/mp-skills/skills/productivity/{grilling,grill-me,handoff} ~/.claude/skills/
cp -r ./ ~/.claude/skills/project-flow
```

### DeepSeek Harness

```powershell
$sk = Join-Path $env:USERPROFILE ".dsh\skills"
New-Item -ItemType Directory -Path (Join-Path $sk "project-flow") -Force | Out-Null
Copy-Item .\SKILL.md (Join-Path $sk "project-flow\SKILL.md") -Force
```

> ⚠️ DSH 把 `disable-model-invocation: true` 当**硬禁用**处理。
> 需要把那几个技能里的这一行删掉，否则路由表指向的技能 AI 根本调不动。

### 手动 / 其他 agent

把 `SKILL.md` 放到你的 skills 目录，或直接把内容粘贴进系统提示 / `AGENTS.md`。

---

## 让它更可靠：加一段 AGENTS.md

技能靠"描述匹配"触发，会受措辞影响；`AGENTS.md` 是**每轮强制注入**的，命中率 100%。

见 [`examples/AGENTS.md-snippet.md`](examples/AGENTS.md-snippet.md)。

---

## 仓库结构

```
project-flow/
├── SKILL.md                      # 技能本体（可移植版，约 100 行）
├── README.md
├── LICENSE                       # MIT
├── CHANGELOG.md
├── docs/
│   ├── walkthrough.md            # 九幕完整实例：从"我有个想法"到 PR
│   ├── design-notes.md           # 为什么这么短 / 取舍 / 局限
│   └── install.md                # 各 harness 安装与常见坑
└── examples/
    └── AGENTS.md-snippet.md
```

---

## 边界

**它依赖下游技能存在。** 单独装 `project-flow` 只会得到一张指向空处的路由表。
完整依赖链：`grilling` / `grill-me` / `grill-with-docs` / `domain-modeling` /
`setup-matt-pocock-skills` / `to-spec` / `to-tickets` / `implement` / `tdd` /
`code-review` / `prototype` / `retro`。

**它不是给"想清楚了的活儿"用的。** 改个 bug、加个设置，直接做，
用了反而慢。

---

## 致谢

整套方法论来自 [Matt Pocock](https://github.com/mattpocock/skills)（MIT）。
本仓库只提供**路由与门禁这一层**，不重写任何下游技能。

Grilling 内核、spec 模板、tracer-bullet 切片法、双轴 code review 均为其原创。

---

## License

MIT
