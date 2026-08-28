# Skill: Dispatch Task Card

## 何时触发

当 Tomz 使用下列表达时，默认采用本技能：

- “派卡” / “把这张卡派出去”
- “让施工线程做 Txxx / MOB-xxx”
- “给 OpenCode / Codex / Trae 施工”
- “这些任务哪些可以并行，然后派卡”
- “开施工线程” / “交给小弟做”

如果用户明确要求只转发原任务卡、不补上下文，则按本次明确指令执行。

## 核心原则

> **任务卡是工作合同，不是完整上下文。派卡不是把 Markdown 转发给另一个 Agent，而是为施工方构造一个可信的 Execution Context。**

即使模型能力很强，在缺少项目上下文时也会一本正经地跑偏。派卡前应主动补齐施工所需事实，而不是依赖施工 Agent 自己从任务标题脑补项目。

同时不要把任务卡膨胀成百科全书。稳定的项目经验应沉淀在项目文档、AGENTS / Skill / 架构说明中；任务卡只引用它们。

## 四层上下文

派卡前至少区分四层：

### 1. Project Context

长期稳定的项目事实，例如：

- 产品 / 系统边界；
- 架构职责和真相源；
- 已冻结合同；
- 目录、模块、平台关系；
- 明确禁止的改动；
- 已知但当前接受的技术债。

这些内容优先引用目标仓库当前的 `AGENTS.md`、架构文档、项目 Skill、README 或其他正式约定，不从旧聊天记忆反推。

### 2. Task Contract

当前任务卡本身负责：

- 背景；
- 目标；
- Scope / Out of scope；
- Acceptance；
- 依赖；
- 验证方式；
- 明确的风险与限制。

不要在派卡时擅自扩大 Scope，也不要因为施工 Agent 能力强就顺手重构无关区域。

### 3. Execution Context

这是派卡时必须现查的内容：

- 当前仓库与目标分支；
- 当前 base / HEAD；
- 与任务直接相关的真实代码入口；
- 相邻实现与调用关系；
- 当前测试 / build / CI 约定；
- 本任务可能触碰的共享状态、API、schema、route、token、native config 等。

**Execution Context 不能只靠任务卡提供，必须读取当前代码确认。**

### 4. Review / Experience Context

项目在长期 Review 中形成的稳定经验，例如：

- 常见误判；
- 高风险边界；
- 平台特有检查点；
- validation gap 与真实 bug 的区别；
- 什么情况下必须人工介入；
- 哪些历史决定不能重新讨论。

这类经验优先进入项目 Review Skill / Builder Skill / 项目文档，而不是每张任务卡重复复制。

## 派卡前必须做什么

### 1. 确认任务和施工目标

至少确认：

```text
repository
target branch / base
Task ID
任务卡路径
Builder 类型（如有指定）
是否允许并行
```

用户已经明确的信息不要重复询问。

### 2. 读取当前项目事实

优先读取：

```text
AGENTS.md / AGENT.md
work ledger / roadmap
对应 task card
相关架构 / contract 文档
项目级 Skill
相关代码与测试
```

只读与本任务有关的上下文，不做无边界全仓漫游。

### 3. 找出“施工方不知道就容易跑偏”的事实

重点识别：

- 真相源是谁；
- 哪些职责属于别的端 / 服务 /模块；
- 哪些合同已经冻结；
- 哪些行为看起来能优化但其实不能改；
- 哪些功能只是本地状态，哪些是 authoritative state；
- 哪些验证缺失只是 gap，不等于实现错误；
- 哪些跨平台 / 安全 / 发布边界与本任务有关。

### 4. 不确定就标出来，不要补脑

派卡内容中的信息分成三类：

```text
Verified Facts     已由仓库 / 用户明确确认
Hard Constraints   本次必须遵守
Open / Unknown     尚未确认，不允许施工方自行当成事实
```

如果一个未知项会改变实现方向，必须在施工前解决或明确要求 Builder 停止在该决策点，不得自行选一个“合理方案”继续。

## 标准派卡包

派给施工 Agent 的内容尽量保持短，但必须能定位完整上下文：

```text
# Task Dispatch

Task: MOB-xxx / Txxx
Repo: owner/repo
Base: dev @ <sha if available>
Goal: 一句话目标

## Must Read
- AGENTS.md
- docs/task-cards/...
- docs/contract/...
- relevant project Skill

## Verified Context
- 只写本任务真的需要知道的项目事实
- 尽量引用路径，不复制整篇文档

## Hard Constraints
- 不得破坏的合同
- Out of scope
- 禁止顺手重构的区域

## Execution Entry Points
- src/.../foo.ts
- src/.../bar.tsx
- tests/...

## Validation
- typecheck / lint / test / build
- 需要的 platform / device / integration evidence

## Unknown / Human Decision
- 没有则写 None

## Handoff
施工前先核对上述文件与当前代码；若仓库事实与本派卡冲突，以当前仓库事实为准，并先报告冲突，不要继续脑补施工。
```

## 并行派卡原则

并行不是 KPI。

两个任务即使不改同一个文件，也可能在以下位置产生语义竞态：

```text
API contract
shared types
global state
route registry
schema / migration
design token
permission model
package config
native config
generated files
```

因此：

> **无法证明可以并行，就默认串行。**

如果允许并行，派卡包还应记录：

- 共同 base SHA；
- 各自 worktree / branch；
- 潜在共享合同；
- 推荐集成顺序；
- 前一个任务合入后，后续任务是否必须 rebase / replay / 重新验证。

## 派卡后的施工原则

施工 Agent 收到派卡后，不应立即改代码。

先完成：

```text
读取 Must Read
↓
定位 Execution Entry Points
↓
核对任务卡与当前代码是否一致
↓
发现冲突则报告
↓
无冲突再施工
```

任务卡不能覆盖更新后的仓库事实；旧派卡也不能覆盖新的 HEAD。

## 检查点

派卡完成前检查：

- [ ] 仓库、分支、Task ID 正确；
- [ ] 对应任务卡确实存在；
- [ ] 已读取当前项目约定，而不是仅靠聊天记忆；
- [ ] 已定位真实代码入口；
- [ ] 已指出关键真相源和禁止越界区域；
- [ ] 未把未知推断写成项目事实；
- [ ] 验收与验证方式可执行；
- [ ] 若并行，已判断语义竞态而不只是文件冲突；
- [ ] 派卡包足够短，稳定上下文以引用为主而非复制。

## 安全与权限边界

- 派卡不等于扩大 Builder 权限。
- 不因为任务要求施工，就默认允许 push、merge、deploy、发布或修改仓库外资源。
- 不在派卡中写入 token、密码、cookie、私钥等秘密。
- 高风险动作仍按目标项目自己的权限规则执行。
- 如果用户明确限定“只改这一处 / 不改其他逻辑”，该限制必须进入 Hard Constraints。

## 冲突优先级

发生冲突时，按以下顺序处理：

```text
用户本次明确指令
↓
目标仓库当前正式合同 / AGENTS / task card
↓
当前代码与 CI 事实
↓
本 Playbook Skill
↓
历史聊天 / 旧经验
```

Playbook 的作用是减少重复解释，不是覆盖目标项目的当前事实。
