---
name: git-collaboration
description: >
  Git 协作工作流专家技能。适用于任何使用 Git 进行团队协作的场景，特别是 Fork + Feature Branch + Pull Request
  模式。

  当用户需要执行 Git 操作（pull/push/branch/merge/rebase）、解决 Git 问题（冲突、推送失败、分支混乱）、创建分支、

  发起 PR、同步上游代码、或者询问 Git 工作流程时自动激活。也适用于新手学习 Git 协作、code review 前的代码同步、

  多 remote 仓库管理等场景。
version: 1.0.0
author: Claude Code
metadata:
  title: Git 协作工作流
  description_zh: Git 协作工作流专家技能，覆盖 Fork + Feature Branch + Pull Request 模式
  author: joe
  version: 1.0.0
  license: MIT
---

# Git 协作工作流

通用 Git 协作助手，覆盖 **Fork + Feature Branch + Pull Request** 模式的完整工作流。

## 适用场景

✅ **激活本 skill 的场景**：
- 执行 Git 命令（pull/push/branch/merge/rebase/stash）
- 解决 Git 问题（冲突、推送失败、分支落后、误提交）
- 创建功能分支或修复分支
- 发起 Pull Request（PR）或 Merge Request（MR）
- 同步上游代码（upstream/origin）
- 解释 Git 工作流程或分支策略
- Git 新手需要指导
- Code review 前的代码同步准备

❌ **不适用场景**：
- 纯本地 Git 操作（单用户、无协作）
- Git 配置问题（user.name/email 等）
- Git 高级功能（reflog、bisect、filter-branch 等历史修复）

## 核心工作流

### Fork + Feature Branch + Pull Request 模式

**架构**：
```
上游仓库（upstream/origin）← 原始仓库（团队维护）
    ↓ (fork)
你的仓库（origin）← 你的工作空间
    ↓ (clone)
本地仓库 ← 你在本地开发
```

**标准流程（7 步）**：
```bash
# 1. 切换到主线，更新到最新（每天开始前）
git checkout main
git pull upstream main              # 从上游拉取最新代码
# 或使用 rebase（推荐，线性历史）
git pull upstream main --rebase

# 2. 创建功能分支（基于最新 main）
git checkout -b feat/你的功能名

# 3. 开发 + 频繁提交（每 1-2 小时一次）
git add .
git commit -m "feat(模块): 简短描述"

# 4. 推送到你的远程仓库
git push origin feat/你的功能名

# 5. 在代码托管平台发起 Pull Request → target: main

# 6. Code review + 修改（如有需要）

# 7. 合并 → 删除本地和远程分支
git branch -d feat/你的功能名
git push origin --delete feat/你的功能名
```

### 提交信息规范

**格式**：`<类型>(<模块>): <简短描述>`

**类型对照表**：
| 类型 | 用途 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(agent2): 双源交叉验证命中 CV003` |
| `fix` | 修复 bug | `fix(profile): 修复画像数据不刷新` |
| `docs` | 文档更新 | `docs: 更新 API 接口说明` |
| `style` | 格式调整（不影响功能） | `style: 修复缩进和空格` |
| `refactor` | 重构（无功能变化） | `refactor(orchestrator): 提取公共方法` |
| `test` | 添加/修复测试 | `test(profile): 画像反馈接口测试` |
| `chore` | 构建/工具/依赖更新 | `chore: 升级 pytest 到 8.0` |

**好 vs 坏**：
- ✅ `feat(agent2): 添加双源数据验证逻辑`
- ✅ `fix(server): 修复 /api/analyze 超时问题`
- ❌ `改了东西`
- ❌ `fix bug`
- ❌ `update`

## 高频命令速查

### 基础操作

| 场景 | 命令 | 说明 |
|------|------|------|
| 查看状态 | `git status` | 哪些文件改动/未跟踪 |
| 查看提交历史 | `git log --oneline --graph` | 图形化提交历史 |
| 查看分支 | `git branch -a` | 本地 + 远程所有分支 |
| 切换分支 | `git checkout <branch>` | 切换到指定分支 |
| 新建分支 | `git checkout -b <branch>` | 创建并切换到新分支 |
| 删除分支 | `git branch -d <branch>` | 删除已合并的分支 |
| 强制删除 | `git branch -D <branch>` | 删除未合并的分支（慎用） |

### 提交和推送

| 场景 | 命令 |
|------|------|
| 提交所有改动 | `git add . && git commit -m "信息"` |
| 提交指定文件 | `git add <file1> <file2>` |
| 推送新分支 | `git push -u origin <branch>` |
| 推送已有分支 | `git push origin <branch>` |
| 修改最近提交 | `git commit --amend -m "新信息"` |
| 添加遗漏文件到最近提交 | `git add <file> && git commit --amend --no-edit` |

### 同步和合并

| 场景 | 命令 |
|------|------|
| 拉取 upstream 最新代码 | `git pull upstream main --rebase` |
| 拉取 origin 最新代码 | `git pull origin <branch>` |
| 拉取所有远程分支更新 | `git fetch --all` |
| 合并其他分支 | `git merge <branch>` |
| 变基（线性历史） | `git rebase main` |
| 终止变基冲突 | `git rebase --abort` |

### 撤销和恢复

| 场景 | 命令 |
|------|------|
| 撤销未提交的改动 | `git checkout -- <file>` |
| 撤销所有未提交改动 | `git reset --hard HEAD` |
| 撤销最近提交（保留改动） | `git reset --soft HEAD~1` |
| 撤销最近提交（丢弃改动） | `git reset --hard HEAD~1` |
| 暂存当前改动 | `git stash` |
| 查看暂存列表 | `git stash list` |
| 恢复最近暂存 | `git stash pop` |
| 删除暂存 | `git stash drop` |

## 常见问题处理

### 问题 1：分支落后上游 N 个提交

**症状**：
```
Your branch is behind 'upstream/main' by 9 commits.
```

**原因**：上游有新的合并，你的本地分支没更新。

**解决**：
```bash
# 方案 1: Rebase（推荐，保持提交历史线性）
git pull upstream main --rebase

