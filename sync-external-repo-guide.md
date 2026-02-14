# 同步外部仓库操作指南

## 概述

本文档记录了如何将外部 Git 仓库（以 `ethereum/go-ethereum` 为例）同步到自己的仓库中的完整流程。

## 1. 添加远程仓库

将外部仓库添加为远程源，命名为 `external`：

```bash
git remote add external https://github.com/ethereum/go-ethereum.git
```

验证是否添加成功：

```bash
git remote -v
# 预期输出：
# external  https://github.com/ethereum/go-ethereum.git (fetch)
# external  https://github.com/ethereum/go-ethereum.git (push)
```

## 2. 拉取远程数据

```bash
git fetch external
```

## 3. 创建本地分支并绑定远程分支

创建本地分支 `sync-external`，跟踪远程 `external/master`：

```bash
git checkout -b sync-external external/master
```

验证跟���关系：

```bash
git branch -vv
# 预期输出：
# * sync-external  xxxxxxx [external/master] 最新提交信息
```

## 4. 同步远程最新代码

```bash
git checkout sync-external
git fetch external
git merge external/master
```

## 5. 合并 sync-external 到 main

由于两个分支来自不同仓库，没有共同的提交历史，需要添加 `--allow-unrelated-histories` 参数：

```bash
git checkout main
git merge --no-ff --allow-unrelated-histories sync-external -m "Merge sync-external (go-ethereum) into main"
```

如果出现冲突：

```bash
git status
# 手动解决冲突后
git add .
git commit -m "Merge sync-external (go-ethereum) into main"
```

推送到远程：

```bash
git push origin main
```

## 6. 标签管理

### 6.1 查看当前标签

```bash
git tag
git tag | wc -l
```

### 6.2 批量删除同步过来的标签

```bash
# 删除所有本地标签
git tag | xargs git tag -d

# 删除所有远程标签（如果已推送）
git push origin --delete $(git tag -l)

# 或只删除特定前缀的标签（例如 v1.* 开头）
git tag -l "v1.*" | xargs git tag -d
git tag -l "v1.*" | xargs -I {} git push origin --delete {}
```

### 6.3 打上自己的标签

```bash
git tag -a v1.0.0 -m "Initial merge of go-ethereum into main"
git push origin v1.0.0
```

### 6.4 防止后续同步时拉取外部标签

```bash
# 永久配置，fetch external 时不拉取标签
git config remote.external.tagOpt --no-tags

# 后续 fetch 时也可以手动指定
git fetch external --no-tags
```

## 7. 日常同步流程（后续操作）

每次需要同步外部仓库最新代码时，执行以下步骤：

```bash
# 1. 切换到 sync-external 分支
git checkout sync-external

# 2. 拉取最新代码（不拉标签）
git fetch external --no-tags
git merge external/master

# 3. 切换到 main 分支并合并
git checkout main
git merge --no-ff sync-external -m "Sync go-ethereum updates into main"

# 4. 推送到远程
git push origin main
```

## 附录：常用命令速查

| 操作 | 命令 |
|------|------|
| 查看远程仓库 | `git remote -v` |
| 查看所有分支 | `git branch -a` |
| 查看分支跟踪关系 | `git branch -vv` |
| 查看所有标签 | `git tag` |
| 查看标签详情 | `git show <tag-name>` |
| 删除本地标签 | `git tag -d <tag-name>` |
| 删除远程标签 | `git push origin --delete <tag-name>` |
| 查看提交历史 | `git log --oneline` |