---
module: deepseek-harness/11-exercises-plugin
type: exercises
keywords: [exercises, plugin, profile, bundle, extension]
---

# 11 — 插件系统练习题

> 5 道题,覆盖 profile / bundle / cordis.yml / 扩展点

---

## Q1: 概念题 — 5 个 Profile 的区别

**Type**: recall
**Difficulty**: easy

`web` / `headless` / `sdk` / `sdk-minimal` / `acp` 这 5 个 profile 各自适合什么场景?为什么 `sdk-minimal` 不叠 `dsh-base`?

<details>
<summary>💡 提示</summary>

看 `bundle/` 里 6 个 bundle 的 README + `docs/architecture.md#profiles-and-bundles`。
</details>

> [!answer]- 答案
>
> | Profile | 场景 | 关键 Bundle |
> |---|---|---|
> | `web` | 浏览器 UI 调试 | base + web-app |
> | `headless` | 一次性跑任务 | base + headless |
> | `sdk` | JSON-RPC 远程调用 | base + sdk-app |
> | `sdk-minimal` | 极简 SDK | **sdk-minimal(不叠 base)** |
> | `acp` | Agent Client Protocol 自动化 | base + acp-app |
>
> **`sdk-minimal` 不叠 `dsh-base` 的原因**:
> - 它是一个**完整的显式 SDK 树**,不需要共享第一层
> - 共享层会让它被迫接受 base 的所有默认(model / tools / persistence),这违背"minimal"原则
> - 故意做例外,告诉读者"这个 bundle 自己说了算"

---

## Q2: 代码阅读 — 解析 cordis.yml

**Type**: code-reading
**Difficulty**: medium

读 `packages/bundle/base/cordis.yml`(或类似文件)。一个 plugin 行通常包含哪些字段?每个字段的作用?

<details>
<summary>💡 提示</summary>

最关键的字段是 `id`、`config`、`disabled`。
</details>

> [!answer]- 答案
>
> 典型 plugin 行:
>
> ```yaml
> plugins:
>   - id: deepseek-llm
>     config:
>       model: deepseek-chat
>       api_key: ${DEEPSEEK_API_KEY}
>   - id: session-storage
>     config:
>       backend: sqlite
>       path: ~/.dsh/sessions.db
>   - id: experimental-feature
>     disabled: true   # 临时禁用(用 !!js,不能 !js)
> ```
>
> 字段:
> - `id` — 唯一标识,patch 按这个匹配
> - `config` — 该 plugin 的配置(传给 effect 注册)
> - `disabled` — 临时关掉(注意:`!!js`,**不是** `!js`)
> - `dependencies` — 声明其他 plugin 作为前置
> - `overrides` — 强制覆盖上层 bundle 的同 id 配置
>
> **验证门**: `verify-cordis-config` 强制 raw/Web bundle 的 bare plugin 必须在 resolver manifest 的 `dependencies` 里。

---

## Q3: 应用题 — 写一个 patch

**Type**: application
**Difficulty**: medium

假设你要在用户的 Harness home 里把默认模型从 `deepseek-chat` 改成 `deepseek-coder`。写 `cordis.patch.yml` 的内容。

<details>
<summary>💡 提示</summary>

patch 按 id 匹配,**整行替换** config。
</details>

> [!answer]- 答案
>
> `~/.dsh/cordis.patch.yml`:
>
> ```yaml
> plugins:
>   - id: deepseek-llm
>     config:
>       model: deepseek-coder   # 覆盖默认
>       api_key: ${DEEPSEEK_API_KEY}  # 必须重复,否则被清空
> ```
>
> ⚠️ 注意:patch 是**整行替换**,不是字段合并。所以必须把 `api_key` 也写上,否则 env 变量读不到。

---

## Q4: 概念题 — Live patch reload 的边界

**Type**: analysis
**Difficulty**: hard

为什么 `web` profile 启用 live patch reload,但 `headless` / `sdk` 只在启动时应用一次?

<details>
<summary>💡 提示</summary>

想"替换运行时依赖"会作废什么样的生命周期?
</details>

> [!answer]- 答案
>
> **`web` 启 live reload 的原因**:
> - 浏览器应用是 long-lived server,改 patch 不需要重启
> - 用户的 home patch 在开发时频繁改
>
> **`headless` / `sdk` 不启 live reload 的原因**:
> - `headless` 是一次性 runner,跑完即结束,live reload 意义小
> - `sdk` 是 stdio / RPC server,可能已经"在做事"(处理一个长 RPC),替换它的依赖会让进行中的工作作废
> - 替换已拥有工作的进程的依赖 = 生命周期作废
>
> **设计原则**: 配置热重载 = 进程的运行时边界要和配置的运行时边界匹配。
> Long-lived service 可以 live reload;One-shot runner 不需要。

---

## Q5: 应用题 — 写自己的 Plugin

**Type**: application
**Difficulty**: hard

写一个 plugin,在每次 `user/message` 时打日志 + 计数。要求:用 effect(可逆),不要污染 session log。

<details>
<summary>💡 提示</summary>

```ts
ctx.effect(() => {
  // 注册
  return () => { /* 卸载 */ };
});
```
</details>

> [!answer]- 答案
>
> ```ts
> // packages/my-plugin/src/index.ts
> import type { Context } from '@deepseek-ai/cordis';
>
> export function applyMsgCounter(ctx: Context) {
>   let count = 0;
>   const disposers: Array<() => void> = [];
>
>   // 1. 注册 service(可查)
>   ctx.set('msgCounter', { get: () => count });
>
>   // 2. 监听事件(可逆)
>   const off = ctx.on('session/event', (e) => {
>     if (e.type === 'user/message') {
>       count++;
>       console.log(`[msg-counter] total user messages: ${count}`);
>     }
>   });
>   disposers.push(off);
>
>   // 3. 返回 disposer(plugin 卸载时自动调用)
>   return () => {
>     disposers.forEach(d => d());
>   };
> }
> ```
>
> 然后在 `cordis.yml` 里挂:
>
> ```yaml
> plugins:
>   - id: msg-counter
>     apply: '@deepseek-ai/dsh-msg-counter'
> ```
>
> **关键点**:
> - 计数变量**在 effect 闭包**里,不在 session log(避免污染)
> - `ctx.set('msgCounter', ...)` 让其他 plugin 能查
> - 返回 disposer,卸载时自动清理监听器
> - 不直接 import `session` 包,只 listen `session/event`(避免循环依赖)

---

## 评分

- 5/5 = 能独立写新 plugin
- 4/5 = 能改 patch 但写新 plugin 还需要模板
- 3/5 = 理解概念但要查文档才能动手

## 相关

- [[00-MOC]]
- [[10-exercises-core]] — 核心抽象练习
- [[01-Quick-Reference]] — 速查表
- [[02-Exam-Traps]] — 常见误区
