# 演示代码索引

本目录包含分享中 6 个实战案例的源代码，按章节组织。

---

## Part 2: SubAgent 深入讲解

### 案例一：Code Reviewer（单 Agent）— Slide 12~15

**目录：** `case1-code-reviewer/`

| 文件 | 用途 |
|------|------|
| `.claude/agents/code-reviewer.md` | SubAgent 定义文件（展示 frontmatter + prompt 结构） |
| `src/auth.js` | 被审查的代码（故意包含安全漏洞，用于现场 Demo） |

**演示步骤：**
1. 打开 `code-reviewer.md` 讲解结构
2. 打开 `auth.js` 展示故意植入的安全问题
3. 在 Claude Code 中执行审查，展示输出报告

---

### 案例二：Parallel Explore（并行 Agent）— Slide 17~19

**目录：** `case2-parallel-explore/`

| 文件 | 用途 |
|------|------|
| `.claude/agents/auth-explorer.md` | 认证模块探索子代理 |
| `.claude/agents/db-explorer.md` | 数据库模块探索子代理 |
| `.claude/agents/api-explorer.md` | API 模块探索子代理 |

**演示步骤：**
1. 展示三个 agent 文件的共同结构（只读、haiku 模型）
2. 一句话启动并行探索
3. 展示三份报告同时返回的效果

---

### 案例三：Agent Team Bug Hunt — Slide 21~24

**目录：** `case3-agent-team-bug-hunt/`

| 文件 | 用途 |
|------|------|
| `bug-report.md` | Bug 报告（描述三个用户症状） |
| `team-prompt.md` | Agent Team 启动指令（定义 4 个侦探角色） |
| `findings-template.md` | 调查报告模板 |

**演示步骤：**
1. 展示 `bug-report.md` 中的症状描述
2. 展示 `team-prompt.md` 中的角色设计
3. 启动 Agent Team，展示多角色协作调查过程

---

## Part 3: Skills + Commands

### 案例四：基础 Skill 编写 — Slide 28~29

**目录：** `case4-basic-skill/`

| 文件 | 用途 |
|------|------|
| `.claude/skills/code-reviewing/SKILL.md` | 最小可用 Skill 定义文件 |

**演示步骤：**
1. 展示 SKILL.md 的 frontmatter 结构
2. 与案例一的 SubAgent md 做对比
3. 用自然语言触发 Skill，展示自动匹配效果

---

### 案例五：SubAgent + Skill 组合 — Slide 31~32

**目录：** `case5-agent-skill-combo/`

| 文件 | 用途 |
|------|------|
| `.claude/agents/api-doc-generator.md` | SubAgent 定义（WHO/WHAT/WHERE） |
| `.claude/skills/api-generating/SKILL.md` | Skill 定义（HOW/WITH/STANDARD） |
| `.claude/skills/api-generating/scripts/detect-routes.py` | 路由检测脚本 |
| `.claude/skills/api-generating/templates/api-doc.md` | 文档模板 |

**演示步骤：**
1. 并排展示 SubAgent 和 Skill 的分工
2. 启动 SubAgent，观察自动加载 Skill
3. 展示 detect-routes.py 运行和文档生成

---

### 案例六：常用 Commands — Slide 34~36

**目录：** `case6-commands/` + `case6-commands-advanced/`

**基础命令（case6-commands）：**

| 文件 | 命令 | 用途 |
|------|------|------|
| `.claude/commands/commit.md` | `/commit` | 快速 Git 提交 |
| `.claude/commands/review.md` | `/review` | 代码审查 |
| `.claude/commands/explain.md` | `/explain` | 代码解释 |
| `.claude/commands/todo.md` | `/todo` | 添加 TODO |
| `.claude/commands/git/status.md` | `/git:status` | Git 状态 |
| `.claude/commands/git/log.md` | `/git:log` | Git 日志 |

**高级命令（case6-commands-advanced）：**

| 文件 | 命令 | 亮点 |
|------|------|------|
| `.claude/commands/pr-create.md` | `/pr-create` | 多参数 `$1` `$2` + 预处理命令 |
| `.claude/commands/safe-deploy.md` | `/safe-deploy` | Hooks 支持 |
| `.claude/commands/analyze.md` | `/analyze` | 文件引用 `@file` |

**演示步骤：**
1. 展示 `/commit` 的 frontmatter（allowed-tools 预授权）
2. 现场执行 `/commit`
3. 展示 `/pr-create` 的高级特性（多参数、预处理命令）
