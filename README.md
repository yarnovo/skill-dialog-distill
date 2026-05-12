# skill-dialog-distill

老板/lead 对话沉淀 skill · 扫最近对话识别可沉淀点 · 推荐沉淀位置 · 老板挑后自动写 4 真源 + commit。

## 4 真源

1. `~/.claude/CLAUDE.md` · 全局指令
2. `~/.claude/memories/*.md` · 默认/按需加载 memory
3. `~/.claude/skills/<name>/SKILL.md` · skill 行为 + TRIGGER
4. `~/.claude/agents/*.md` · advisor 提示词 + TRIGGER

## 调用

老板说 "/dialog-distill" 或 "沉淀一下" 等自然语言 trigger。

## 跟相邻 skill 区别

- `fix-prompt-source` · 单错反推 (窄 · 立即)
- `memory-to-skill` · 已稳 memory 抽 skill (不同方向)
- `dialog-distill` (本) · 整段对话扫多沉淀点 (广 · 老板调)

详见 `SKILL.md`。

## 分支策略

- `main` = 主分支 (= 部署源 if 该仓有部署 · 否则 sync ~/.claude submodule)
- 临时分支: `feat/<name>` · `fix/<name>` · base 永远 `origin/main`
- PR target = `main` · CI pass + mergeable + 无 conflict 才合
- lead merge 不 auto · subagent 不自合
- subagent 创 worktree 干活前必读本 README

详见 `~/.claude/memories/feedback_lead_main_branch_only.md`
