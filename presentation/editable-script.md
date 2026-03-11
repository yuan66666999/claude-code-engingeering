# AI 编程效率实战分享 — 演讲稿

> SubAgent / Skills / Commands 三大核心组件的实战应用

---

## 目录


| 章节     | 主题                | 内容                            | 时长      |
| ------ | ----------------- | ----------------------------- | ------- |
| Part 1 | 基础认知              | 三大组件是什么、解决什么问题                | ~10 min |
| Part 2 | SubAgent          | 单Agent / 并行Agent / Agent Team | ~30 min |
| Part 3 | Skills + Commands | 能力包 + 命令模板                    | ~25 min |
| Part 4 | 场景选型总结            | 决策树 + 一页总结                    | ~5 min  |


---

# Part 1: 基础认知 — 三大核心组件

## 1.1 三大组件架构定位

![三大组件架构定位](images/slide-1-1-architecture.png "三大组件架构定位")

三者在 Claude Code 中的位置：

- **Commands** = 用户手动触发的 Prompt 模板（`/commit`）
- **Skills** = LLM 自动推理触发的能力包（语义匹配）
- **SubAgent** = 隔离上下文的专职小助手（独立窗口执行）

Commands 和 Skills 共享主对话上下文，SubAgent 在隔离的上下文中执行。

## 1.2 三大组件对比

![三大组件对比表](images/slide-1-2-comparison-table.png "三大组件对比表")


| 维度   | Commands                | Skills              | SubAgent              |
| ---- | ----------------------- | ------------------- | --------------------- |
| 一句话  | 消灭重复输入                  | AI 自动判断何时做什么        | 隔离 + 分工 + 并行          |
| 触发   | `/命令名` 手动               | LLM 语义推理自动          | 指定调用或自动匹配             |
| 上下文  | 共享主对话                   | 共享主对话               | 隔离执行                  |
| 定义位置 | `.claude/commands/*.md` | `skills/*/SKILL.md` | `.claude/agents/*.md` |
| 适合场景 | 固定流程、团队共享               | 需要专业知识和策略           | 高噪声、强约束、可并行           |
| 类比   | 快捷键/宏                   | 专家大脑                | 专职员工                  |


## 1.3 三者如何协作？

**场景：提交代码前的质量保障**


| 步骤  | 组件           | 动作                           |
| --- | ------------ | ---------------------------- |
| 1   | **Commands** | 用户输入 `/commit`，触发提交流程        |
| 2   | **Skills**   | AI 自动加载「代码规范」Skill，提供审查策略    |
| 3   | **SubAgent** | 派出只读 Code Reviewer，在隔离上下文中审查 |
| 4   | **SubAgent** | 返回审查摘要到主对话                   |
| 5   | **Commands** | 确认无问题后执行 git commit          |


> Commands 管触发、Skills 管策略、SubAgent 管执行 — 各司其职

---

# Part 2: SubAgent 深入讲解

## 2.1 SubAgent 核心概念

![SubAgent核心概念](images/slide-2-1-subagent-concept.png "SubAgent核心概念")

子代理 = 一个"专职小助手"，带着自己的规则、工具权限和上下文窗口，去完成某一类任务，然后把"结果摘要"带回来。

三大特点：

- **隔离上下文** — 子代理干活的探索过程、冗长输出不污染主对话
- **强约束** — 你能规定它"只能读文件/只能跑测试/不能改代码"
- **可复用** — 做成文件后下次直接用，像工程资产

## 2.2 适用场景

![SubAgent适用场景](images/slide-2-2-subagent-scenarios.png "SubAgent适用场景")

**适合的 4 类场景：**


| 场景类型  | 典型任务          | 为什么用 SubAgent |
| ----- | ------------- | ------------- |
| 高噪声输出 | 跑测试、扫日志、全项目搜索 | 不污染主对话        |
| 角色边界  | 只读审查、只读 DB 查询 | 限制工具权限        |
| 并行研究  | 多模块探索、技术选型    | 同时执行提速        |
| 流水线分段 | 定位→分析→修复→验证   | 各阶段专人负责       |


**不适合的情况：** 频繁交互 / 强耦合上下文 / 极简任务

## 2.3 SubAgent 三种模式演进

![SubAgent三种模式演进](images/slide-2-3-subagent-evolution.png "SubAgent三种模式演进")


