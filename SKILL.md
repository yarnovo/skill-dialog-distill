---
name: dialog-distill
description: |
  老板/lead 对话驱动的 4 真源 CRUD · 扫最近对话识别可沉淀点 (decisions · push back · 偏好 · 新 SOP · 新规约 · 冲突点) · 推荐沉淀位置 (CLAUDE.md / memories / skills / agents) · 老板挑后自动 Create/Update/Delete 4 真源 + commit · 对话实践跟历史 memory 冲突时以对话为准。
  TRIGGER when 老板说"/dialog-distill"、"分析对话"、"分析最近对话"、"分析对话沉淀"、"沉淀"、"沉淀对话"、"对话沉淀"、"把这条规约记下来"、"总结最近对话沉淀进 prompt source"、"我们对话学的东西沉淀一下"、"提炼对话"、"复盘对话"、"对话跟记忆冲突"、"以我们的为准修记忆"、"回顾对话改 skill/agents/memory"。
  DO NOT TRIGGER when 是单条 fix 错误 (用 fix-prompt-source 更窄)、是已 commit 的代码改动 (用 git log)、是临时调试笔记 (chat history 自带)。
argument-hint: ""
category: meta
allowed-tools: Bash, Read, Edit, Write, AskUserQuestion
---

# dialog-distill · 对话→提示词源蒸馏

## 一句话

老板跟 lead 对话本质是强化学习 · 蒸馏对话产物到 4 真源:

