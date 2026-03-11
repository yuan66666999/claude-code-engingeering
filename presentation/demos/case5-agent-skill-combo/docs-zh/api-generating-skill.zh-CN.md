---
name: api-generating
description: 从 Express 路由文件生成 API 端点文档。当用户要求生成、更新或审查 Express.js 路由的 API 文档时使用。
allowed-tools: [Read, Grep, Glob, Write, Bash(python *)]
---

# API 文档生成 Skill

## 工作流程 — 必须执行

**重要提示**：你必须按顺序执行以下步骤，不可跳过或替换任何步骤。

### 第 1 步：路由发现

**必须使用 Python 脚本进行路由检测：**

```bash
python3 skills/scripts/detect-routes.py src/
```

不要使用 Grep 手动搜索路由 — 脚本能处理各种边界情况
（动态路由、中间件挂载的子路由、重导出的路由），这些是 Grep
模式无法覆盖的。

### 第 2 步：路由分析

对脚本发现的每个路由：

1. 读取路由处理函数的源文件
2. 识别：HTTP 方法、路径、参数、请求体 Schema、响应 Schema
3. 检查认证中间件（如 `requireAuth`、`isAdmin`）
4. 检查验证中间件（如 `validate(schema)`）

### 第 3 步：文档生成

使用 `templates/api-doc.md` 中的模板生成文档。

**输出规则：**
- 每个路由组生成一个 Markdown 文件（如 `docs/api/users.md`）
- 包含请求/响应示例
- 需要认证的端点用 🔒 标记

## 参考文件

- 路由检测脚本：`scripts/detect-routes.py`
- 文档模板：`templates/api-doc.md`
- Express 路由模式：参见同目录下的 PATTERNS.md

## 质量检查清单

完成前请验证：
- [ ] 脚本输出的所有路由均已记录
- [ ] 请求/响应 Schema 与实际代码一致
- [ ] 认证要求已标注
- [ ] 示例为合法 JSON
