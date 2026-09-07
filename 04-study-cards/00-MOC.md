---
module: deepseek-harness/00-overview
type: moc
keywords: [cordis, plugin, architecture, monorepo]
---

# 00 — Map of Content (MOC)

> tutor-skills Codebase Mode C5(Dashboard)产出 · DeepSeek Harness 教程入口

## 模块地图(7 大组 / 52 包)

| 组 | 包数 | 代表包 | 章节 |
|---|---|---|---|
| **core/** | 9 | session / agent / agent-loop / tools | [[02-cordis-harness#抽象 2-session会话]] |
| **llm/** | 6 | llm / llm-deepseek / llm-retry | [[02-cordis-harness#抽象 4-capability-seam能力接缝]] |
| **shell/** | 9 | shell / bash-local / bash-sandbox / tool-bash | [[02-cordis-harness#抽象 4-capability-seam能力接缝]] |
| **bundle/** | 6 | base / web-app / headless / sdk-app | [[01-plugin-system#2-3-bundle-层packages-bundle]] |
| **boot/** | 2 | cmdline / app-boot | [[01-plugin-system#2-1-cli-层packages-boot-cmdline]] |
| **api/** | 1 | api + typert | [[03-package-tour#6-api--typert类型--网关2-包]] |
| **其他** | 20+ | e2b / fs / web / subagent / workflow / skill ... | [[03-package-tour#7-其他能力20-包]] |

## 5 个核心抽象

1. **Cordis Plugin Tree** — 运行时,[[02-cordis-harness#2-抽象-1cordis-plugin-tree]]
2. **Session** — append-only event log,[[02-cordis-harness#3-抽象-2session会话]]
3. **Agent Loop** — 智能体循环,[[02-cordis-harness#4-抽象-3agent-loop智能体循环]]
4. **Capability Seam** — 三角色模式,[[02-cordis-harness#5-抽象-4capability-seam能力接缝]]
5. **Profile + Bundle** — 配置叠加,[[02-cordis-harness#6-抽象-5profile--bundle配置叠加层]]

## 推荐阅读路径

1. 📘 [00 总览 HTML](../00-overview.html) — 5 分钟建立 mental model
2. 🔌 [01 插件系统 HTML](../01-plugin-system.html) — 理解"全插件"怎么落地
3. 🧠 [02 Cordis + 5 抽象 HTML](../02-cordis-harness.html) — 抽象级讲解
4. 📦 [03 52 包巡礼 HTML](../03-package-tour.html) — 看具体包
5. 🃏 04 学习卡片(本目录)— 边读边测

## 关键不变量(从 AGENTS.md 提取)

> 改 packages/ 之前必读!来自 `AGENTS.md` L105-L112

- **Registrations are effects**:一切注册走 `ctx.effect()` / `ctx.on()`;`register()` 返回 disposer
- **Runtime invariants assert owned relationships**: 只在独立观察可分歧时 publish `./invariant`
- **Typed events use declaration merging**: 事件 JSDoc 需要 `@mode` + payload `@param`
- **Switch on discriminant tags**: 闭合 union 用 `assertNever`
- **Waterfall listeners MUST call `next()`**: 不调 = 短路整条链
- **Model-visible ⟺ logged**: 任何到模型请求的内容必须从 log 重建
- **Plugins, not loop changes**: 新行为走扩展点;改 `agent-loop` 必须更新 `docs/architecture.md`
- **Capability seam = 3 角色**: Definition / Provider / Consumer 缺一不可

## 相关链接

- [项目仓库](https://github.com/deepseek-ai/deepseek-harness)
- [官方文档](https://deepseek-harness.github.io/deepseek-harness/)
- [Cordis 框架](https://github.com/cordiverse/cordis)
- [上游 Cordis 论文 (arXiv 2608.25512)](https://arxiv.org/abs/2608.25512)
- 配套调研: [codebase-to-course-survey](../../workspace/hermes_coder/codebase-to-course-survey/readme.md)
