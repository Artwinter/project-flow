# 安装

## 依赖

`project-flow` 只是路由层。要真的跑起来，需要装齐下游技能：

```
必需（主链）:  grilling  grill-me  grill-with-docs  domain-modeling
               setup-matt-pocock-skills  to-spec  to-tickets  implement
               tdd  code-review
按需:          prototype  handoff  retro
```

来源：[mattpocock/skills](https://github.com/mattpocock/skills)

---

## Claude Code

```bash
TMP=$(mktemp -d)
git clone --depth 1 https://github.com/mattpocock/skills.git "$TMP/mp"
SK="$HOME/.claude/skills"

cp -r "$TMP/mp/skills/productivity/grilling"          "$SK/"
cp -r "$TMP/mp/skills/productivity/grill-me"          "$SK/"
cp -r "$TMP/mp/skills/productivity/handoff"           "$SK/"
cp -r "$TMP/mp/skills/engineering/grill-with-docs"    "$SK/"
cp -r "$TMP/mp/skills/engineering/domain-modeling"    "$SK/"
cp -r "$TMP/mp/skills/engineering/setup-matt-pocock-skills" "$SK/"
cp -r "$TMP/mp/skills/engineering/to-spec"            "$SK/"
cp -r "$TMP/mp/skills/engineering/to-tickets"         "$SK/"
cp -r "$TMP/mp/skills/engineering/implement"          "$SK/"
cp -r "$TMP/mp/skills/engineering/tdd"                "$SK/"
cp -r "$TMP/mp/skills/engineering/code-review"        "$SK/"
cp -r "$TMP/mp/skills/engineering/prototype"          "$SK/"
cp -r "$TMP/mp/skills/engineering/retro"              "$SK/"

mkdir -p "$SK/project-flow"
cp ./SKILL.md "$SK/project-flow/SKILL.md"
```

每个项目首次使用前，在项目根目录跑一次：

```
setup-matt-pocock-skills
```

---

## DeepSeek Harness

```powershell
$sk = Join-Path $env:USERPROFILE ".dsh\skills"

# 1) 下游技能（按需复制，结构与上面 Claude Code 相同）
# 2) 本技能
New-Item -ItemType Directory -Path (Join-Path $sk "project-flow") -Force | Out-Null
Copy-Item .\SKILL.md (Join-Path $sk "project-flow\SKILL.md") -Force
```

### ⚠️ DSH 必做的一步

上游四个技能带 `disable-model-invocation: true`：

```
grill-me  grill-with-docs  to-spec  to-tickets  implement
setup-matt-pocock-skills  retro
```

DSH 把这个字段当**硬禁用**处理——模型调用会直接报错：

```
skill "to-spec" is not available for model invocation
```

**保留这一行 = 路由表指向的技能根本调不动 = 本技能失效。**

删掉即可（每个文件删一行）：

```powershell
$sk = Join-Path $env:USERPROFILE ".dsh\skills"
$targets = @("grill-me","grill-with-docs","to-spec","to-tickets","implement",
             "setup-matt-pocock-skills","retro")
foreach ($t in $targets) {
  $p = Join-Path (Join-Path $sk $t) "SKILL.md"
  $enc = New-Object System.Text.UTF8Encoding($false)
  $lines = [System.IO.File]::ReadAllLines($p)
  $kept  = $lines | Where-Object { $_ -notmatch "^disable-model-invocation:\s*true\s*$" }
  [System.IO.File]::WriteAllText($p, ($kept -join [Environment]::NewLine), $enc)
}
```

> 代价：模型理论上可能自己触发 `to-spec`。由本技能的三道 gate 兜底。

---

## 手动 / 其他 agent

把 `SKILL.md` 的内容直接贴进系统提示、`AGENTS.md`、或 `CLAUDE.md`。
它的正文不依赖任何特定工具名，只有「适配不同 harness」一节提到工具差异。

---

## 验证安装成功

1. 技能列表里能看到 `project-flow`
2. 在任意目录说一句 **「开坑」**
3. 期望响应：

```
这是 ①全新项目、②已有仓库里的新功能、还是 ③只是个想法还没决定要不要做？
```

**如果它直接开始写代码**：说明没命中，去加 `examples/AGENTS.md-snippet.md` 里的兜底。

---

## 常见坑

| 现象 | 原因 | 处理 |
|---|---|---|
| 说了"开坑"没反应 | 描述没命中 | 加 AGENTS.md 兜底 |
| 技能调不动 / not available | `disable-model-invocation` 未删 | 见上方 DSH 一节 |
| `to-spec` 说找不到 tracker | 没跑 setup | 项目根跑 `setup-matt-pocock-skills` |
| `to-tickets` 读 spec 被截断 | 中途清了上下文 | 同一窗口连着跑 |
| 多个 agent 把 spec 当工单干 | `ready-for-agent` 没摘 | 发布 tickets 后立刻摘掉 |
