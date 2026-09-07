---
module: deepseek-harness/10-exercises-core
type: exercises
keywords: [exercises, core, session, agent, cordis]
---

# 10 — Core 抽象练习题

> 5 道题(代码阅读 + 概念 + 应用),覆盖 5 个核心抽象

---

## Q1: 概念题 — Cordis vs Spring IoC

**Type**: recall
**Difficulty**: medium

Cordis 框架和 Spring IoC 容器有什么关键区别?为什么 DeepSeek Harness 选择 Cordis 而不是 Spring?

<details>
<summary>💡 提示</summary>

想三件事:注册语义、effect 反向、组合能力。
</details>

> [!answer]- 答案
>
> **关键区别**:
>
> 1. **注册语义**: Spring 注册是"配置",卸载复杂;Cordis 注册是 **effect**,自带 disposer,卸载时自动反向
> 2. **回滚能力**: Spring 容器关闭是显式 lifecycle;Cordis 卸载 plugin = 自动反向所有 effect
> 3. **组合能力**: Spring 容器组合需要"父容器"模型;Cordis 支持 **spacetime composability**——按时间 / 空间维度组合 plugin
>
> **为什么选 Cordis**: dsh 是 pre-release,要快速迭代 + 允许用户替换任意 plugin(包括"核心")。Spring 的"扩展"模型不够灵活,Cordis 的"全插件"模型更贴合 dsh 的"everything-is-a-plugin"哲学。

---

## Q2: 代码阅读 — 追踪一个 `user/message` 事件

**Type**: code-reading
**Difficulty**: medium

读 `packages/core/session/src/`。当一个 `user/message` 事件被追加时,代码路径是什么?哪些插件可能 observe 它?

<details>
<summary>💡 提示</summary>

看 `store.ts` 的 `append` 方法 + `index.ts` 的注册。
</details>

> [!answer]- 答案
>
> 1. 事件创建 → `SessionEvent` 对象构造
> 2. `store.append(event)` → 写 SQLite(append-only)+ 更新内存投影
> 3. 内存 store 触发 `session/event` 频道
> 4. observers 收到事件:
>    - `core/agent-loop` 监听到 → 触发 `agent/inbox` 唤醒 agent
>    - `session/project` 监听 → 更新 query 用内存视图
>    - 任何注册了 `ctx.on('session/event')` 的 plugin
>
> 注意:事件本身是**持久化**的(SQLite),不是只在内存里。

---

## Q3: 应用题 — 设计一个新的 Capability Seam

**Type**: application
**Difficulty**: hard

设计一个"图像生成" capability seam,要求完整 3 角色。给每个角色写接口签名。

<details>
<summary>💡 提示</summary>

参考 `shell/` 的结构:`shell/shell`(Definition)+ `bash-local`(Provider)+ `tool-bash`(Consumer)。
</details>

> [!answer]- 答案
>
> ```ts
> // 1. Definition: packages/imagegen/imagegen/src/definition.ts
> export interface ImageGenService {
>   generate(prompt: string, opts?: ImageGenOpts): Promise<GeneratedImage>;
> }
>
> // 2. Provider: packages/imagegen/openai-dalle/src/index.ts
> export class DalleProvider implements ImageGenService {
>   async generate(prompt: string, opts?: ImageGenOpts) {
>     // call OpenAI DALL-E API
>   }
> }
>
> // 3. Consumer: packages/imagegen/tool-imagegen/src/index.ts
> export const toolImageGen = defineTool({
>   name: 'image_generate',
>   description: 'Generate image from text prompt',
>   uses: 'imagegen',  // ctx.imagegen
>   schema: z.object({ prompt: z.string() }),
>   execute: async ({ prompt }, ctx) => {
>     return await ctx.imagegen.generate(prompt);
>   }
> });
> ```
>
> 然后在 `bundle/base` 里挂载 Definition + 一个默认 Provider,让 LLM 通过 tool 调用。

---

## Q4: 概念题 — 为什么 Capability Seam 必须 3 角色完整?

**Type**: analysis
**Difficulty**: hard

如果只写 Definition + Consumer(没 Provider),系统会怎样?反之(只 Provider + Consumer,没 Definition)?

<details>
<summary>💡 提示</summary>

把"契约"、"实现"、"使用"分开想。试着在 `dsh --dump-config` 里找有缺陷的 seam。
</details>

> [!answer]- 答案
>
> **只 Definition + Consumer(无 Provider)**:
> - Definition 声明能力存在,Consumer 想用
> - 但 ctx.imagegen 是 undefined → Consumer 调用时 `TypeError: Cannot read property 'generate' of undefined`
> - 错误很晚才暴露(运行时),不是启动时
>
> **只 Provider + Consumer(无 Definition)**:
> - Provider 实现了某些接口,但接口契约没有"标准"声明
> - 多个 Provider 之间无统一约定,容易"我说我的接口,你说你的"
> - Consumer 需要知道具体哪个 Provider,无法"按 seam 找"
>
> **完整 3 角色 = 契约 + 实现 + 使用三方对齐**,改任意一方只影响"那一角色",其他两方不用动。

---

## Q5: 应用题 — Agent Loop 的 Turn 状态机

**Type**: application
**Difficulty**: hard

读 `docs/architecture.md` 的 "Turn flow" 段。画出 turn 的状态机(包括 reject、empty first claim、startsRequestSeries 这几个分支)。

<details>
<summary>💡 提示</summary>

起点: `turn/start`;终点: `turn/end`。中间可能 0 step、1+ step、或被 reject。
</details>

> [!answer]- 答案
>
> ```
> turn/start
>   │
>   ├─ claim next-step input + 1 queued msg
>   │
>   └─> agent/pre-step (waterfall)
>        │
>        ├─ reject → turn/end (no step, but log the attempt)
>        │
>        ├─ empty first claim → turn/end (no step)
>        │
>        └─ enter(messages, startsRequestSeries?)
>             │
>             ├─ startsRequestSeries=true → 新的 model-message series
>             │   (log request/header, reason: "series" or "change")
>             │
>             └─ step/start
>                  ├─ log user/message
>                  ├─ derive model history from log
>                  ├─ agent/request → llm/stream → assistant/chunk* → assistant/message
>                  ├─ tool/call* → tools/pre-execute → tools/execute → tools/post-execute → tool/result*
>                  ├─ step/end
>                  │
>                  ├─ tools owe another request OR new input → claim → next step (loop back)
>                  │
>                  └─ done → agent/turn-stopping (serial) → turn/end
> ```

---

## 评分

- 5 题全对 = 深度掌握 5 个核心抽象
- 4/5 = 中级,可读代码但写新 plugin 还需查文档
- 3/5 = 入门,需要继续看 [[00-MOC]] 列出的其他抽象笔记

## 相关

- [[00-MOC]]
- [[11-exercises-plugin]] — 插件系统练习
- [[01-Quick-Reference]] — 速查表
- [[02-Exam-Traps]] — 常见误区