| 模式         | 子代理数 | 关系       | 代表案例                    |
| ---------- | ---- | -------- | ----------------------- |
| 单 Agent    | 1    | 主→子      | 01-code-reviewer        |
| 并行 Agent   | N    | 主→子x3 同时 | 04-parallel-explore     |
| Agent Team | N    | 多会话协作    | 06-agent-teams-bug-hunt |


---

## 案例一：Code Reviewer（单 Agent）

![案例一CodeReviewer](images/slide-2-4-code-reviewer.png "案例一：Code Reviewer（单Agent）")

> 演示代码：`demos/case1-code-reviewer/`

### Agent 定义文件

文件位置：`.claude/agents/code-reviewer.md`

```yaml
---
name: code-reviewer
description: Review code changes for quality, security, and best practices.
tools: Read, Grep, Glob, Bash          # 没有 Edit/Write → 只读
model: sonnet
---

You are a senior code reviewer.
**You are strictly read-only. NEVER modify any files.**

When Invoked:
1. Run `git diff` or read specified files
2. Analyze: Security → Performance → Maintainability → Best Practices
3. Report: Critical / Warning / Suggestion
```

关键设计：


| 配置项   | 值                               | 设计意图             |
| ----- | ------------------------------- | ---------------- |
| tools | Read, Grep, Glob, Bash          | 只读，不给 Edit/Write |
| model | sonnet                          | 平衡质量与速度          |
| 输出格式  | Critical / Warning / Suggestion | 分级报告             |


### 审查对象示例

被审查的 `src/auth.js` 故意包含多个安全问题：

```javascript
// 硬编码密钥
const JWT_SECRET = 'my-super-secret-key-123';

// 弱加密算法
function hashPassword(password) {
  return crypto.createHash('md5').update(password).digest('hex');
}

// 信息泄露
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack,           // 生产环境暴露堆栈
    dbConnection: config.db     // 暴露数据库配置
  });
});
```

### 使用方式

在主对话中提示：

> 用 code-reviewer 检查 src/auth.js 的安全问题

### 审查报告输出示例

```markdown
## Code Review Report

### Critical Issues
- [auth.js:3] 硬编码 JWT 密钥 → 应使用环境变量
- [auth.js:6] 使用 MD5 哈希密码 → 应使用 bcrypt 或 argon2

### Warnings
- [auth.js:11] 生产环境暴露错误堆栈和数据库配置

### Summary
- Critical: 2 | Warnings: 1 | Suggestions: 3
- Overall risk: HIGH
```

**核心卖点：** 角色边界约束 — 子代理不会"手贱"改你的代码

> **Demo 时间**

---

## 案例二：Parallel Explore（并行 Agent）

![案例二ParallelExplore](images/slide-2-6-parallel-explore.png "案例二：Parallel Explore（并行Agent）")

> 演示代码：`demos/case2-parallel-explore/`

### 三个子代理定义

三个文件结构完全相同，只是探索目标不同：

```yaml
# .claude/agents/auth-explorer.md
---
name: auth-explorer
description: Explore and analyze the authentication module
tools: Read, Grep, Glob          # 全部只读
model: haiku                      # 快速 + 便宜
---
Analyze the src/auth/ directory:
1. Architecture overview
2. Auth flow documentation
3. Security assessment
4. Dependencies mapping
Return a structured analysis report.
```


| 子代理           | 探索目标          | Model | 工具               |
| ------------- | ------------- | ----- | ---------------- |
| auth-explorer | src/auth/     | haiku | Read, Grep, Glob |
| db-explorer   | src/database/ | haiku | Read, Grep, Glob |
| api-explorer  | src/api/      | haiku | Read, Grep, Glob |


### 使用方式

在主对话中一句话即可：

> "帮我摸清这个项目的架构，分别分析 auth、database、api 三个模块"

Claude Code 自动：

1. 同时启动 3 个子代理（并行）
2. 每个子代理在隔离上下文中独立探索
3. 各自返回结构化报告
4. 主对话综合三份报告给出全景分析

**核心卖点：** 接手新项目时，3 分钟摸清整个代码库（3x 提速）

> **Demo 时间**

---

## Agent Teams 介绍

### 什么是 Agent Teams？

与 SubAgent 的本质区别：


| 维度  | SubAgent              | Agent Teams    |
| --- | --------------------- | -------------- |
| 会话  | 独立窗口，单次任务             | 多会话持续协作        |
| 通信  | 不互通，只回报主对话            | **互相发消息、共享发现** |
| 协作  | 各干各的                  | **辩论、挑战假设**    |
| 定义  | `.claude/agents/*.md` | 自然语言创建团队       |


