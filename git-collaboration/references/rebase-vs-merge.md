# Rebase vs Merge 决策树

### 什么时候用 Rebase

```
✅ 你自己的私有分支（还没推送到远程）
✅ 想保持提交历史线性（一条直线）
✅ 想把 N 个小提交合并成 1 个
```

**示例**：
```bash
# 开发了 3 个提交，想整理成 1 个
git rebase -i HEAD~3

# 同步最新的 main 到你的分支
git pull upstream main --rebase
```

### 什么时候用 Merge

```
✅ 公共分支（多人协作）
✅ 不想改写历史
✅ 保留完整的分支合并历史
```

**示例**：
```bash
# 合并 main 到你的分支（保留历史）
git merge main

# 合并你的分支到 main（PR 合并时）
git merge feat/your-feature
```

### 快速决策

```
分支是否已推送到远程？
  ├─ 是（公共分支） → 用 Merge
  └─ 否（私有分支）
      ├─ 想线性历史 → Rebase
      └─ 想保留完整历史 → Merge
```

### 禁止操作

```bash
❌ git rebase main（已推送到远程的分支）   # 改写历史，引发他人冲突
❌ git push --force（公共分支）             # 覆盖他人的代码
```