# 方案 2: Merge（会生成合并提交，历史不线性）
git pull upstream main

# 如果遇到冲突，解决后：
git add .
git rebase --continue
```

### 问题 2：推送被拒绝（non-fast-forward）

**症状**：
```
! [rejected]        feat/xxx -> feat/xxx (non-fast-forward)
```

**原因**：远程有你的分支没有的提交（别人改了你推的分支）。

**解决**：
```bash
# 1. 先拉取远程更新
git pull origin feat/xxx

# 2. 解决冲突（如果有）
# 手动编辑冲突文件，然后：
git add .
git commit -m "merge: 解决冲突"

# 3. 重新推送
git push origin feat/xxx
```

### 问题 3：代码冲突

**冲突文件示例**：
```python
<<<<<<< HEAD
# 你的代码
health_score = 60
=======
# 别人的代码
health_score = 55
>>>>>>> feat/xxx
```

**解决步骤**：
1. 打开冲突文件
2. 删除 `<<<<<<<`、`=======`、`>>>>>>>` 标记
3. 保留需要的代码（或合并）
4. `git add .` → `git commit` → `git push`

### 问题 4：误提交敏感文件

**风险文件**：`.env`、`profiles/`、`traces/`、密钥、密码

**紧急处理**：
```bash
# 1. 立即从 Git 删除但保留本地文件
git rm --cached .env

# 2. 确保 .gitignore 已忽略
echo ".env" >> .gitignore

# 3. 提交并推送
git commit -m "chore: 移除敏感文件 .env"
git push origin <branch>

# ⚠️ 如果已推送到远程：
# - 立即修改所有密码/密钥/API Key
# - 联系仓库管理员清理 Git 历史（git filter-branch 或 BFG）
```

### 问题 5：分支状态检查

**查看所有分支的 ahead/behind 状态**：
```bash
git branch -vv

# 输出解读：
# * feat/mobile-backend-integration  e4dfb9d [origin/feat/mobile-backend-integration]
#   ↑ 当前分支                         ↑ 提交哈希   ↑ 远程分支
#   main                             4dfcf7f [origin/main: ahead 1]
#                                                  ↑ 比远程多 1 个提交（需要 push）
```

**状态标记**：
- `[ahead N]`：比远程多 N 个提交（需要 push）
- `[behind N]`：比远程少 N 个提交（需要 pull）
- `[gone]`：远程分支已被删除

### 问题 6：不小心改乱了

```bash
# 撤销工作区改动（未提交）
git checkout -- <file>

# 撤销所有未提交改动
git reset --hard HEAD

# 撤销最近一次提交（保留改动到工作区）
git reset --soft HEAD~1

# 撤销最近 3 次提交（危险！）
git reset --hard HEAD~3
```

### 问题 7：从错误分支提交了代码

```bash
# 1. 记录当前提交哈希
git log --oneline -1
# 输出：e4dfb9d feat(module): 提交信息

# 2. 切换到正确分支
git checkout correct-branch

# 3. 把提交 cherry-pick 过去
git cherry-pick e4dfb9d

# 4. 切回错误分支，撤销提交
git checkout wrong-branch
git reset --hard HEAD~1
```

## 分支管理规范

### 分支命名规范

```
✅ feat/user-profile         # 用户画像功能
✅ feat/drg-audit            # DRG 审计功能
✅ fix/agent2-score          # Agent2 评分修复
✅ docs/api-documentation    # API 文档
✅ chore/update-dependencies # 依赖更新

