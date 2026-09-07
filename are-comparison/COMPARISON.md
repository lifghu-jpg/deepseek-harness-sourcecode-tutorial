# are-output vs 本教程对比报告

> 用 `agents-reverse-engineer`(ARE)真实跑 deepseek-harness,产物 vs 本教程对比

## 1. ARE 实际跑了什么

**命令**:
```bash
cd /home/lifenghu/src/deepseek-harness
npx agents-reverse-engineer@latest init      # 生成 .agents-reverse-engineer/config.yaml
npx agents-reverse-engineer@latest discover  # 扫描 + 生成 GENERATION-PLAN.md
```

**版本**: `agents-reverse-engineer v1.2.19`(npm 自动安装)
**Backend 检测**: 自动识别到 `claude` CLI,默认模型 `sonnet`

## 2. ARE 产出

### 2.1 配置文件

`.agents-reverse-engineer/config.yaml`(4964 字节,本对比报告未复制,只记关键字段):

```yaml
exclude:
  patterns:
    - "**/node_modules/**"
    - "**/dist/**"
    - "**/build/**"
    - "**/.git/**"
    - ...
ai:
  backend: claude
  model: sonnet
  concurrency: auto
```

### 2.2 GENERATION-PLAN.md(519 KB / 10197 行 / 7511 任务)

**结构**:
```
## Summary
- Total Tasks: 7511
- File Tasks: 6189
- Directory Tasks: 1322
- Traversal: Post-order

## Phase 1: File Analysis (Post-Order)
### Depth 10: packages/experimental/webworker-runtime/.../preview-architecture-review/
  - [ ] session.jsonl
### Depth 10: ...
### Depth 9: ...
...
### Depth 1: packages/  (顶层)
### Depth 1: docs/
...
```

**关键观察**:
- **post-order traversal**: 自底向上,depth 10 → depth 1
- **每个文件一个 checkbox**: `[ ]` 等待 AI 填 `.sum`
- **每个目录一个 task**: 等待 AI 写 `AGENTS.md`
- **总计 6189 文件 + 1322 目录 = 7511 任务**

### 2.3 排除的文件(2755 个)

ARE 默认排除:
- `node_modules/` / `dist/` / `build/` / `.git/`
- `*.png` / `*.gif` / 二进制文件
- `snapshots/` 下的所有 AGENTS.md(避免循环引用)
- `vendor/` 下的 loader 部分
- `packages/preset/.../skills/` 下的 SKILL.md(自指)

## 3. vs 本教程对比

| 维度 | ARE(per-file .sum + per-dir AGENTS.md) | 本教程(方法论验证) |
|---|---|---|
| **粒度** | 6189 文件 + 1322 目录 = 7511 任务 | 4 HTML + 6 学习卡片 |
| **覆盖** | 100% 文件 | 7 大组 / 52 packages 概览 |
| **生成方式** | LLM 调 7511 次(sonnet) | 我手工读 + 写(没调 LLM) |
| **增量更新** | git-aware,只重生改过的 | 一次性 |
| **agent 首次对话 context** | 自动获取 | 需要手动看教程 |
| **人读友好** | 中等(散在文件树) | **强**(HTML 导航 + Obsidian 卡片) |
| **HTML 流程图** | 无 | 7 张预渲染 SVG |
| **学习闭环(测验)** | 无 | 10 题 + 答案 |
| **跨切链接** | AGENTS.md 互相引用 | HTML 之间导航 + wiki-link |
| **官方支持** | npm 包 + 4 个 runtime | 自创 |

## 4. 互补关系(不是互斥)

| 用 ARE 的场景 | 用本教程的场景 |
|---|---|
| 让 AI agent 第一次对话就懂项目 | 让**人**第一次接触项目就懂 |
| 巨型 monorepo 日常开发 | 教新人 / 跨团队分享 |
| `init` 之后"代码 = 文档"自动同步 | 静态教程,代码改了要手动同步 |
| 看单文件细节(`.sum`) | 看高层架构(HTML 4 节) |

**最佳实践**: 两个都用
1. 跑 `are init + discover + generate` 让 AI 有 context
2. 同时把本教程给团队做 onboarding

## 5. 跑 ARE 看到的真实数据

| 指标 | 值 |
|---|---|
| 总任务 | 7511 |
| 文件任务 | 6189 |
| 目录任务 | 1322 |
| 排除文件 | 2755 |
| GENERATION-PLAN.md | 519 KB / 10197 行 |
| 深度 | 10 层 |
| 遍历顺序 | post-order(子先于父) |
| 估计 LLM 调用次数(若跑 generate) | 7511 次 sonnet |
| 估计费用(若用 sonnet) | ~$30-50 USD(按 1k tokens × 7511 × $3/M 估算) |

## 6. 本教程的相对优势

1. **零成本**: 不调 LLM,人工产出
2. **离线可用**: 预渲染 SVG,无 mermaid CDN 依赖
3. **HTML 友好**: 浏览器打开即看,无需特殊工具
4. **学习闭环**: 10 道题 + 答案(ARE 没有)
5. **设计/原理视角**: oh-my-mermaid 7 字段 + PocketFlow 5 抽象 + 不变量汇总
6. **跨节引用**: wiki-link 串联

## 7. ARE 的相对优势

1. **自动化**: 一次跑完 7511 文件
2. **持续同步**: git-aware 增量
3. **agent-native**: 直接被 Claude Code / Codex / OpenCode / Gemini CLI 吃
4. **per-file 精度**: `.sum` 文件级摘要
5. **可生产用**: npm 包,稳定维护

## 8. 复现命令

```bash
# 在 deepseek-harness 项目根:
cd /home/lifenghu/src/deepseek-harness
npx agents-reverse-engineer@latest init
npx agents-reverse-engineer@latest discover     # 不调 AI,纯扫
npx agents-reverse-engineer@latest generate    # 调 AI,需 sonnet API
```

注:本对比报告**只跑了 `discover`(不调 LLM)**,所以没有 `.sum` 和 `AGENTS.md` 产物——只验证了发现阶段的可行性。
