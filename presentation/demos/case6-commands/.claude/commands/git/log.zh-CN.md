---
# 命令用途：显示最近的 git 提交历史并附带摘要
description: 显示最近的 git 提交历史并附带摘要
# 命令参数提示：可选，显示多少条提交记录，默认 5 条
argument-hint: [可选：提交数量，默认 5]
# 允许调用的工具：git 相关 Bash 命令
allowed-tools: Bash(git:*)
# 使用的模型
model: haiku
---

显示最近的 git 提交历史。

提交数量：`$ARGUMENTS`（如果未指定，默认显示 5 条）

## 步骤

1. 执行 `git log --oneline -n [count]`
2. 给出简短摘要

## 输出格式

```
## Recent Commits

| Hash | Message | Author | Time |
|------|---------|--------|------|
| abc123 | feat: add login | John | 2h ago |
| def456 | fix: null check | Jane | 5h ago |
...

### Summary
- Total: [n] commits shown
- Most active: [most frequent commit type]
- Recent focus: [what recent work seems to be about]
```

保持输出简洁、易扫描。