1. **~/.claude/CLAUDE.md** · 全局指令 (默认加载)
2. **~/.claude/memories/*.md** · 默认加载 / 按需加载 memory
3. **~/.claude/skills/<name>/SKILL.md** · skill 行为 + TRIGGER
4. **~/.claude/agents/*.md** · advisor 提示词 + TRIGGER

## 跟 fix-prompt-source 区别

|  | fix-prompt-source | dialog-distill (本) |
|---|---|---|
| 触发 | lead 犯错被纠正后 (反向修源) | 老板主动调 (前向沉淀对话产物) |
| 范围 | 1 个错 / 1 个 prompt 段 | 整段对话 / 多个产物 |
| 频率 | 出错后立刻 | 老板拍 / 周期性 |
| 输出 | 修错的指令 + 强化对的 | 新增/修改 4 类真源 |

## 触发后流程 (5 步)

### 1. 扫最近对话 · 5 类目标 (C/U/D 全谱)

lead 内置识别 · 不需外部工具。**5 类目标全扫 · 不只 Create**:

| 类目标 | 表现 | 落点动作 |
|---|---|---|
| **C 新增** | 老板拍新规约 / 新偏好 / 新资源 / 新决策 | Write memory / Write skill / Write agent / Edit CLAUDE.md 加索引 |
| **U 更新** | 老板细化 / 修正 / 推翻已有规约 | Edit 对应 memory/skill/agent · 不留旧版本层 (见 `feedback_memory_latest_only`) |
| **D 删除** | 老板撤掉旧规约 (例 5-14 撤 5-13 "必派 subagent") · 或冲突识别后旧条过时 | Delete memory 文件 + Edit CLAUDE.md 删索引行 · Edit skill 段落 · Edit agent 段落 |
| **冲突识别** | 对话实践跟历史 memory/skill/agent 不一致 | **以对话为准** (老板原话权威) · 走 D 或 U 路径修正旧源 |
| **lead 执行漏** | 老板纠的错 · 但 memory 内容本身已对 (lead 加载了没执行) | **不动 memory** · 在 commit message / 沉淀报告里标 "lead 执行漏 · memory 无错" · 留 git trail 即可 |

### 1.5 冲突识别 SOP (对话 vs 历史源)

每个候选必带"冲突扫描":

```
1. grep 对话关键词 → 看 memories/ + skills/ + agents/ + CLAUDE.md
2. 找到相关条 → 跟对话拍板对比
3. 分三类:
   a. 内容一致 → lead 执行漏 (不动源)
   b. 内容部分冲突 → U 更新 (改细节 · 留主框架)
   c. 内容根本冲突 (旧规约被推翻) → D 删除 + 新 C 写入
4. 报告里标清 (a/b/c)
```

⚠️ **lead 执行漏 ≠ memory 错** · 不要误判改 memory · 这会把对的规约改坏。

重点抓 (5 类内的具体 signal):

- 老板原话引用 (不脑补 · 不极端化 · 见 `feedback_dont_extrapolate_boss_words`)
- 决策点 / push back / 拍板 / 偏好表态
- 新 SOP / 新规约 / 新流程
- 新资源 / 新凭证 / 新域名
- 反例 (老板纠正 lead) → 但若是单错优先走 fix-prompt-source
- **冲突点** (老板说"以我们的为准" / "这跟之前矛盾" / "之前那条不算了") → 优先 D/U 旧条

### 2. 列可沉淀点

每个点用统一格式:

```
### 候选 N · <一句话标题>

- 类型: <decision / push back / 偏好 / SOP / 反例 / 凭证 / 资源 / 冲突点>
- 操作: <C 新增 / U 更新 / D 删除 / NoOp lead 执行漏>
- 老板原话: "<≤ 2 句直接引用>"
- 推荐落点: <CLAUDE.md / memories/<file>.md / skills/<name>/SKILL.md / agents/<name>.md>
- 冲突扫描: <相关历史源 grep 结果 · 列已存在条 + 一致/冲突判定>
- 推荐 frontmatter:
    - memory: name / description / type / activation
    - skill: TRIGGER / DO NOT TRIGGER
    - agent: TRIGGER / DO NOT TRIGGER
- 推荐内容: <一段草拟 · ≤ 5 行>
```

### 3. 老板挑

用 AskUserQuestion 一次性列所有候选。老板回 "1,3,5" 或 "全要" 或 "1 改成 X / 3 不要"。

不让老板一个个拍 · 一次菜单挑完。

### 4. 自动 CRUD 文件

按老板拍的 · 4 真源各支持 C/U/D:

**memory**:
- C 新增 → Write `~/.claude/memories/<filename>.md` + Edit `~/.claude/CLAUDE.md` 加索引行
- U 更新 → Edit `~/.claude/memories/<file>.md` · 必要时同步改 CLAUDE.md 索引摘要
- D 删除 → `rm ~/.claude/memories/<file>.md` + Edit `~/.claude/CLAUDE.md` 删索引行

**CLAUDE.md**:
- Edit (改硬规则段 / 索引段 / 调顺序)

**skill** (各 skill 真源在 `~/.claude/repos/skills/<name>/`):
- C 新增 → 走 project-skill-setup (不在本 skill · 老板手动调)
- U 更新 → Edit `~/.claude/repos/skills/<name>/SKILL.md`
- D 删除 → 整 skill 退役: `cd ~/.claude/repos/skills/<name> && git rm -r .` + 主仓删 submodule entry + 删 ~/.claude/skills/<name> symlink

**agent** (各 agent 真源在 `~/.claude/repos/agents/<name>/`):
- C 新增 → 走 agent-hiring (不在本 skill · 老板手动调)
- U 更新 → Edit `~/.claude/repos/agents/<name>/<name>.md`
- D 删除 → 整 agent 退役: 同 skill 模式

每次 Edit / Write / Delete 必须真改真验证 · 不只草拟。

### 4.5 NoOp 路径 (lead 执行漏)

候选标 "操作: NoOp" 时不动任何源文件 · 只:
- 在沉淀报告里标 "lead 执行漏 · memory `<name>` 内容无错"
- commit message 引用本次冲突点 (留 git trail)
- 不写新 memory (反复犯同款再考虑加强源)

### 5. commit + push

按落点的归属仓 commit:

- ~/.claude main 仓 (CLAUDE.md / memories / agents): cd ~/.claude && git add ... && git commit && git push
- 各 skill 子仓: cd ~/.claude/repos/skills/<name> && git add ... && git commit && git push
- 子仓改了之后 ~/.claude main 还要再 commit submodule pointer

跟 fix-prompt-source / memory-to-skill 一致。

## 7 类沉淀对象 (lead 扫对话时识别)

| 类型 | 落点 | 例 |
|---|---|---|
| 全局硬规则 | CLAUDE.md | "lead 不准 push --no-verify" |
| 行为 / 偏好 | memories/feedback_*.md | "回复硬规则 ≤ 10 行" |
| 用户身份 / 角色 | memories/user_*.md | "老板 phone-only · 带宽低" |
| 项目阶段 / 决策 | memories/feedback_*.md | "5-9 聚焦小喜单产品" |
| 资源 / 凭证位置 | memories/reference_*.md | "akongtech.cn 是企业邮箱主域" |
| 反复 SOP | skills/<name>/SKILL.md | "agent 产品 4 仓嵌套套" |
| advisor 行为 / TRIGGER | agents/<name>.md | "fc-agent-deployer brief 必含 prefix" |

## 不沉淀

- 一次性 fix (用 git log 留底即可)
- 已 commit 的代码改动 (代码自身是真源)
- 临时调试笔记 (chat history)
- 老板情绪 / 闲聊 (不是规约)
- 还没拍板的 brainstorm (用 todo skill)

## 跟其他 skill 边界

| skill | 范围 |
|---|---|
| **fix-prompt-source** | 单错反推 1 个 prompt 修源 (窄 · 立即) |
| **dialog-distill** (本) | 整段对话扫多个沉淀点 (广 · 老板调) |
| **memory-to-skill** | 已稳 memory 抽 skill (不同方向 · 输出统一改 skill) |
| **knowledge-graph** | 概念节点 (非提示词源) |
| **wiki** | 调研笔记 / 方法论 (非提示词源) |
| **todo** | 没拍板的 brainstorm (沉之前) |

## 质量约束

1. **老板原话保真** · 引用必直接复制粘贴 · 不改写不脑补 (见 feedback_dont_extrapolate_boss_words)
2. **不极端化** · "我的页面应该就是对话页面" 不等于 "砍所有页面"
3. **每个候选独立** · 老板可单独挑 · 不绑定打包
4. **commit message 引用对话来源** · 例: "+ memory: agent 对外 3 件套域名规约 (老板 5-7 拍 · 6 域名/agent)" 跟现有 commit 风格一致

## 入口

- TRIGGER 词列表 (frontmatter description)
- 老板 phone 调用方式: "/dialog-distill" 或 "沉淀一下" 等自然语言
