# 演示文稿使用说明

本演示文稿使用 [Marp](https://marp.app/) 格式，可导出为 PPT / PDF / HTML。

## 方式一：VS Code / Cursor 插件（推荐）

1. 安装插件 **Marp for VS Code**（搜索 `marp-team.marp-vscode`）
2. 打开 `slides.md`
3. 点击右上角预览按钮即可预览 Slide
4. 导出：`Ctrl+Shift+P` → 输入 `Marp: Export Slide Deck` → 选择格式（PDF / PPTX / HTML）

## 方式二：Marp CLI

```bash
# 安装
npm install -g @marp-team/marp-cli

# 导出 PDF
marp slides.md --pdf --allow-local-files

# 导出 PPTX
marp slides.md --pptx --allow-local-files

# 导出 HTML（可直接浏览器打开）
marp slides.md --html --allow-local-files

# 启动预览服务器
marp slides.md --server --allow-local-files
```

> `--allow-local-files` 参数是必须的，否则本地图片无法加载。

## 文件结构

```
presentation/
├── slides.md           ← 演示文稿主文件（Marp 格式）
├── HOW-TO-USE.md       ← 本说明文件
└── images/             ← Slide 图片素材
    ├── slide-1-1-architecture.png
    ├── slide-1-2-comparison-table.png
    ├── slide-2-1-subagent-concept.png
    ├── slide-2-2-subagent-scenarios.png
    ├── slide-2-3-subagent-evolution.png
    ├── slide-2-4-code-reviewer.png
    ├── slide-2-6-parallel-explore.png
    ├── slide-2-8-agent-team.png
    ├── slide-2-9-bug-cascade.png
    ├── slide-3-1-skills-vs-subagent.png
    ├── slide-3-2-progressive-loading.png
    ├── slide-3-3-commands-overview.png
    ├── slide-3-4-skill-vs-agent-code.png
    ├── slide-3-5-agent-skill-combo.png
    ├── slide-4-1-decision-tree.png
    └── slide-4-2-summary.png
```

## Slide 总览（共 30 页）

| 页码 | 内容 | 类型 |
|------|------|------|
| 1 | 封面 | 标题页 |
| 2 | 今天的分享内容 | 目录 |
| 3 | Part 1 标题页 | 分隔页 |
| 4 | 三大组件架构定位（图片） | 图表页 |
| 5 | 三大组件对比（图片） | 表格页 |
| 6 | 三者如何协作 | 流程页 |
| 7 | Part 2 标题页 | 分隔页 |
| 8 | SubAgent 核心概念（图片） | 图表页 |
| 9 | 适用场景速查（图片） | 图表页 |
| 10 | 三种模式演进（图片） | 图表页 |
| 11 | 案例一标题页 | 分隔页 |
| 12 | Code Reviewer（图片） | 案例页 |
| 13 | Agent 定义文件 | 代码页 |
| 14 | 审查对象示例 | 代码页 |
| 15 | 审查报告输出 + Demo | 案例页 |
| 16 | 案例二标题页 | 分隔页 |
| 17 | Parallel Explore（图片） | 案例页 |
| 18 | 三个子代理定义 | 代码页 |
| 19 | 使用方式 + Demo | 案例页 |
| 20 | 案例三标题页 | 分隔页 |
| 21 | Agent Team 协作（图片） | 案例页 |
| 22 | 问题场景 | 表格页 |
| 23 | Bug 级联因果（图片） | 图表页 |
| 24 | 角色设计 + Demo | 表格页 |
| 25 | SubAgent 小结 | 总结页 |
| 26 | Part 3 标题页 | 分隔页 |
| 27-32 | Skills + Commands 内容 | 混合页 |
| 33 | Part 4 标题页 | 分隔页 |
| 34 | 决策树（图片） | 图表页 |
| 35 | 一页总结（图片） | 总结页 |
| 36 | 案例回顾 | 总结页 |
| 37 | Takeaway | 结束页 |
| 38 | 参考资源 | 信息页 |
| 39 | Thank You + Q&A | 结束页 |
