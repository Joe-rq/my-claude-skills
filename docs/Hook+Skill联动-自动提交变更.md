# Hook + Skill 联动实例：自动提交变更

> 来源：一篇关于用 Claude Code 管理写作仓库的推文。原始场景：写作者用 Git 管理文章，但经常忘记提交，导致变更堆积。

## 核心模式

**Stop Hook 当守门员 + Skill 当执行者**

```
Claude 完成任务 → Stop Hook 触发
  → 检查 git 状态
  → 有未提交变更？ → 阻止退出，指示调用 /commit
  → 无变更？ → 正常结束
```

这是超级峰提到的 "Hook + Skill 联动" 的第一个落地实例。

---

## 可泛化的模式：任务结束质量门

Stop Hook 的本质是**退出前的强制检查**。不只是 commit，可以套到任何场景：

| Hook 检查内容 | 不通过时触发的 Skill | 适用场景 |
|:--|:--|:--|
| 有未提交变更 | `/commit` | 写作仓库、日常开发 |
| 测试没跑 | `/test` | 代码修改后 |
| 配置格式不合法 | `/validate` | 配置文件修改后 |
| 文档有拼写错误 | `/proofread` | 文档写作后 |

关键区别：写在 CLAUDE.md 里的规则是"建议"，Hook 是"强制"。

---

## 实现配置

### Hook 配置（.claude/settings.local.json）

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/auto-commit.sh"
      }]
    }]
  }
}
```

### Hook 脚本（.claude/hooks/auto-commit.sh）

```bash
#!/bin/bash
# Stop hook: 任务完成后检测未提交变更，有则阻止退出并触发 /commit

cd "$CLAUDE_PROJECT_DIR" 2>/dev/null || exit 0

if git diff --quiet 2>/dev/null && \
   git diff --cached --quiet 2>/dev/null && \
   [ -z "$(git ls-files --others --exclude-standard 2>/dev/null)" ]; then
  exit 0  # 无变更，放行
fi

# 有变更，拦截
cat <<'EOF'
{"decision": "block", "reason": "检测到未提交的变更，请调用 /commit 技能提交更新。"}
EOF
```

### Commit Skill（.claude/skills/commit/SKILL.md）核心逻辑

1. `git status --short` 查看变更
2. 按路径模式分类（文章 / 技能配置 / 代码 / 配置文件）
3. 按主题分组提交，不用 `git add .`
4. 生成中文 commit message，格式固定（添加/润色/更新/优化/修复 + 主题）
5. 明确指定提交文件，排除临时文件

---

## 原文的问题与改进

### 问题 1：防循环机制是摆设

原文用 `stop_hook_active` 标志防止无限循环，但这不是 Claude Code Stop Hook 事件的标准字段，脚本永远读到 false。实际没死循环是因为 commit 后 `git diff --quiet` 通过了——**git 状态检查本身就是防循环的**，不需要额外 flag。

### 问题 2：缺少失败处理

如果 commit 失败（冲突、pre-commit hook 拒绝等），会反复拦截 → 反复失败 → 真正的无限循环。

### 改进建议：加重试上限

```bash
#!/bin/bash
cd "$CLAUDE_PROJECT_DIR" 2>/dev/null || exit 0

# 重试计数，防止 commit 失败时无限循环
LOCK_FILE="/tmp/claude-auto-commit-$$"
RETRY_COUNT=0
if [ -f "$LOCK_FILE" ]; then
  RETRY_COUNT=$(cat "$LOCK_FILE")
fi

if [ "$RETRY_COUNT" -ge 3 ]; then
  rm -f "$LOCK_FILE"
  exit 0  # 超过 3 次，放行并警告
fi

if git diff --quiet 2>/dev/null && \
   git diff --cached --quiet 2>/dev/null && \
   [ -z "$(git ls-files --others --exclude-standard 2>/dev/null)" ]; then
  rm -f "$LOCK_FILE"
  exit 0
fi

echo $((RETRY_COUNT + 1)) > "$LOCK_FILE"
cat <<'EOF'
{"decision": "block", "reason": "检测到未提交的变更，请调用 /commit 技能提交更新。"}
EOF
```

> 注意：`$$` 是当前进程 PID，每次 Claude Code 会话不同，所以计数是按会话隔离的。但如果 Hook 每次是新进程调用，PID 会变——实际使用时需要换成会话级别的标识（如 `$CLAUDE_SESSION_ID`）。
