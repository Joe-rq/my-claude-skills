# Git 协作工作流详细案例

基于真实项目场景的 Git 协作案例，帮助理解理论在实际中的应用。

## 案例 1：第一天入职，如何开始开发

**场景**：新队员加入团队，需要同步代码并开始开发新功能。

### Step 1: 配置环境

```bash
# 1. Clone 你自己的 fork
git clone https://gitee.com/你的用户名/AskHealth.git
cd AskHealth

# 2. 配置用户名和邮箱
git config user.name "你的名字"
git config user.email "你的邮箱@example.com"

# 3. 添加 upstream（队长的仓库）
git remote add upstream https://gitee.com/monicabear/AskHealth.git

# 4. 验证配置
git remote -v
# 应看到 origin 和 upstream 两个地址
```

### Step 2: 同步最新代码

```bash
# 切换到 main 分支
git checkout main

# 从 upstream 拉取最新代码（不是 origin！）
git pull upstream main

# 推送到你的 origin（保持同步）
git push origin main
```

### Step 3: 创建功能分支

```bash
# 基于最新的 main 创建分支
git checkout -b feat/user-profile

# 开始开发...
```

---

## 案例 2：开发了一半，需要同步队友的最新代码

**场景**：你正在开发 `feat/user-profile`，队友已经合并了 PR 到 main，你的分支落后了。

### 方案 1: Rebase（推荐）

```bash
# 1. 提交你当前的改动
git add .
git commit -m "feat(profile): 添加基础档案展示"

# 2. 切换到 main，更新
git checkout main
git pull upstream main

# 3. 切回你的分支，rebase 到最新 main
git checkout feat/user-profile
git rebase main

# 4. 如果有冲突，解决后继续
git add .
git rebase --continue

# 5. 强制推送（因为 rebase 改了历史）
git push origin feat/user-profile --force
```

### 方案 2: Merge（安全，但历史不线性）

```bash
# 1. 切换到 main，更新
git checkout main
git pull upstream main

# 2. 切回你的分支，merge main
git checkout feat/user-profile
git merge main

# 3. 解决冲突（如有）
git add .
git commit -m "merge: 同步 main 最新代码"

# 4. 正常推送
git push origin feat/user-profile
```

**什么时候用哪个？**
- **Rebase**：你自己的私有分支，追求干净的线性历史
- **Merge**：公共分支或不想改写历史

---

## 案例 3：发起 PR 后，需要根据 review 意见修改

**场景**：你在 Gitee 发起了 PR，队长 review 后要求修改几个问题。

```bash
# 1. 切回你的功能分支
git checkout feat/user-profile

# 2. 修改代码...

# 3. 重新提交
git add .
git commit -m "fix(profile): 根据 review 修改数据类型问题"

# 4. 推送到远程（PR 自动更新）
git push origin feat/user-profile

# 5. 在 Gitee PR 页面刷新，队长会看到更新
```

---

## 案例 4：多人修改了同一个文件，冲突了

**场景**：你和队友都修改了 `agents/orchestrator.py` 的同一块代码，合并时冲突。

### 示例冲突

```python
<<<<<<< HEAD
# 你的代码
def calculate_score(data):
    return data['score'] * 1.2

=======
# 队友的代码（来自 feat/xxx）
def calculate_score(data):
    base_score = data['score']
    return base_score * 1.2 + 10

>>>>>>> feat/xxx
```

### 解决步骤

```bash
# 1. 打开冲突文件，手动合并
# 删除 <<<<<<<、=======、>>>>>>> 标记
# 保留你需要的代码（或合并逻辑）

# 合并后的结果：
def calculate_score(data):
    base_score = data['score']
    return base_score * 1.2 + 10  # 使用队友的加权逻辑

# 2. 标记冲突已解决
git add agents/orchestrator.py

# 3. 完成合并
git commit -m "merge: 解决 orchestrator.py 冲突"

# 4. 推送
git push origin feat/user-profile
```

---

## 案例 5：想撤销某次提交

### 场景 A：最近一次提交错了，想修改

```bash
# 修改最近一次提交信息
git commit --amend -m "feat(profile): 修正提交信息"

# 或添加遗漏的文件到最近提交
git add 遗漏的文件
git commit --amend --no-edit

# 强制推送（因为改了历史）
git push origin feat/user-profile --force
```

### 场景 B：提交了不该提交的文件（.env）

