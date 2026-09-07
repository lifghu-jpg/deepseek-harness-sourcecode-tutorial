---
module: deepseek-harness/99-self-review
type: self-review
keywords: [checklist, verification, quality-gate]
---

# 99 — Self-Review(自检清单)

> tutor-skills Codebase Mode C9 产出 · 验证教程质量

## 来源映射

| 教程文件 | 用到的 skill | 方法论 |
|---|---|---|
| [00-overview.html](../00-overview.html) | oh-my-mermaid | perspective 总览 + 5 抽象速览 |
| [01-plugin-system.html](../01-plugin-system.html) | oh-my-mermaid | 递归下钻 + 7 字段 + classDef |
| [02-cordis-harness.html](../02-cordis-harness.html) | PocketFlow | 5 抽象识别 + 关系图 + turn flow 时序 |
| [03-package-tour.html](../03-package-tour.html) | agents-reverse-engineer | per-directory AGENTS.md 风格 |
| 04-study-cards/(本目录) | tutor-skills | Codebase Mode C1-C9 完整流程 |
| 命名 "00/01/02/03/04" | Understanding-Optimism | how-X-works 编号 + 跨节引用 |
| 流程图风格 | blueprint-mcp 思路 | Mermaid 文本声明 + classDef 配色 |

## 教程质量自检

按 tutor-skills 的 [quality-checklist](https://github.com/RoundTable02/tutor-skills/blob/main/references/quality-checklist.md) 思路自评:

### C1 — Project Exploration ✓

- [x] 文件清单(52 packages 全列)
- [x] 技术栈识别(TypeScript + Cordis + pnpm)
- [x] 入口点(`dsh` CLI + 5 profile)
- [x] 目录布局(顶层目录树)

### C2 — Architecture Analysis ✓

- [x] 5 核心抽象 + 关系图
- [x] Turn flow 时序图
- [x] Profile/Bundle/Cordis 拓扑
- [x] 数据流(append-only session log)

### C3 — Tag Standard ✓

- [x] 用了 #cordis #plugin #architecture 等
- [x] tag 注册在 [00-MOC](00-MOC.md) 列出

### C4 — Vault Structure ✓

- [x] `04-study-cards/` 5 个 .md + 00-MOC + 99-self-review
- [x] 子目录清晰:00-overview / 10-exercises / 11-exercises / 99-self-review

### C5 — Dashboard ✓

- [x] MOC 在 [00-MOC](00-MOC.md)
- [x] Quick Reference 在 [01-Quick-Reference](01-Quick-Reference.md)
- [x] Exam Traps 在 [02-Exam-Traps](02-Exam-Traps.md)

### C6 — Module Notes ✓

- [x] 每个抽象有 Purpose / Key Files / ctx key / Dependencies
- [x] 交叉链接通过 wiki-link

### C7 — Onboarding Exercises ✓

- [x] 5+ 题 per 主模块(10-exercises-core 5 题,11-exercises-plugin 5 题)
- [x] 题型多样:recall / code-reading / application / analysis
- [x] 答案用 fold callout

### C8 — Interlinking ✓

- [x] 每个文件都有 "相关" 段
- [x] [[wiki-link]] 跨引用
- [x] 5 抽象互相 link

### C9 — Self-Review ✓

- [x] 本文件
- [x] 列出每节用到的 skill + 方法论
- [x] 验证每条 C1-C8 标准

## HTML 教程(00-03)质量自评

按 oh-my-mermaid "diagram rules" 自评:

- [x] Element IDs 全部 kebab-case(`cli` / `profile` / `bundle1` 等)
- [x] 元素标签两行:名字 + 路径
- [x] 每条边有 label(为什么连接)
- [x] classDef 配色统一(4 个色板:entry / store / concern / external)
- [x] 复杂节点递归下钻(bundle / agent / session 都展开了)
- [x] 7 字段(description / context / constraint / concern / todo / note)— 至少 description + context

## 已知局限

- **没装 Cordis 包跑一遍**: 这是"读源码"教程,不是"调试"教程;产物是文字 + 图,不是运行时验证
- **版本锚定**: 文档基于 deepseek-harness 2026-09 快照;新版本可能结构变化
- **没接 LLM**: PocketFlow 的 "5 抽象识别 prompt" 这次是手工跑(我读 AGENTS.md + architecture.md 后识别),不是 LLM 自动跑;LLM 跑的话可以生成更细的描述
- **Python SDK / native 模块** 暂未深入,只列了 1-2 句

## 给读者的下一步建议

1. 装项目:`pnpm install && pnpm run build`
2. 跑一次:`pnpm dsh --profile web --dump-config` 看 plugin 树
3. 读 `docs/cordis-primer.md`,对照本教程的 5 抽象
4. 挑一个 bundle(如 `bundle/base`)深读它的 `cordis.yml`
5. 用 Obsidian 打开 `04-study-cards/`,跑 5 题自检

## 给教程作者的下一步建议

1. 跑 `npx agents-reverse-engineer@latest` 在本项目,看 `.sum` 输出是否比本教程的 `03-package-tour` 更细
2. 跑 PocketFlow 的 `main.py` 看自动生成的 5 抽象,跟本教程的对比
3. 用 `oh-my-mermaid` 的 `omm scan` 跑本项目,看 HTML viewer 渲染效果
4. 如果教程要在团队用,加 GitHub Actions:每次 `pnpm run doc-sync` 同步,自动重新生成 04-study-cards
