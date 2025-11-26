# Git Revert 详细教程

`git revert` 是一个安全的撤销提交的方法，它通过创建一个新的提交来反转之前提交的更改，而不是删除历史记录。

## 基本概念

**git revert vs git reset 的区别：**
- `git revert`: 创建新提交来撤销更改，保留历史记录（安全）
- `git reset`: 直接移动 HEAD 指针，可能删除历史记录（危险，尤其在共享分支上）

## 基本用法

### 1. 撤销最近的一次提交
```bash
git revert HEAD
```

### 2. 撤销特定的提交
```bash
git revert <commit-hash>
```

例如：
```bash
git revert abc123d
```

### 3. 撤销多个提交
```bash
# 方法1：逐个撤销
git revert commit1 commit2 commit3

# 方法2：撤销一个范围（注意：不包括 older-commit）
git revert older-commit..newer-commit

# 方法3：撤销一个范围（包括 older-commit）
git revert older-commit^..newer-commit
```

## 常用选项

### --no-commit (-n)
撤销更改但不立即提交，允许你一次性撤销多个提交：
```bash
git revert -n commit1
git revert -n commit2
git revert -n commit3
git commit -m "Revert multiple commits"
```

### --no-edit
使用默认的 revert 提交信息，不打开编辑器：
```bash
git revert --no-edit abc123d
```

### --edit (-e)
编辑 revert 提交信息（默认行为）：
```bash
git revert -e abc123d
```

## 实际操作步骤

### 步骤 1：查看提交历史
```bash
git log --oneline
```

输出示例：
```
d4e5f6g (HEAD -> main) Add feature C
c3d4e5f Add feature B
b2c3d4e Add feature A
a1b2c3d Initial commit
```

### 步骤 2：选择要撤销的提交
假设你想撤销 "Add feature B" (c3d4e5f)：
```bash
git revert c3d4e5f
```

### 步骤 3：处理编辑器
Git 会打开编辑器，默认提交信息类似：
```
Revert "Add feature B"

This reverts commit c3d4e5f.
```

保存并关闭编辑器。

### 步骤 4：查看结果
```bash
git log --oneline
```

现在会看到：
```
e5f6g7h (HEAD -> main) Revert "Add feature B"
d4e5f6g Add feature C
c3d4e5f Add feature B
b2c3d4e Add feature A
a1b2c3d Initial commit
```

## 处理冲突

如果 revert 时出现冲突：

```bash
git revert abc123d
# 出现冲突信息
```

解决步骤：
1. 查看冲突文件：
```bash
git status
```

2. 手动编辑冲突文件，移除冲突标记 (`<<<<<<<`, `=======`, `>>>>>>>`)

3. 标记为已解决：
```bash
git add <conflicted-files>
```

4. 继续 revert：
```bash
git revert --continue
```

或者放弃 revert：
```bash
git revert --abort
```

## 高级场景

### 撤销 merge 提交
Merge 提交有两个父提交，需要指定保留哪个：
```bash
# -m 1 保留第一个父提交（通常是主分支）
git revert -m 1 <merge-commit-hash>
```

### 撤销已推送到远程的提交
```bash
git revert <commit-hash>
git push origin <branch-name>
```

## 最佳实践

1. **在共享分支上优先使用 revert**：保留完整历史，团队成员不会遇到问题
2. **本地未推送的提交可以用 reset**：更简洁
3. **撤销多个提交时注意顺序**：从最新到最旧
4. **写清楚 revert 原因**：在提交信息中说明为什么撤销

## 示例场景

### 场景 1：撤销最近 3 个提交
```bash
git revert HEAD~2..HEAD
# 或
git revert HEAD HEAD~1 HEAD~2
```

### 场景 2：撤销但先不提交（批量操作）
```bash
git revert -n HEAD~2..HEAD
# 检查更改
git status
git diff --staged
# 一次性提交
git commit -m "Revert last 3 commits due to bug"
```

---

[← 返回文档首页](./index.md)