```bash
# 1. 从 Git 删除但保留本地
git rm --cached .env

# 2. 确保 .gitignore 包含 .env
echo ".env" >> .gitignore

# 3. 提交并推送
git commit -m "chore: 移除敏感文件 .env"
git push origin feat/user-profile

# ⚠️ 紧急处理：如果 .env 已推送到远程
# - 立即在 Gitee 仓库设置中重新生成 API Key
# - 修改所有相关密码
# - 联系仓库管理员清理历史
```

### 场景 C：撤销最近 3 次提交（危险！）

```bash
# 查看最近 3 次提交
git log --oneline -3

# 输出：
# abc1234 feat(profile): 添加画像功能
# def5678 feat(profile): 添加偏好学习
# ghi9012 feat(profile): 初始化画像模块

# 回滚到 abc1234（保留这 3 次提交的改动到工作区）
git reset --soft abc1234^

# 或丢弃这 3 次提交的所有改动
git reset --hard abc1234
```

---

## 案例 6：同步 upstream 的 bug fix

**场景**：上游仓库（upstream/main）修复了一个 bug，你想同步到你的分支。

```bash
# 方案 1: 直接同步整个 main 分支
git checkout main
git pull upstream main
git push origin main
git checkout feat/your-branch
git rebase main
git push origin feat/your-branch --force

# 方案 2: 只 cherry-pick 特定的 bug fix 提交
# 1. 查看 upstream 的提交
git fetch upstream
git log --oneline upstream/main

# 2. 找到 bug fix 的 commit hash，比如 abc1234
git cherry-pick abc1234

# 3. 解决冲突（如有）
git push origin feat/your-branch
```

---

## 案例 7：PR 合并后，清理本地分支

```bash
# 1. 确认 PR 已在 Gitee 合并

# 2. 切换到 main 并更新
git checkout main
git pull upstream main

# 3. 删除本地分支
git branch -d feat/your-branch

# 4. 删除远程分支
git push origin --delete feat/your-branch
```

---

## 案例 8：误删了未提交的代码，如何恢复

```bash
# 查看最近的操作记录
git reflog

# 输出示例：
# abc1234 HEAD@{0}: commit: feat(profile): 添加画像功能
# def5678 HEAD@{1}: commit: feat(profile): 添加偏好学习
# ghi9012 HEAD@{2}: checkout: moving from feat/user-profile to main
# jkl0123 HEAD@{3}: commit: feat(profile): 初始化画像模块

# 恢复到误删前的状态（比如恢复到 def5678）
git reset --hard def5678

# 代码就恢复了！
```

---

## 常见术语对照

| 术语 | 解释 | 实际场景 |
|------|------|---------|
| **Fork** | 从别人的仓库复制一份到你的账号 | 队长仓库 → 你的仓库 |
| **Clone** | 把远程仓库下载到本地 | `git clone <url>` |
| **Remote** | 远程仓库地址 | origin / upstream |
| **Branch** | 分支 | feat/xxx / fix/xxx |
| **Merge** | 合并分支 | 把 feat/xxx 合并到 main |
| **Pull Request (PR)** | 申请合并代码 | 在 Gitee/GitHub 发起 PR |
| **Rebase** | 变基 | 把你的提交"接到"最新的 main 后面 |
| **Cherry-pick** | 移植特定提交 | 只合并某个 bug fix，不合并整个分支 |
| **Commit** | 提交 | 保存一次改动 |
| **Conflict** | 冲突 | 多人改同一个文件，Git 不知道听谁的 |
| **Fast-forward** | 快进合并 | 没有冲突的干净合并（线性前进） |

---

## 进阶技巧

### 并行开发多个功能

```bash
# 使用 git worktree 在多个目录同时开发
git worktree add ../feature-1 feat/feature-1
git worktree add ../feature-2 feat/feature-2

# 现在你可以在不同目录同时开发不同分支
cd ../feature-1  # 开发 feature-1
cd ../feature-2  # 开发 feature-2
```

### 临时保存改动

```bash
# 当前改到一半，需要切换分支
git stash

# 切换到其他分支干活
git checkout main

# 干完活切回来
git checkout feat/xxx
git stash pop  # 恢复改动
```

### 查看某行代码是谁改的

```bash
git blame <file>
# 输出示例：
# abc1234 (Alice 2024-06-01 10:30) def calculate_score(data):
# def5678 (Bob   2024-06-02 14:20)     return data['score'] * 1.2
```

### 批量撤销多个文件

```bash
# 撤销所有 .py 文件的改动
git checkout -- '*.py'

# 撤销 agents/ 目录下所有文件
git checkout -- agents/
```

---

**记住**：真实项目中遇到的 90% 的 Git 问题都可以用本文档的方法解决。