> 核心价值：**当问题需要跨视角协作时，并行独立调查不如团队辩论**

### 启动方式


| 步骤      | 操作                                              |
| ------- | ----------------------------------------------- |
| 1. 开启功能 | `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` |
| 2. 创建团队 | 告诉 Claude "创建一个 agent team，生成 N 个 teammates"    |
| 3. 分配角色 | 每个 teammate 指定调查方向和假设                           |
| 4. 协作调查 | teammates 自动执行、共享发现、互相挑战                        |


**交互操作**：

- `Shift + ↑/↓` 在 Teammates 之间切换
- `Ctrl + T` 查看共享任务列表
- `Shift + Tab` 进入 Delegate Mode（Lead 只协调不分析）

**三种启动模式**：

1. **完整竞争假设模式**（推荐）：指定每个 teammate 的假设方向，要求互相挑战
2. **Delegate Mode**：Lead 只负责协调，不自己分析代码
3. **Plan Approval**：要求 teammate 提交调查计划等待审批后再执行

### 适用场景


| 适合 Agent Teams         | 不适合（用普通 SubAgent 即可） |
| ---------------------- | -------------------- |
| 多个假设需要**竞争验证**         | 任务目标明确、无争议           |
| Bug 之间可能有**关联/级联**     | 彼此独立的任务              |
| 需要**跨领域视角**（前后端/DB/缓存） | 单一领域的分析              |
| 需要**辩论和挑战**才能收敛        | 一个人就能做的审查            |
| 复杂系统的**架构评审**          | 简单的代码审查              |


> 判断标准：**是否需要多人"吵一吵"才能得出结论？**

---

## 案例三：Agent Team Bug Hunt

![案例三AgentTeam](images/slide-2-8-agent-team.png "案例三：Agent Team Bug Hunt")

> 演示代码：`demos/case3-agent-team-bug-hunt/`

### 问题场景

ShopStream 电商平台，用户报告三个症状：


| 优先级    | 症状     | 表现                 |
| ------ | ------ | ------------------ |
| **P0** | 数据泄漏   | 用户 A 看到用户 B 的订单    |
| **P1** | 会话丢失   | 登录后随机掉线            |
| **P2** | API 变慢 | 响应时间从 200ms 飙升到 5s |


### Bug 级联因果分析

![Bug级联因果分析](images/slide-2-9-bug-cascade.png "Bug级联因果分析")

4 个相互关联的 Bug 形成级联：


| Bug  | 位置         | 问题          | 导致的症状       |
| ---- | ---------- | ----------- | ----------- |
| Bug1 | db.js      | DB 连接池过小    | 触发 Bug2/3/4 |
| Bug2 | session.js | Redis 不处理重连 | 会话丢失 (P1)   |
| Bug3 | orders.js  | 订单 N+1 查询   | API 变慢 (P2) |
| Bug4 | cache.js   | 缓存竞态        | 数据泄漏 (P0)   |


### Agent Team 角色设计

通过 `team-prompt.md` 定义 4 个调查角色：


| 角色         | 调查方向 | 切入点                  |
| ---------- | ---- | -------------------- |
| Session 侦探 | 会话管理 | Redis 连接、session 中间件 |
| 数据库侦探      | 数据层  | 连接池、查询性能             |
| 缓存侦探       | 缓存机制 | 缓存竞态、key 设计          |
| 架构侦探       | 全局视角 | 中间件链、请求生命周期          |


各角色**共享发现、互相挑战假设**，最终合并为完整调查报告。

与 SubAgent 的区别：


| 对比   | SubAgent              | Agent Team       |
| ---- | --------------------- | ---------------- |
| 协作方式 | 独立执行，不互通              | 共享发现，互相挑战        |
| 会话   | 单次任务                  | 持续多轮协作           |
| 定义方式 | `.claude/agents/*.md` | `team-prompt.md` |


**核心卖点：** 复杂问题需要多视角协作

> **Demo 时间**

---

## SubAgent 小结


| 模式         | 子代理数 | 核心特点       | 适用场景        |
| ---------- | ---- | ---------- | ----------- |
| 单 Agent    | 1    | 角色隔离、权限约束  | 审查、测试、日志分析  |
| 并行 Agent   | N    | 同时执行、效率翻倍  | 多模块探索、方案比较  |
| Agent Team | N    | 多角色协作、共享发现 | 复杂问题调查、架构评审 |


