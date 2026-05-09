---
name: dialog-distill
description: |
  老板/lead 对话沉淀 · 扫最近对话识别可沉淀点 (decisions · push back · 偏好 · 新 SOP · 新规约) · 推荐沉淀位置 (CLAUDE.md / memories / skills / agents) · 老板挑后自动写 4 真源 + commit。
  TRIGGER when 老板说"/dialog-distill"、"分析对话"、"分析最近对话"、"分析对话沉淀"、"沉淀"、"沉淀对话"、"对话沉淀"、"把这条规约记下来"、"总结最近对话沉淀进 prompt source"、"我们对话学的东西沉淀一下"、"提炼对话"、"复盘对话"。
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

### 1. 扫最近对话

lead 内置识别 · 不需外部工具。重点抓:

- 老板原话引用 (不脑补 · 不极端化 · 见 feedback_dont_extrapolate_boss_words)
- 决策点 / push back / 拍板 / 偏好表态
- 新 SOP / 新规约 / 新流程
- 新资源 / 新凭证 / 新域名
- 反例 (老板纠正 lead) → 但若是单错优先走 fix-prompt-source

### 2. 列可沉淀点

每个点用统一格式:

```
### 候选 N · <一句话标题>

- 类型: <decision / push back / 偏好 / SOP / 反例 / 凭证 / 资源>
- 老板原话: "<≤ 2 句直接引用>"
- 推荐落点: <CLAUDE.md / memories/<file>.md / skills/<name>/SKILL.md / agents/<name>.md>
- 推荐 frontmatter:
    - memory: name / description / type / activation
    - skill: TRIGGER / DO NOT TRIGGER
    - agent: TRIGGER / DO NOT TRIGGER
- 推荐内容: <一段草拟 · ≤ 5 行>
```

### 3. 老板挑

用 AskUserQuestion 一次性列所有候选。老板回 "1,3,5" 或 "全要" 或 "1 改成 X / 3 不要"。

不让老板一个个拍 · 一次菜单挑完。

### 4. 自动写 / 改文件

按老板拍的:

- 新 memory → Write `~/.claude/memories/<filename>.md` + Edit `~/.claude/CLAUDE.md` 索引行
- 修 memory → Edit `~/.claude/memories/<file>.md`
- 修 CLAUDE.md → Edit
- 新 skill → 走 project-skill-setup (不在本 skill 范围 · 让老板手动调)
- 修 skill → Edit `~/.claude/repos/skills/<name>/SKILL.md`
- 新 agent → Write `~/.claude/agents/<name>.md`
- 修 agent → Edit

每次 Edit / Write 必须真改真验证 · 不只草拟。

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
