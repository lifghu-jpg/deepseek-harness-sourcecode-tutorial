---
module: deepseek-harness/00-quick-reference
type: quick-reference
keywords: [quick-ref, cheatsheet, cordis]
---

# 01 — Quick Reference

> 速查表:5 个抽象 + 关键 API + 常用命令

## Cordis 核心 API

```ts
// 1. 注册 service(可逆)
ctx.effect(() => {
  const svc = create();
  ctx.set('key', svc);
  return () => svc.dispose();
});

// 2. 监听 event
ctx.on('session/event', (e) => { /* ... */ });

// 3. Waterfall(必须 next)
ctx.on('agent/pre-step', async (decision, next) => {
  return await next(decision);
});

// 4. Serial(无 next)
ctx.on('agent/turn-stopping', (decision) => {
  // 不调 next
});
```

## 5 抽象 → ctx 键

| 抽象 | ctx 键 | 位置 |
|---|---|---|
| Cordis Plugin Tree | (运行时本身) | `vendor/cordis/` |
| Session | `ctx.sessions` | `packages/core/session/` |
| Agent | `ctx.agents` | `packages/core/agent/` |
| Agent Loop | `ctx.agentLoop` | `packages/core/agent-loop/` |
| System Prompt | `ctx.systemPrompt` | `packages/core/system-prompt/` |
| Tools | `ctx.tools` | `packages/core/tools/` |
| LLM | `ctx.llm` | `packages/llm/llm/` |
| Shell | `ctx.shell` | `packages/shell/shell/` |

## 5 个 Profile

| Profile | 入口 | Bundle 组合 |
|---|---|---|
| `web` | `dsh web` | base + web-app |
| `headless` | `dsh --profile headless` | base + headless |
| `sdk` | `dsh --profile sdk` | base + sdk-app |
| `sdk-minimal` | `dsh --profile sdk-minimal` | sdk-minimal(<b>不叠 base</b>)|
| `acp` | `dsh --profile acp` | base + acp-app |

## Bundle 叠加顺序(自下而上)

```
1. profile.bundles[i]    (按 profile 列出的顺序)
2. profile.cordis.patch.yml
3. home cordis.patch.yml  (用户 home)
4. --patch overlay        (命令行)
```

每层都能 patch 上一层的某一行(按 id 匹配)。

## 关键命令

```bash
# 跑起来
npx @deepseek-ai/dsh web                    # 浏览器
npx @deepseek-ai/dsh --profile headless "task"

# 从源码跑
pnpm install
pnpm run build
pnpm dsh web

# 看启动的 plugin 树
dsh --profile web --dump-config

# 跑测试
pnpm run test           # 单元
pnpm run test:e2e       # 真实 API
pnpm run test:coverage  # CI gate:per-file 100%
pnpm run typecheck
pnpm run lint

# 文档门禁
pnpm run doc-sync
```

## 事件 vs Effect 速记

| 类型 | 谁发 | 谁接 | 调 next? |
|---|---|---|---|
| Session events | 内核 | 持久化插件 | n/a(append-only) |
| Agent events | `ctx.agents` | observer | 部分 waterfall |
| Capability events | seam | adapter | 多数 waterfall |
| `agent/pre-step` | agent-loop | interceptor | **必须 next** |
| `agent/request` | agent-loop | llm adapter | **必须 next** |
| `llm/stream` | llm adapter | 监听器 | **必须 next** |
| `tools/pre-execute` | tools | guard | **必须 next** |
| `tools/execute` | tools | provider | **必须 next** |
| `tools/post-execute` | tools | 监听器 | **必须 next** |
| `agent/turn-stopping` | agent-loop | finalizer | **无 next(serial)** |

## 跟"普通应用"对比(读代码时的脑内模型)

| 普通应用 | dsh |
|---|---|
| 静态 main() | Cordis plugin tree(可热重载) |
| 写死的注册 | `ctx.effect()` 返回 disposer |
| 配置即代码 | 配置即 YAML,叠加多层 |
| 改核心 = 改应用 | 改核心 = 替换一个 plugin |
| 跑测试要重启 | dev profile live patch reload |

## 必读文件清单(按顺序)

1. `README.md` — 10 分钟
2. `AGENTS.md` — 仓库约定(尤其 packages 列表)
3. `docs/architecture.md` — 改 packages/ 必读
4. `docs/cordis-primer.md` — Cordis 入门
5. `docs/glossary.md` — 术语表
6. `docs/AGENTS.md` — 文档贡献
7. `docs/capability-seams.md` — seam 模式
8. `docs/agent-lifecycle.md` — agent 生命周期
