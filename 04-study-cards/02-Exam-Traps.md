---
module: deepseek-harness/00-exam-traps
type: exam-traps
keywords: [traps, pitfalls, common-mistakes]
---

# 02 — Exam Traps(常见误区)

> 改 dsh 时最容易踩的坑。贡献者必看。

## ❌ Trap 1:把 plugin 当普通模块 import

```ts
// ❌ 错 — 这会绕过 Cordis 注册
import { Session } from '@deepseek-ai/dsh-session';
const s = new Session();
```

```ts
// ✅ 对 — 通过 ctx 拿
ctx.on('app/ready', (ctx) => {
  const s = ctx.sessions;  // <-- 这才是注册的服务
});
```

## ❌ Trap 2:在 plugin 里写硬编码配置

```yaml
# ❌ 错 — 部署相关值不能写死
plugins:
  - id: my-llm
    config:
      api_key: sk-xxxxxx
      model: deepseek-chat
```

```yaml
# ✅ 对 — 走 Config 字段
plugins:
  - id: my-llm
    config:
      api_key: ${DEEPSEEK_API_KEY}    # env 注入
      model: ${MODEL_NAME:deepseek-chat}  # 带 default
```

**不变量**: *No hardcoded tunables in plugins* — 部署相关选择必须是 Config 字段。

## ❌ Trap 3:waterfall listener 不调 next

```ts
// ❌ 错 — 短路整条链
ctx.on('agent/pre-step', async (decision) => {
  return { ...decision, messages: [] };  // 没调 next
});
```

```ts
// ✅ 对 — 调 next 委托
ctx.on('agent/pre-step', async (decision, next) => {
  return await next(decision);  // 必须
});
```

**不变量**: *Waterfall listeners MUST call `next()`* — 不调 = 短路整条链。

## ❌ Trap 4:改 agent-loop 不更新文档

`docs/architecture.md` 列了 agent-loop 的 turn flow。任何改 agent-loop 行为的工作都必须更新这个文件。

```ts
// ❌ 错 — 改 agent-loop 流程但没更新 docs/architecture.md
function processStep(input) {
  // 新加了一段逻辑
  doExtraValidation(input);  // <-- 这会让文档过时
}
```

**不变量**: *Plugins, not loop changes* — 优先用扩展点;真要改 loop = 同时改文档。

## ❌ Trap 5:给老数据写兼容层

```ts
// ❌ 错 — dsh 处于 pre-release,不要写兼容
if (schemaVersion === 0) {
  return migrateV0ToV1(data);  // 不要做这个
}
```

```ts
// ✅ 对 — 拒绝老格式,要求用户重新生成
if (schemaVersion < CURRENT_SCHEMA_VERSION) {
  throw new Error(`Refusing to load old format v${schemaVersion}`);
}
```

**哲学**: *Foundation over blast radius* — pre-release 阶段优先正确的基础,不做兼容垫片。

## ❌ Trap 6:Capability Seam 只写 1-2 个角色

```ts
// ❌ 错 — 只有 Definition,没 Provider / Consumer
export interface ShellService { /* ... */ }
// 没有 Provider,也没有 Consumer
```

```ts
// ✅ 对 — 3 角色齐全
// 1. Definition
export interface ShellService { /* ... */ }
// 2. Provider
export class BashLocalProvider implements ShellService { /* ... */ }
// 3. Consumer
export const toolBash = defineTool({ uses: 'shell', /* ... */ });
```

**不变量**: *Capability seam = 3 角色* — Definition / Provider / Consumer 缺一不可。

## ❌ Trap 7:闭合 union 不用 assertNever

```ts
// ❌ 错 — 加新 case 时不会编译失败
function handle(event: 'a' | 'b' | 'c') {
  switch (event) {
    case 'a': return doA();
    case 'b': return doB();
    // 漏了 'c'
  }
}
```

```ts
// ✅ 对 — 加新 case 时会编译失败
function handle(event: 'a' | 'b' | 'c') {
  switch (event) {
    case 'a': return doA();
    case 'b': return doB();
    case 'c': return doC();
    default: return assertNever(event);
  }
}
```

**不变量**: *Switch on discriminant tags* — 闭合 union 用 `assertNever`。

## ❌ Trap 8:跳过 sandbox 验证

```bash
# ❌ 错 — sandbox 失败就禁用 sandbox
if ! sandbox-works; then
  DISABLE_SANDBOX=1  # 永远不要这样
fi
```

```
✅ 对 — sandbox 失败 = 测试失败,要求用户修
"Never bypass test failures or the product sandbox."
```

## ❌ Trap 9:不区分 source plane / artifact plane

```ts
// ❌ 错 — 测试用 build 后的 lib/,源码改了看不出
import { foo } from '../lib/index.js';
```

```ts
// ✅ 对 — 测试用 src/,通过 tsconfig paths
import { foo } from '../src/index.ts';  // tsconfig paths
```

**不变量**: *Source plane vs artifact plane, never mixed.*

## ❌ Trap 10:TypeScript 信任错了边界

```ts
// ❌ 错 — 同进程 typed 边界也加运行时校验
async function callApi(input: ApiInput): Promise<ApiOutput> {
  if (!isValidInput(input)) throw new Error();  // 多余
  return http.post(input);
}
```

```ts
// ✅ 对 — 只在跨边界处校验
// parser / config / queued / model / tool JSON / durable / file / worker / process / wire
```

**不变量**: *Trust TypeScript at typed same-process boundaries.*

## 相关链接

- [[00-MOC]] — 主索引
- [[01-Quick-Reference]] — 速查表
- [[10-exercises-core]] — 核心抽象练习题
- [[11-exercises-plugin]] — 插件系统练习题
