# DeepSeek Harness 源码教程

> **方法论验证项目**:用 6 个"源码方法论 skill"实际跑一遍 deepseek-ai/deepseek-harness,产出可阅读的源码教程。

## 目录

```
deepseek-harness-tutorial/
├── index.html                 ← 入口(打开这个)
├── 00-overview.html           ← 总览:多视角架构图 + 5 抽象速览
├── 01-plugin-system.html      ← 插件系统(递归下钻视角)
├── 02-cordis-harness.html     ← Cordis 框架 + 5 个核心抽象
├── 03-package-tour.html       ← 52 包巡礼(per-package 摘要)
├── 04-study-cards/            ← Obsidian 学习卡片
│   ├── 00-MOC.md
│   ├── 01-Quick-Reference.md
│   ├── 02-Exam-Traps.md
│   ├── 10-exercises-core.md
│   ├── 11-exercises-plugin.md
│   └── 99-self-review.md
└── assets/                    ← 预渲染的 Mermaid SVG(7 张)
```

## 用法

```bash
# 浏览器打开入口
xdg-open index.html

# 或在 Obsidian 打开 04-study-cards/ 做题
```

**完全离线可用** —— 7 张流程图都是预渲染的 SVG,不依赖 mermaid CDN。

## 方法论来源

| 教程文件 | 用的 skill | 链接 |
|---|---|---|
| 00 总览 | oh-my-mermaid | <https://github.com/oh-my-mermaid/oh-my-mermaid> |
| 01 插件 | oh-my-mermaid(递归 + 7 字段) | 同上 |
| 02 Cordis | PocketFlow-Codebase-Knowledge | <https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge> |
| 03 52 包 | agents-reverse-engineer | <https://github.com/GeoloeG-IsT/agents-reverse-engineer> |
| 04 卡片 | tutor-skills(Codebase Mode C1-C9) | <https://github.com/RoundTable02/tutor-skills> |
| 命名 "00-04" | Understanding-Optimism-Codebase | <https://github.com/joohhnnn/Understanding-Optimism-Codebase> |
| 流程图风格 | blueprint-mcp 思路 | <https://github.com/ArcadeAI/blueprint-mcp> |

完整调研报告:[`/home/lifenghu/workspace/hermes_coder/codebase-to-course-survey/`](../../workspace/hermes_coder/codebase-to-course-survey/)

## 项目基础

- **项目**: [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)
- **版本**: 2026-09 快照
- **规模**: 52 packages, TypeScript + Cordis plugin 框架
- **官方文档**: <https://deepseek-harness.github.io/deepseek-harness/>

## 已知限制

1. **02-cordis-harness-02.svg**(Turn flow 序列图)用了简化版 alias(原版用 `participant Loop` 会被 mermaid 11 解析错乱,改为 `participant L as Loop`)
2. **没接 LLM**: 5 抽象识别是手工读 AGENTS.md 后提取,不是 LLM 自动跑
3. **Python SDK / native 模块** 暂未深入,只列了 1-2 句
4. **04-study-cards** 是为 Obsidian 设计的(用 `[[wiki-link]]` / `> [!answer]-` 折叠块),普通 Markdown 查看器效果略差

## 自我评估

完整自检见 [04-study-cards/99-self-review.md](./04-study-cards/99-self-review.md)。