❌ feature-branch            # 太泛
❌ my-branch                 # 不知道干啥的
❌ test                       # 和测试文件重名
❌ temp                       # 临时分支
❌ update                     # 没意义的描述
```

### 分支保护规则

| 分支 | 保护级别 | 操作限制 |
|------|---------|---------|
| `main` | 🔒 严格保护 | 禁止直接 push，必须通过 PR + review |
| `develop` | 🟡 中度保护 | 允许核心维护者 push，其他需 PR |
| `feat/*` | 🟢 自由开发 | 开发者自由 push |
| `fix/*` | 🟢 自由开发 | 开发者自由 push |
| `hotfix/*` | 🟡 快速通道 | 紧急修复可快速合并 |

### 多 Remote 管理

**典型配置**：
```bash
# 查看所有 remote
git remote -v

# 添加 upstream（首次协作时）
git remote add upstream https://github.com/owner/repo.git

# 拉取 upstream 最新代码
git fetch upstream
git merge upstream/main

# 推送到 origin（你自己的仓库）
git push origin feature-branch
```

**命名约定**：
- `origin` → 你自己的远程仓库（用于 push）
- `upstream` → 原始仓库/团队仓库（用于 pull 同步）

## 协作最佳实践

### ✅ 推荐做法

1. **频繁同步**
   - 每天开始前 `git pull upstream main --rebase`
   - 开发前先确认分支是最新的

2. **小步提交**
   - 每完成一个小功能就 commit
   - 粒度：1-2 小时的工作量
   - 提交信息清晰具体

3. **先测试再推送**
   - 本地跑通测试 `pytest` / `npm test`
   - 确保代码可运行再 push

4. **PR 描述清晰**
   - 改了什么 + 为什么改 + 如何验证
   - 附上截图/录屏（前端项目）
   - 关联 issue（如有）

5. **及时沟通**
   - 遇到冲突或不理解的地方，**先问团队**
   - 不要盲目执行危险命令（`reset --hard`、`rebase` 等）

6. **Code Review 态度**
   - Review 别人的 PR 时提建设性意见
   - 被 review 时虚心接受，不要 defensive

### ❌ 避免做法

1. **不要直接 push main** → 必须通过 PR + review
2. **不要提交敏感文件** → `.env`、密钥、个人配置
3. **不要 rebase 已推送的共享分支** → 改写历史引发他人冲突
4. **不要盲目 `git reset --hard`** → 会丢失未提交的代码
5. **不要从 upstream push** → 你没有权限（报错是正常的）
6. **不要一人占着分支不合并** → 开发完及时发起 PR
7. **不要提交大文件** → 使用 Git LFS 或外部存储

## Git 高级技巧

### 交互式 Rebase（整理提交历史）

```bash
# 整理最近 3 次提交（合并/修改/删除）
git rebase -i HEAD~3

# 常用操作：
# pick = 保留提交
# squash = 合并到前一个提交
# reword = 修改提交信息
# drop = 删除提交
```

### Cherry-pick（移植特定提交）

```bash
# 把某个提交应用到当前分支
git cherry-pick <commit-hash>

# 批量移植多个提交
git cherry-pick <commit1> <commit2> <commit3>
```

### 查看文件改动历史

```bash
# 查看某个文件的提交历史
git log --oneline <file>

# 查看某文件的具体改动
git show <commit-hash>:<file>

# 查看某文件每一行的最后修改者和时间
git blame <file>
```

### 标签管理

```bash
# 创建标签
git tag v1.0.0

# 推送标签到远程
git push origin v1.0.0

# 查看所有标签
git tag --list
```

---

## 🚀 新手快速入口

> **第一次用 Git？** → 阅读 [`references/quick-start.md`](references/quick-start.md)（口语极简版，4 个场景 + 3 个常见问题）

**核心场景**：
1. **每天上班** → `git pull upstream main`
2. **开发新功能** → `git checkout -b feat/功能名`
3. **保存代码** → `git add . && git commit -m "feat(模块): 描述"`
4. **推送代码** → `git push origin feat/功能名`

**常见问题**：
- 推送失败 → 先 `git pull` 再 `git push`
- 代码冲突 → 手动选择保留的代码
- 分支落后 → `git pull upstream main`

**完整版本**：参见 [`references/quick-start.md`](references/quick-start.md)（可打印版）

---

## 🔀 Rebase vs Merge 快速决策

快速参考：[`references/rebase-vs-merge.md`](references/rebase-vs-merge.md)

```
分支是否已推送到远程？
  ├─ 是（公共） → Merge
  └─ 否（私有）
      ├─ 要线性历史 → Rebase