> SubAgent = 专职员工，给明确任务，限定权限，隔离执行

---

# Part 3: Skills + Commands

## 3.0 什么是 Skills？— 让模型在正确的时刻拥有正确的知识

![什么是Skills](images/slide-3-0-what-is-skills.png "什么是Skills — 从人调度模型到模型调度能力")

### 一个现实问题

每个团队都有大量的"隐性知识"——代码规范散落在 Wiki 里，部署流程记在老员工脑子里，排查经验埋在历史 Issue 中。新人上手靠"师傅带"，AI 上手靠"你喂 Prompt"。

问题在于：

- 上下文窗口是有限的，不可能把所有知识一次性塞给模型
- 什么时候该用哪块知识，过去全靠人来判断和调度
- 当知识规模和场景组合持续增长，"人预编排一切"的模式开始失效

**Skills 解决的核心问题：在有限的上下文窗口中，让 Agent 在正确的时刻拥有正确的领域知识。**

### Skills 到底是什么？

用一个类比帮助理解：

| 类比           | 说明                                   |
| ------------ | ------------------------------------ |
| 传统做法 = 纸质手册  | 所有流程印成厚厚的操作手册，用不用、翻到哪页，全靠人自己判断      |
| Prompt = 口头嘱咐 | 每次开工前口头交代一遍注意事项，说漏了就没了              |
| **Skill = 智能工具箱** | 工具箱自己知道"现在该递扳手还是螺丝刀"，按需打开正确的那一格 |

具体来说，一个 Skill 包含三层内容：

1. **description（触发条件）** — 告诉模型"什么情况下该加载我"，模型通过语义推理自动匹配
2. **正文（执行流程）** — 将抽象的原则和经验转化为可执行的步骤
3. **附录（参考资料）** — 模板、脚本、检查清单，按需加载不浪费 token

### 为什么说 Skills 是一种范式转变？

过去：**人调度模型**

```
人判断场景 → 人选择知识 → 人编写 Prompt → 模型执行
```

现在：**模型调度能力**

```
模型理解意图 → 模型匹配 Skill → 自动加载知识 → 按流程执行
```

这个转变的意义：

| 维度     | 过去（人调度模型）           | 现在（模型调度能力）           |
| ------ | ------------------- | -------------------- |
| 知识获取   | 人翻文档、问同事            | 模型按语义自动匹配 Skill      |
| 流程执行   | 人编排步骤写进 Prompt      | Skill 内置标准化流程        |
| 质量标准   | 靠经验、靠记忆             | Skill 定义"什么算数、什么标准成立" |
| 扩展性    | 场景越多，Prompt 越复杂     | 新增 Skill 即可，互不干扰     |
| 可复用性   | 每次重新写 Prompt        | 一次编写，整个团队复用          |

### 一句话总结

> **Skills 不是教模型怎么干活，而是定义在你的项目里：什么行为算数，什么标准成立，什么判断合理。它是连接"行动能力"与"语义世界"的中间层 — 一种让通用模型具备专业化、按需调用能力的通用设计模式。**

---

## 3.1 Skills vs SubAgent

![Skills vs SubAgent对比](images/slide-3-1-skills-vs-subagent.png "Skills vs SubAgent对比")


| 维度       | Skills             | SubAgent              |
| -------- | ------------------ | --------------------- |
| 类比       | "怎么做"的专家手册         | "谁去做"的专职员工            |
| 上下文      | 共享主对话              | 隔离执行                  |
| 触发       | LLM 语义推理自动         | 显式调用或描述匹配             |
| 定义       | `SKILL.md` + 渐进式加载 | `.claude/agents/*.md` |
| 核心内容     | 知识 + 策略 + 模板       | 角色 + 权限 + 任务          |
| Token 开销 | 按需加载，省 token       | 独立上下文窗口               |


**一句话：SubAgent 是"谁去做"，Skills 是"怎么做"。**

## 3.2 Skills 渐进式加载

![Skills渐进式加载](images/slide-3-2-progressive-loading.png "Skills渐进式加载")


| 层级           | 内容                       | Token 消耗    |
| ------------ | ------------------------ | ----------- |
| 第1层：目录页      | 扫描所有 Skill 的 description | ~100 tokens |
| 第2层：SKILL.md | 核心流程和规则                  | <5k tokens  |
| 第3层：附录按需     | 模板/脚本/参考                 | 只加载被引用的     |


