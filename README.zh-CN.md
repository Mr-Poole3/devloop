# 🔄 DevLoop

> 一个需求进去，一个经过验证、规格驱动、已归档的变更出来。

[English](README.md) | **[中文](README.zh-CN.md)** | [日本語](README.ja.md)

DevLoop 是一套整合式 AI 开发工作流，将[需求追问](https://github.com/mattpocock/skills)、[OpenSpec](https://github.com/Fission-AI/OpenSpec) 规格驱动开发、TDD、代码审查和归档合并为**一个自动化闭环** 🚀

你只需提供一个需求，DevLoop 自动驱动整个过程——澄清、探索、规格化、实现、测试、验证和归档。

---

## 🤔 为什么需要 DevLoop？

AI 编码 Agent 很强大，但也容易失控。常见问题：

| 问题 | 原因 | DevLoop 的解决方案 |
|------|------|-------------------|
| Agent 做了错误的东西 | 理解偏差 | 🔍 写代码前先追问澄清 |
| Agent 产出一团乱码 | 缺乏架构约束 | 📐 规格先行原则 + 模块规格追踪 |
| 没人知道做了什么决策 | 决策留在聊天记录里 | 📄 OpenSpec 工件作为持久事实 |
| 规格与代码脱节 | 没有验证步骤 | ✅ 归档前三层验证 |
| 小修复被过度文档化 | 一刀切流程 | ⚖️ 风险分级路由（L0-L3） |
| 中断后工作丢失 | 仅依赖会话状态 | 💾 `.state.yaml` 跨会话恢复 |

---

## ⚙️ 工作原理

```text
用户: /develop 我想增加暗黑模式支持

DevLoop:
  1. 读取项目上下文（配置、架构地图、领域语言）
  2. 风险分级 → L2（中等功能）
  3. 探索受影响模块，构建缺失的规格
  4. 追问用户，澄清每个决策分支
  5. 创建 OpenSpec 变更（proposal + specs + design + tasks）
  ── ⏸️ 用户确认需求理解 ──
  ── ⏸️ 用户确认方案和规格 ──
  6. 使用 TDD 实现（红-绿-重构）
  7. 发现规格缺口？→ 先更新 OpenSpec，再改代码
  8. 运行测试、类型检查、Lint
  9. 执行 /opsx:verify（规格完整性、正确性、一致性）
 10. 执行代码审查（标准 + 规格合规）
  ── ⏸️ 用户确认归档 ──
 11. 同步规格到正式注册表
 12. 归档变更
 13. 更新模块索引和架构地图
 14. 输出交付报告 📦
```

---

## 🛠️ 两个 Skill

### `devloop-setup` — 初始化

在任何仓库中设置、检查和修复 DevLoop 工作流。

```text
/devloop-setup              # 检查状态（默认）
/devloop-setup init         # 首次设置
/devloop-setup repair       # 修复缺失配置
/devloop-setup reconfigure  # 更新设置
```

**功能：**
- 🔎 自动检测技术栈（语言、框架、包管理器、测试/Lint/构建命令）
- 🗺️ 扫描源码目录，构建 L0 架构地图
- 📁 创建 `devloop/` 目录结构
- ⚙️ 初始化 OpenSpec 配置
- 📝 使用检测值生成配置文件
- 🔁 幂等——可安全重复运行

### `devloop` — 开发

驱动需求走完完整开发闭环。

```text
/develop 我想增加暗黑模式支持
/develop 修复登录重定向 Bug
/develop 将支付模块重构为事件驱动架构
/develop 为所有服务增加多租户隔离
```

**功能：**
- 💾 读取 `.state.yaml` 恢复中断的工作
- ⚖️ 根据升级因素进行风险分级（L0-L3）
- 🔥 追问用户，解决每个决策分支
- 📄 创建 OpenSpec 变更工件（proposal、specs、design、tasks）
- 🧪 使用 TDD 实现
- ✅ 对照规格验证
- 👁️ 审查代码质量
- 📦 归档并同步正式规格
- 📊 输出交付报告

---

## 🚀 快速开始

### 1. 安装 DevLoop skills

**方式 A — 通过 `npx skills` 安装（推荐）**

```bash
npx skills@latest add Mr-Poole3/devloop
```

CLI 会自动检测 `devloop-setup` 和 `devloop` 两个 skill，让你选择安装到哪些 Agent（Claude Code、Codex、OpenCode、TRAE、Cursor 等 50+ 宿主）。

也可以单独安装某个 skill：

```bash
npx skills@latest add Mr-Poole3/devloop/devloop-setup
npx skills@latest add Mr-Poole3/devloop/devloop
```

### 2. 在项目中运行初始化

```text
/devloop-setup init
```

这将创建 `devloop/` 目录、检测技术栈并生成所有配置。

### 3. 开始开发

```text
/develop <你的需求>
```

就这样，剩下的交给 DevLoop。🎉

---

## ⚖️ 风险等级

DevLoop 自动对每个需求进行风险分级，并选择相应重量的工作流。

| 等级 | 触发条件 | 工作流 | 确认点 |
|------|---------|--------|--------|
| **L0** | 无代码行为变化（文档、注释、配置修正） | 直接处理 | 无 |
| **L1** | 单模块、低风险修复 | `diagnose → tdd → review` | 开始、完成 |
| **L2** | 中等功能或重构 | `grill → OpenSpec → apply → verify → archive` | 需求、方案、归档 |
| **L3** | 跨模块、高风险、架构变更 | `grill-with-docs → explore → spec/tickets → OpenSpec → apply → verify → review → archive` | 需求、方案、开始编码、归档 |

**🚨 升级因素**（以下任意一项即至少升级到 L2）：

- 数据模型变化
- 公共 API 变化
- 权限、支付、隐私或用户数据
- 数据迁移
- 跨服务或跨模块影响
- 向后兼容要求
- 回滚要求
- 高技术不确定性
- 多人或 多 Agent 协作

---

## 📁 目录结构

运行 `devloop-setup init` 后，项目中会生成一个 `devloop/` 目录：

```text
your-project/
├── devloop/                        # 所有 DevLoop 产物都在这里
│   ├── config.yaml                 # DevLoop 总控配置
│   ├── .state.yaml                 # 运行状态（已 gitignore）
│   ├── context/                    # 项目上下文和架构地图
│   │   ├── architecture-map.md     # L0 模块索引（快速结构扫描）
│   │   ├── tech-stack.md           # 检测到的语言、框架、命令
│   │   ├── module-index.yaml       # 模块规格追踪（has_spec 标记）
│   │   └── CONTEXT.md              # 领域语言和业务上下文
│   ├── decisions/                  # 架构决策记录（ADR）
│   ├── research/                   # 调研笔记和方案草稿
│   ├── tickets/                    # 任务拆分输出
│   ├── reports/                    # 验证报告和交付总结
│   └── openspec/                   # OpenSpec 原生目录
│       ├── config.yaml             # OpenSpec 工件规则和上下文
│       ├── specs/                  # 已归档的正式行为规格
│       └── changes/                # 进行中的变更
│           └── archive/            # 已完成变更的历史
├── src/                            # 你的源代码（不修改）
├── package.json                    # 你的项目清单（不修改）
└── ...
```

**文件管理规则：**
1. 正式决策不放在临时文件中——必须迁移到 `CONTEXT.md`、ADR 或 OpenSpec。
2. 每条信息只保留一个主文件——其他文件引用，不复制。
3. 文件生命周期明确——临时文件可清理、进行中的变更在 `changes/`、完成后归档。

---

## 🔄 生命周期

DevLoop 遵循 10 阶段状态机。阶段不是锁死的步骤——而是可以重复执行的动作。

```text
intake          接收需求，加载项目上下文
triaging        风险分级（L0-L3）
exploring       读取架构地图，构建缺失的模块规格（L1 按需）
grilling        澄清需求（L2 用 grill-me，L3 用 grill-with-docs）
specifying      创建 OpenSpec 变更（proposal、specs、design、tasks）
reviewing_plan  ⏸️ [确认] 展示需求 + 方案摘要
implementing    ⏸️ [L3 确认] 使用 TDD 执行任务，规格先行
verifying       运行测试、/opsx:verify、代码审查
archiving       ⏸️ [确认] 同步规格、归档变更、更新模块索引
done            输出交付报告 📦
```

**允许回退：**
- `implementing → specifying`：发现规格缺口
- `implementing → exploring`：代码库结构与预期不符
- `verifying → implementing`：验证发现代码问题
- `verifying → specifying`：验证发现规格问题

---

## ⏸️ 确认点

即使在高度自动化模式下，DevLoop 也会在关键节点暂停以防止严重错误。

| 确认点 | 时机 | 展示内容 |
|--------|------|---------|
| **需求理解** | 追问后，创建规格前 | 目标、用户、范围、成功标准、决策、不确定项 |
| **方案和规格** | OpenSpec 工件生成后，编码前 | Proposal、specs、design、tasks、测试策略、风险、回滚 |
| **开始编码**（仅 L3） | 首次修改代码前 | 要修改的文件、新增文件、测试文件 |
| **最终归档** | 验证后，归档前 | 交付总结、测试结果、验证结果、警告 |

---

## 📐 规格先行原则

```text
这个变更是否影响外部行为或架构？
  是 → 先更新 OpenSpec，再修改代码
  否 → 直接改代码，在交付总结中说明
```

这能防止规格漂移——代码和规格随时间逐渐脱节，导致 `/opsx:verify` 失去意义。

---

## 🏗️ Brownfield 策略：结构化索引 + 按需建模

DevLoop 不要求全新项目。对于已有代码库，采用三层策略：

```text
L0 — 架构地图（初始化时）
     快速结构扫描。记录模块、路径、职责、依赖。
     不写行为规格，不做深度分析。几分钟完成。

L1 — 按需模块规格（首次触及时）
     当变更首次触及 has_spec: false 的模块时，
     DevLoop 深入该模块并构建行为规格。
     该规格可被后续所有变更复用。

L2 — 持续维护（归档时）
     每次归档变更时自动将增量规格同步到正式注册表。
     如果结构变化，更新模块索引和架构地图。
```

这意味着你的规格覆盖率随实际开发活动自然增长——不会浪费前期文档。🌱

---

## 💾 状态恢复

DevLoop 维护 `devloop/.state.yaml`（已 gitignore）用于跨会话恢复：

```yaml
current_change: add-dark-mode
stage: implementing
risk_level: L2
confirmed: [requirement_understanding, plan_and_spec]
pending_confirmation: []
tasks:
  total: 12
  completed: 7
  current: "2.3 Update theme toggle component"
consecutive_failures: 0
last_updated: "2026-08-01T10:30:00Z"
```

恢复时，DevLoop 读取状态文件并询问：**继续 / 重新开始 / 放弃？**

---

## 🛡️ 失败恢复

| 错误类型 | 恢复策略 |
|---|---|
| 测试失败 | 进入 `diagnosing-bugs` skill——复现、缩小范围、假设、观测、修复、回归测试 |
| 类型检查/Lint 失败 | 直接修复；连续 3 次失败则暂停 |
| OpenSpec CLI 失败 | 修复配置或工件格式后重试；不绕过 CLI |
| `/opsx:verify` 不一致 | 逐项分类，修代码或修规格，重新验证 |
| 代码库结构不符 | 回退到 `exploring`，重新调查 |
| **连续 3 次失败** | **🚨 强制暂停**——输出诊断报告，请求用户介入 |
| 用户中断 | 保存状态，输出 handoff 摘要 |
| 依赖缺失 | 暂停并报告，不尝试替代方案 |

---

## ✅ 三层完成标准

一个变更只有在三层全部通过时才算完成：

```text
代码层     🧪 测试 + 类型检查 + Lint + 构建
规格层     📋 /opsx:verify: 0 个严重问题
交付层     📦 任务 + 代码审查 + 交付总结
```

### 测试标准

| 等级 | 覆盖要求 |
|------|---------|
| **L1** | 至少 1 个回归测试覆盖 Bug 场景 |
| **L2** | 所有 OpenSpec specs scenario 有对应测试覆盖 |
| **L3** | 所有 scenario + 关键路径集成测试 |

**关键领域**（安全、数据、金钱、隐私）无论风险等级如何，都必须有专用测试覆盖。

**核心规则：**
- 测试遵循项目惯例：使用现有测试目录和 `*.test.ts` 命名。
- 测试只测外部可观察行为，不测内部实现细节。
- 不允许修改测试断言来通过而不验证根本原因。
- 测试与实现代码在同一 OpenSpec change 中提交。
- 测试失败是硬性门槛——不可跳过或绕过，否则不能归档。

---

## 🖥️ 宿主兼容性

DevLoop 设计为跨 AI 编码 Agent 工作，不依赖特定宿主功能。

| 宿主 | 状态 | 备注 |
|------|------|------|
| Claude Code | ✅ 完整支持 | 子 Agent 并行代码审查 |
| Codex | ✅ 完整支持 | 串行代码审查 |
| OpenCode | ✅ 完整支持 | 串行代码审查 |
| TRAE | ✅ 完整支持 | 串行代码审查 |
| Cursor | ✅ 完整支持 | 串行代码审查 |

**降级规则：**
- 不支持子 Agent → `code-review` 串行执行
- 不支持自动触发 Skill → 用户手动调用 `/develop`
- 无 OpenSpec CLI → 仅支持 L0/L1；L2/L3 暂停并提示安装

---

## 📂 Skill 文件结构

```text
devloop-setup/
├── SKILL.md                              # 主指令
├── references/
│   ├── setup-checklist.md                # 文件级初始化/检查/修复清单
│   ├── project-detection.md              # 技术栈和模块检测规则
│   ├── routing-policy.md                 # 风险分级规则（L0-L3）
│   └── repair-rules.md                   # 修复模式能做什么、不能做什么
└── templates/
    ├── devloop-config.yaml               # DevLoop 总控配置模板
    ├── openspec-config.yaml              # OpenSpec 工件规则模板
    ├── architecture-map.md               # L0 架构地图模板
    ├── tech-stack.md                     # 技术栈记录模板
    ├── module-index.yaml                 # 模块索引（含 has_spec 追踪）
    ├── CONTEXT.md                        # 领域语言模板
    └── adr-template.md                   # 架构决策记录模板

devloop/
├── SKILL.md                              # 主指令（10 阶段状态机）
├── references/
│   ├── lifecycle.md                      # 完整状态机（含回退路径）
│   ├── routing.md                        # 风险分类与工作流选择
│   ├── confirmation-points.md            # 每个确认点展示什么
│   ├── recovery.md                       # 失败、中断、中途变更处理
│   └── completion-checklist.md           # 三层验证标准
└── templates/
    ├── requirement-summary.md            # 确认点 1 模板
    ├── plan-summary.md                   # 确认点 2 模板
    └── delivery-summary.md               # 确认点 4 模板
```

---

## 📦 依赖

| 依赖 | 用于 | 安装 |
|------|------|------|
| [OpenSpec CLI](https://github.com/Fission-AI/OpenSpec) | L2/L3 完整工作流 | `npm install -g @fission-ai/openspec@latest` |
| [mattpocock/skills](https://github.com/mattpocock/skills) | 追问、TDD、调试、代码审查 | `npx skills@latest add mattpocock/skills` |

DevLoop 在无依赖时可处理 L0/L1 任务。L2/L3 会暂停并报告缺失依赖。

---

## 💡 核心原则

1. **单入口**——用户只提供需求，不需要手动编排。
2. **先理解再执行**——写任何代码前先追问澄清。
3. **OpenSpec 是事实源**——当前变更的 proposal、specs、design 和 tasks 是唯一执行依据。
4. **Skills 是能力层**——追问、TDD、调试和审查是可组合工具，不是刚性框架。
5. **风险决定重量**——小修复保持轻量；大变更完整闭环。
6. **实现允许回退**——新信息先更新规格再改代码。
7. **完成必须可验证**——三层验证：代码、规格、交付。

---

## 🏛️ 权威层级

```text
CONTEXT.md / ADR              → 长期项目事实和领域语言
devloop/openspec/changes/     → 当前变更的唯一执行事实
devloop/openspec/specs/       → 已归档的正式行为规格
Issue Tracker                 → 团队协作、负责人、排期
Code & Tests                  → 实际实现
```

文档冲突时：**当前 OpenSpec 变更优先**，但冲突必须在交付总结中报告。

---

## 🤝 贡献

欢迎贡献。对于重大变更，请遵循 DevLoop 工作流本身：

1. 在你的 fork 中运行 `/devloop-setup init`
2. `/develop <你的改进>`
3. 确保三层验证通过
4. 提交 PR 并附上交付总结

---

## 📄 许可证

[MIT](LICENSE)
