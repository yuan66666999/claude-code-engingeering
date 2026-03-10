---
# 命令用途：显示当前 git 状态，并附带辅助说明
description: 显示当前 git 状态，并附带辅助说明
# 允许调用的工具：git 相关 Bash 命令
allowed-tools: Bash(git:*)
# 使用的模型
model: haiku
---

显示当前 git 状态，并提供易读的上下文说明。

## 步骤

1. 执行 `git status`
2. 用友好的格式总结当前仓库状态

## 输出格式

```
## Git Status

Branch: [branch name]
State: [clean / has changes / has conflicts]

### Staged Changes (ready to commit)
- [files]

### Unstaged Changes (not staged)
- [files]

### Untracked Files
- [files]

### Suggested Next Steps
- [接下来建议执行的操作]
```

如果仓库是干净的，直接输出：
```
✓ 当前分支 [name] 工作区干净
```
