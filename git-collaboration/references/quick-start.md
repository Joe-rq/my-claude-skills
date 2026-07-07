# 口语极简版（新手专用）

> 只记住这 **4 个场景**，足够应付 90% 日常协作

## 1. 每天上班先做这件事

```bash
git pull upstream main
```

**人话**：从队长的远程仓库拉取最新代码

---

## 2. 开发新功能（从零开始）

```bash
git checkout -b feat/你的功能名
```

**人话**：新建一个功能分支

---

## 3. 做完功能保存代码

```bash
git add .
git commit -m "feat(模块): 简短描述"
```

**人话**：保存代码到本地 Git 仓库

---

## 4. 把代码推送到远程

```bash
# 第一次推新分支
git push -u origin feat/功能名

# 以后推送
git push origin feat/功能名
```

**人话**：把本地代码上传到你的远程仓库

---

## 遇到问题怎么办

### 冲突了（两个人改同一个文件）

1. 打开冲突文件，找到 `<<<<<<<` 标记
2. 保留要的代码，删掉标记
3. 保存 → `git add .` → `git commit` → `git push`

### 推送失败（说 non-fast-forward）

```bash
git pull origin 分支名  # 先拉取
# 解决冲突（如果有）
git push origin 分支名  # 再推送
```

### 分支落后了

```bash
git pull upstream main
```

---

## 极简口诀

```
每日先 pull，开发先分支
写完先 add commit，再 push
冲突不要慌，保存就解决
```
