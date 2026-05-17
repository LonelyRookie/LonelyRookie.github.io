---
title: mattpocock-skills入门指南：让AI编码助手真正靠谱的技能集
tags:
  - AI工具
  - Claude Code
  - Spec-Driven Development
categories:
  - AI工具
  - Spec-Driven Development
abbrlink: 1627271544
date: 2026-05-01 14:08:48
---

## 简介

[mattpocock/skills](https://github.com/mattpocock/skills) 是由 TypeScript 领域知名专家 Matt Pocock 创建的一套 AI 编码助手技能集合，旨在解决 AI 代理在日常开发中的常见失败模式。

> **一句话总结：这套技能让你的 AI 编码助手从"能跑就行"变成"真正靠谱"。**

**项目数据**：⭐ 50K+ GitHub Stars | MIT 开源 | Shell 脚本

### 为什么需要这套技能？

AI 编码助手（Claude Code、Codex、Cursor 等）很强大，但经常会遇到这些问题：

| 问题 | 表现 |
|------|------|
| 代理没按预期执行 | 你说东，它往西，最后发现是需求理解偏差 |
| 代理过于冗长 | 简单的事说半天，token 烧得心疼 |
| 代码无法工作 | 写完一跑就挂，缺乏测试反馈 |
| 代码变成泥球 | 功能有了，但架构一团糟，难以维护 |

这套技能正是为解决这些问题而生。

### 与其他方法的区别

| 方法 | 特点 | 限制 |
|------|------|------|
| GSD / BMAD / Spec-Kit | 拥有整个流程 | 剥夺控制权，流程 bug 难解决 |
| **Skills For Real Engineers** | 小型、易适应、可组合 | ✅ 保留控制权，适用任何模型 |

---

## 快速入门 (30秒)

### 安装步骤

```bash
# 步骤 1: 运行安装程序
npx skills@latest add mattpocock/skills

# 步骤 2: 选择你想要的技能和编码代理
# 重要：必须选择 /setup-matt-pocock-skills

# 步骤 3: 在你的 agent 中运行设置命令
/setup-matt-pocock-skills
```

### 设置配置项

运行 `/setup-matt-pocock-skills` 后会询问：

1. **问题跟踪器** — GitHub / Linear / 本地文件
2. **分类标签** — `/triage` 命令使用的标签词汇表
3. **文档位置** — 保存创建文档的路径

---

## 四大核心问题与解决方案

### 问题 #1：代理没按预期执行

**根本原因**：软件开发中最常见的失败模式是**对齐问题**——你以对方理解了需求，但实际构建结果完全不同。

> "No-one knows exactly what they want" — *The Pragmatic Programmer*

**解决方案**：**Grilling Session（深度询问会话）**

让代理在开始编码前，详细询问你要构建的内容。

| 技能 | 用途 | 场景 |
|------|------|------|
| `/grill-me` | 深度访谈 | 非代码场景，对计划或设计进行无情询问 |
| `/grill-with-docs` | 深度访谈+文档 | 代码工程场景，构建共享语言 |

**使用建议**：每次要做出改变时都应使用这些技能。

### 问题 #2：代理过于冗长

**根本原因**：项目开始时，开发者和领域专家使用不同的术语体系。代理需要自己理解术语，导致用 20 个词表达本可以用 1 个词说清楚的内容。

> "With a ubiquitous language, conversations among developers and expressions of the code are all derived from the same domain model." — Eric Evans, *Domain-Driven Design*

**解决方案**：**共享语言文档（CONTEXT.md）**

**示例对比**：

| 场景 | BEFORE | AFTER |
|------|--------|-------|
| 描述问题 | "There's a problem when a lesson inside a section of a course is made 'real' (i.e. given a spot in the file system)" | "There's a problem with the materialization cascade" |

**使用 `/grill-with-docs` 的优势**：

- 变量、函数和文件命名一致
- 代码库更易导航
- 代理消耗更少的 token（语言更简洁）

#### CONTEXT.md 文件结构

`/grill-with-docs` 会自动创建和更新领域语言文档：

**单上下文项目**：

```text
/
├── CONTEXT.md           ← 领域语言词汇表
├── docs/
│   └── adr/             ← 架构决策记录
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

**多上下文项目**（存在 CONTEXT-MAP.md）：

```text
/
├── CONTEXT-MAP.md       ← 指向各上下文位置
├── docs/
│   └── adr/             ← 系统级决策
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md   ← 订购上下文
│   │   └── docs/adr/    ← 上下文特定决策
│   └── billing/
│       ├── CONTEXT.md   ← 账单上下文
│       └── docs/adr/
```

#### ADR（架构决策记录）创建条件

仅在以下三个条件**同时满足**时创建 ADR：

| 条件 | 说明 |
|------|------|
| **Hard to reverse** | 后期更改成本显著 |
| **Surprising without context** | 未来读者会困惑"为什么这样做" |
| **Result of a real trade-off** | 存在真实权衡，有替代方案 |

如果任一条件不满足，跳过 ADR。

### 问题 #3：代码无法工作

**根本原因**：即使对齐了目标，代码仍可能质量低下。关键在于**反馈循环**——没有代码运行反馈，代理会盲目飞行。

> "Always take small, deliberate steps. The rate of feedback is your speed limit." — *The Pragmatic Programmer*

**解决方案**：建立完整的反馈循环

| 反馈类型 | 工具/方法 |
|----------|----------|
| 静态类型 | TypeScript 等 |
| 浏览器访问 | 实时预览 |
| 自动化测试 | Red-Green-Refactor 循环 |

**相关技能**：

| 技能 | 用途 |
|------|------|
| `/tdd` | 测试驱动开发，Red-Green-Refactor 循环 |
| `/diagnose` | 调试诊断循环：复现 → 最小化 → 假设 → 仪器化 → 修复 → 回归测试 |

#### `/diagnose` 六阶段诊断流程详解

这是解决棘手 bug 的核心技能，遵循严格的六阶段循环：

```
Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6
构建循环    复现      假设      仪器化    修复+测试   清理+复盘
```

**Phase 1：构建反馈循环（最关键）**

> "This is the skill. Everything else is mechanical."

构建一个快速、确定性、可自动运行的 pass/fail 信号。尝试顺序：

| 方法 | 适用场景 |
|------|----------|
| Failing test | 单元/集成/E2E 测试 |
| Curl / HTTP script | 针对 dev server |
| CLI invocation + snapshot diff | 命令行工具 |
| Headless browser script | UI 驱动测试 |
| Replay captured trace | 网络/事件回放 |
| Throwaway harness | 最小子系统测试 |
| Property / fuzz loop | 随机输入测试 |
| Bisection harness | `git bisect run` |
| Differential loop | 新旧版本对比 |

**关键指标**：2 秒确定性循环 > 30 秒不稳定循环

**Phase 2：复现**

确认三点：
- [ ] 循环产生用户描述的失败模式
- [ ] 失败可重复（或非确定性 bug 有足够高的复现率）
- [ ] 捕获确切症状以便后续验证

**Phase 3：假设**

生成 **3-5 个可证伪假设**，按优先级排序：

> 格式："如果 <X> 是原因，那么 <改变 Y> 会让 bug 消失 / <改变 Z> 会让它恶化"

**Phase 4：仪器化**

每个探测对应 Phase 3 的具体预测。**一次只改变一个变量**：

- 优先使用 debugger / REPL
- 使用边界处的 targeted logs
- 永远不要"log everything and grep"
- 所有调试日志使用唯一前缀 `[DEBUG-a4f2]`，便于清理

**Phase 5：修复 + 回归测试**

在修复前写回归测试（仅当存在正确的测试 seam）：

1. 将最小化复现转为失败测试
2. 观察失败
3. 应用修复
4. 观察通过
5. 用 Phase 1 循环验证原始场景

**Phase 6：清理 + 复盘**

必须完成：
- [ ] 原始复现不再复现
- [ ] 回归测试通过
- [ ] 所有 `[DEBUG-...]` 仪器移除
- [ ] 临时原型删除
- [ ] 正确假设写入 commit message

**事后反思**：什么能预防此 bug？如果涉及架构问题，移交 `/improve-codebase-architecture`。

### 问题 #4：构建了泥球（Ball of Mud）

**根本原因**：代理加速编码的同时也加速了软件熵增，代码库以前所未有的速度变得复杂。

> "Invest in the design of the system _every day_." — Kent Beck, *Extreme Programming Explained*

**解决方案**：持续关注代码设计

| 技能 | 用途 | 使用频率 |
|------|------|----------|
| `/to-prd` | 创建 PRD 前询问涉及的模块 | 按需 |
| `/zoom-out` | 让代理在系统整体上下文中解释代码 | 按需 |
| `/improve-codebase-architecture` | 拯救变成一团糟的代码库 | **每几天一次** |

#### `/improve-codebase-architecture` 核心术语表

该技能使用特定术语描述架构改进：

| 术语 | 含义 |
|------|------|
| **Module** | 任何有接口和实现的东西（函数、类、包、切片） |
| **Interface** | 调用者必须知道的一切：类型、不变量、错误模式、顺序、配置 |
| **Implementation** | 内部代码 |
| **Depth** | 接口的杠杆：小接口背后的大量行为。深 = 高杠杆 |
| **Seam** | 接口所在位置，可在此改变行为而不需原地编辑 |
| **Adapter** | 满足接口的具体实现 |

**关键原则**：

- **删除测试**：想象删除模块。如果复杂度消失，它是透传层；如果复杂度在 N 个调用者处重现，它有价值。
- **接口即测试面**。
- **一个 Adapter = 假设 Seam。两个 Adapter = 真实 Seam。**

---

## 技能清单详解

### 🔧 Engineering Skills（工程技能）

| 技能 | 功能 | 命令 |
|------|------|------|
| **diagnose** | 针对困难 bug 的纪律化诊断循环：reproduce → minimise → hypothesise → instrument → fix → regression-test | `/diagnose` |
| **grill-with-docs** | 深度询问会话，挑战你的计划，更新 CONTEXT.md 和 ADR | `/grill-with-docs` |
| **triage** | 通过状态机对问题进行分类 | `/triage` |
| **improve-codebase-architecture** | 在代码库中寻找深化机会 | `/improve-codebase-architecture` |
| **setup-matt-pocock-skills** | 初始化仓库配置（问题跟踪器、标签、文档布局） | `/setup-matt-pocock-skills` |
| **tdd** | 测试驱动开发，Red-Green-Refactor 循环 | `/tdd` |
| **to-issues** | 将计划/规格/PRD 拆分为独立 GitHub issues | `/to-issues` |
| **to-prd** | 将对话上下文转换为 PRD 并提交为 GitHub issue | `/to-prd` |
| **zoom-out** | 让代理提供更宏观的代码视角 | `/zoom-out` |

#### `/tdd` 反模式警告：避免水平切片

**错误做法（水平切片）**：

```text
RED:   test1, test2, test3, test4, test5  ← 一次性写完所有测试
GREEN: impl1, impl2, impl3, impl4, impl5   ← 一次性写完所有实现
```

这会产生**劣质测试**：
- 批量测试验证的是**想象的行为**，而非**实际行为**
- 测试的是"形状"（数据结构、函数签名）而非用户行为
- 重构时测试不敏感——行为破坏时通过，行为正常时失败

**正确做法（垂直切片 / Tracer Bullets）**：

```text
RED→GREEN: test1 → impl1    ← 一个测试对应一个实现
RED→GREEN: test2 → impl2
RED→GREEN: test3 → impl3
...
```

每个测试响应上一轮学到的东西。因为刚写完代码，你确切知道什么行为重要。

**核心原则**：测试验证**行为**而非**实现**。好测试读起来像规格说明——"用户可以用有效购物车结账"。

### 🚀 Productivity Skills（生产力技能）

| 技能 | 功能 | 命令 |
|------|------|------|
| **caveman** | 超压缩通信模式，减少约 **75% token 使用** | `/caveman` |
| **grill-me** | 对计划或设计进行无情访谈 | `/grill-me` |
| **write-a-skill** | 创建新技能 | `/write-a-skill` |

#### `/caveman` 模式示例

启用后，代理会使用极简语言，保留所有技术内容，只删除填充词：

| 普通模式 | Caveman 模式 |
|---------|-------------|
| "Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by..." | "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix:" |
| "The inline object prop creates a new reference on each render, which causes the component to re-render unnecessarily. You should use `useMemo` to stabilize it." | "Inline obj prop -> new ref -> re-render. `useMemo`." |

**规则**：
- 删除：冠词（a/an/the）、填充词（just/really/basically）、客套话（sure/certainly）
- 保留：技术术语精确、代码块不变、错误信息完整
- 格式：`[事物] [动作] [原因]. [下一步].`

**例外**：安全警告、不可逆操作确认、多步骤序列时暂时恢复正常模式。

### 📦 Misc Skills（杂项技能）

| 技能 | 功能 | 命令 |
|------|------|------|
| **git-guardrails-claude-code** | 阻止危险的 git 命令（push、reset --hard、clean 等） | `/git-guardrails-claude-code` |
| **migrate-to-shoehorn** | 迁移测试文件到 @total-typescript/shoehorn | `/migrate-to-shoehorn` |
| **scaffold-exercises** | 创建练习目录结构 | `/scaffold-exercises` |
| **setup-pre-commit** | 设置 Husky pre-commit hooks | `/setup-pre-commit` |

---

## 核心设计理念

### 1. 小型、易适应、可组合

每个技能独立工作，不绑定特定 AI 模型，可以自由组合使用。

### 2. 基于经典软件工程经验

这套技能的灵感来源：

| 书籍 | 作者 | 核心思想 |
|------|------|----------|
| 《The Pragmatic Programmer》 | David Thomas & Andrew Hunt | 小步快跑，快速反馈 |
| 《Domain-Driven Design》 | Eric Evans | 统一语言，共享模型 |
| 《Extreme Programming Explained》 | Kent Beck | 持续投资设计 |
| 《A Philosophy Of Software Design》 | John Ousterhout | 深模块设计 |

### 3. 保留开发者控制权

与 GSD、BMAD、Spec-Kit 等试图控制整个流程的方法不同，这套技能让开发者保持控制权，只在需要时召唤特定专家。

---

## 实战使用建议

### 推荐工作流

```
每次开始新功能：
1. /grill-with-docs   → 对齐需求，更新共享语言
2. /tdd               → 测试驱动开发
3. 完成后 /improve-codebase-architecture → 持续优化架构（每几天）
```

### 使用频率建议

| 技能 | 频率 |
|------|------|
| `/grill-me` 或 `/grill-with-docs` | 每次开始新功能 |
| `/tdd` | 编写功能代码时 |
| `/improve-codebase-architecture` | 每几天一次 |
| `/diagnose` | 遇到棘手 bug 时 |
| `/zoom-out` | 理解不熟悉的代码时 |

### 实战案例：开发新功能完整流程

假设要为电商系统添加"优惠券"功能：

**Step 1：对齐需求（`/grill-with-docs`）**

```text
用户：我想添加优惠券功能

代理开始访谈：
Q: 优惠券是否可叠加使用？
A: 不可以，每单限用一张

Q: 优惠券是否有使用门槛（如满100减20）？
A: 是的，需要支持满减和折扣两种类型

Q: 过期优惠券如何处理？
A: 自动失效，用户不可见

→ 自动更新 CONTEXT.md，添加术语：
- Coupon: 优惠券，包含满减券和折扣券
- Coupon stacking: 优惠券叠加，本系统不支持
- Coupon threshold: 使用门槛金额
```

**Step 2：测试驱动开发（`/tdd`）**

```text
RED:   写测试 "用户可以在购物车应用有效优惠券"
GREEN: 实现优惠券应用逻辑
RED:   写测试 "过期优惠券不可使用"
GREEN: 实现过期检查
RED:   写测试 "每单只能使用一张优惠券"
GREEN: 实现单券限制
...
```

**Step 3：架构优化（每几天运行 `/improve-codebase-architecture`）**

```text
代理发现：
- CouponService 和 OrderService 耦合过紧
- 建议将优惠券验证逻辑抽取为独立的 CouponValidator
- 创建 Seam，支持未来扩展多种优惠券类型
```

---

## 常见问题

### Q: 必须使用 `/setup-matt-pocock-skills` 吗？

**是的**。这个技能会初始化问题跟踪器、分类标签和文档布局，其他工程技能依赖这些配置。

### Q: 支持哪些 AI 编码助手？

支持所有主流编码助手：Claude Code、Codex、Cursor、Windsurf 等。

### Q: 与 Spec-Kit 有什么区别？

Spec-Kit 拥有整个流程控制权，而这套技能提供小型、可组合的工具，让开发者保持控制权。

### Q: `/grill-me` 和 `/grill-with-docs` 有什么区别？

- `/grill-me` — 纯访谈，不更新文档
- `/grill-with-docs` — 访谈 + 自动更新 CONTEXT.md 和 ADR

---

## 总结

mattpocock/skills 是一套将经典软件工程最佳实践浓缩为可重复操作的 AI 编码技能集。核心理念：

| 理念 | 实践 |
|------|------|
| 对齐优先 | `/grill-me`、`/grill-with-docs` |
| 共享语言 | CONTEXT.md、ADR |
| 快速反馈 | `/tdd`、`/diagnose` |
| 持续设计 | `/improve-codebase-architecture`、`/zoom-out` |

> **核心价值**：不剥夺控制权，让开发者在 AI 辅助下保持工程纪律，真正实现"vibe coding"的对立面——靠谱的工程实践。

---

## 相关链接

- [GitHub 仓库](https://github.com/mattpocock/skills) - 50K+ Stars
- [Newsletter](https://www.aihero.dev/s/skills-newsletter) - 约 60,000 开发者订阅
- [示例 CONTEXT.md](https://github.com/mattpocock/course-video-manager/blob/076a5a7a182db0fe1e62971dd7a68bcadf010f1c/CONTEXT.md)