Token 节省：~98%（不需要时不加载）

## 3.3 Skill vs SubAgent 定义文件对比

![Skill与SubAgent定义文件对比](images/slide-3-4-skill-vs-agent-code.png "Skill与SubAgent定义文件对比")


| 对比项         | SubAgent (.claude/agents/*.md)  | Skill (SKILL.md)                 |
| ----------- | ------------------------------- | -------------------------------- |
| frontmatter | name, description, tools, model | name, description, allowed-tools |
| body        | 角色设定 + 任务指令                     | 知识 + 流程 + 输出模板                   |
| 触发          | "用 xxx 子代理做..."                 | AI 自动判断                          |
| 执行环境        | 独立上下文窗口                         | 当前对话中                            |


---

## 案例四：基础 Skill 编写

> 演示代码：`demos/case4-basic-skill/`

最小可用的 Skill 文件 — `.claude/skills/code-reviewing/SKILL.md`：

```yaml
---
name: code-reviewing
description: Perform comprehensive code review for quality and security
allowed-tools: Read, Grep, Glob
---

## When to use
When the user asks to review code quality, check for security issues,
or assess code health.

## Process
1. Identify target files or directories
2. Check: security → performance → maintainability
3. Output structured report with severity levels

## Output Format
- Summary → Issues (Critical/Warning/Info) → Recommendations
```

**关键：** `description` 写清楚触发条件，LLM 根据语义自动匹配。

### Skill 的触发方式 vs Command


| 维度   | Command 触发（显式）        | Skill 触发（隐式）            |
| ---- | --------------------- | ----------------------- |
| 用户输入 | `/review src/auth.js` | "帮我看看这段代码有没有安全问题"       |
| 匹配方式 | 精确匹配命令名               | LLM 语义推理，匹配 description |
| 执行时机 | 用户主动调用                | AI 自动判断                 |
| 适合   | 已知要做什么                | 不确定用哪个能力                |


> 同样的能力，不同的触发方式 — 按需选择

---

## 案例五：SubAgent + Skill 组合

![SubAgent+Skill组合实战](images/slide-3-5-agent-skill-combo.png "SubAgent+Skill组合实战")

> 演示代码：`demos/case5-agent-skill-combo/`

### 分工设计


| 职责       | 由谁定义     | 具体内容                     |
| -------- | -------- | ------------------------ |
| WHO      | SubAgent | 角色：API 文档专员              |
| WHAT     | SubAgent | 使命：生成 API 文档             |
| WHERE    | SubAgent | 输出：docs/api/             |
| HOW      | Skill    | 流程：扫描 → 分析 → 生成          |
| WITH     | Skill    | 工具：detect-routes.py      |
| STANDARD | Skill    | 模板：api-doc.md、OpenAPI 规范 |


### 为什么这样分工？

**SubAgent 单独使用的问题：**

- 每次都要在 prompt 里写完整的执行流程
- 脚本、模板硬编码在 agent 定义中
- 不同 agent 需要相同能力时，代码重复

**加入 Skill 后：**

- SubAgent 管调度（谁、做什么、输出到哪）
- Skill 管执行策略（怎么做、用什么工具、遵循什么标准）
- 关注点分离，各自可独立复用

> **Demo 时间：** 启动 SubAgent → 自动加载 Skill → 运行 detect-routes.py → 生成文档

---

## 案例六：常用 Commands

![Commands能力速览](images/slide-3-3-commands-overview.png "Commands能力速览")

> 演示代码：`demos/case6-commands/` + `demos/case6-commands-advanced/`

### Commands 能力速览


| 能力    | 语法                           | 示例                              |
| ----- | ---------------------------- | ------------------------------- |
| 参数    | `$ARGUMENTS` / `$1` `$2`     | `/commit fix login bug`         |
| 工具预授权 | `allowed-tools:`             | `Bash(git commit:*)`            |
| 模型选择  | `model:`                     | `haiku` (快+省)                   |
| 文件引用  | `@file`                      | `@docs/standards.md`            |
| 命令预处理 | `!`cmd``                     | `!`git log --oneline -10``      |
| Hooks | `PreToolUse` / `PostToolUse` | 部署前警告                           |
| 命名空间  | 子目录                          | `git/commit.md` → `/git:commit` |


### `/commit` 命令

```yaml
---
description: Quick git commit with auto-generated or specified message
argument-hint: [optional: commit message]
allowed-tools: Bash(git status:*), Bash(git add:*),
               Bash(git commit:*), Bash(git diff:*)
model: haiku
---

Create a git commit.
If message provided: use $ARGUMENTS
If no message: analyze git diff, auto-generate message

Steps:
1. git status → 2. git add . → 3. git diff --staged → 4. git commit

Commit format: feat: / fix: / docs: / refactor: ...
```

使用方式：

- `/commit` — 自动分析变更生成提交信息
- `/commit fix login bug` — 使用指定信息

### `/pr-create` 命令（高级）

```yaml
---
description: Create a pull request with context
argument-hint: "title" "description"
allowed-tools: Bash(git:*), Bash(gh:*)
model: sonnet
---

Current branch: !`git branch --show-current`         ← 预处理命令
Commits to merge: !`git log origin/main..HEAD --oneline`
Changed files: !`git diff --stat origin/main`

Create PR with title: $1  and description: $2         ← 多参数
```

三个高级能力：

- `!`cmd`` — 命令预处理，执行命令嵌入结果
- `$1` `$2` — 多参数支持
- `allowed-tools` — 工具预授权，免手动确认

> **Demo 时间**

---

## Skills + Commands 小结


| 维度   | Commands      | Skills          |
| ---- | ------------- | --------------- |
| 核心思想 | Prompt 模板化    | 能力包化            |
| 触发   | 手动 `/命令`      | AI 自动推理         |
| 上下文  | 共享主对话         | 共享主对话           |
| 组合   | 可以调用 SubAgent | 可以被 SubAgent 加载 |
| 最佳场景 | 团队标准化流程       | 需要自动匹配的专业知识     |


> Commands = 效率工具，Skills = 智能增强

---

# Part 4: 场景选型总结

## 4.1 决策流程

![场景选型决策树](images/slide-4-1-decision-tree.png "场景选型决策树")

1. **需要隔离上下文？**（高噪声/强约束/并行）→ **SubAgent**
  - 还需要专业知识/执行策略？→ **SubAgent + Skill 组合**
2. **需要 AI 自动判断何时触发？** → **Skills**
3. **是可复用的固定流程？** → **Commands**
4. 以上都不是 → 直接对话即可

## 4.2 一页总结

![一页总结](images/slide-4-2-summary.png "一页总结")


| 组件               | 核心价值      | 典型场景        | 记忆口诀      |
| ---------------- | --------- | ----------- | --------- |
| Commands         | 消灭重复输入    | 提交、PR、格式化   | "快捷键"     |
| Skills           | AI 自动匹配能力 | 代码规范、API 生成 | "专家大脑"    |
| SubAgent         | 隔离分工并行    | 审查、测试、日志分析  | "专职员工"    |
| SubAgent + Skill | 最强组合      | 调度 + 策略     | "经理 + 顾问" |


> 三者不互斥，按场景组合使用，效率最大化

---

## 实战案例回顾


| 案例                  | 组件                | 核心要点             |
| ------------------- | ----------------- | ---------------- |
| Code Reviewer       | SubAgent (单)      | 只读约束、角色边界、分级报告   |
| Parallel Explore    | SubAgent (并行)     | 3 个子代理同时探索、效率 3x |
| Agent Team Bug Hunt | SubAgent (团队)     | 多角色协作、共享发现、挑战假设  |
| Basic Skill         | Skills            | 5 分钟编写、AI 自动触发   |
| Agent + Skill Combo | Skills + SubAgent | 调度与策略分离、最强组合     |
| /commit, /pr-create | Commands          | 团队标准化、一键执行       |


---

## 核心 Takeaway

- **Commands = 快捷键** — 消灭重复输入
- **Skills = 专家大脑** — AI 自动匹配能力
- **SubAgent = 专职员工** — 隔离分工并行

三者不互斥，按场景组合使用，效率最大化。

---

## 参考资源


| 资源               | 链接                                     |
| ---------------- | -------------------------------------- |
| Claude Code 官方文档 | code.claude.com/docs                   |
| SubAgent 官方文档    | code.claude.com/docs/en/sub-agents     |
| Skills 官方文档      | code.claude.com/docs/en/skills         |
| Commands 官方文档    | code.claude.com/docs/en/slash-commands |
| 官方 Skills 仓库     | github.com/anthropics/skills           |


