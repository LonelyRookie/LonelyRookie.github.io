---
title: superpowers入门指南
tags:
  - AI工具
  - Claude Code
  - Spec-Driven Development
categories:
  - AI工具
  - Spec-Driven Development
abbrlink: 1947712232
date: 2026-04-26 22:34:20
---

## 简介

[Superpowers](https://github.com/obra/superpowers) 是一个完整的软件开发方法论，专为编码代理（coding agents）设计。它建立在可组合技能（skills）和初始指令之上，确保你的 AI 编码助手能够更智能、更系统地完成开发任务。

### 为什么需要 Superpowers？

当你启动编码代理时，它通常不会停下来思考你真正想要做什么，而是直接跳入编写代码。Superpowers 改变了这一点：

1. **先思考后编码** - 代理会先理解你的需求，提炼出规格说明
2. **分块展示设计** - 将设计分成易于消化的小块供你审阅
3. **制定清晰计划** - 为"热情但缺乏判断力的初级工程师"级别的执行者提供可遵循的实施计划
4. **自主执行** - 代理可以自主工作数小时而不偏离计划

### 核心理念

| 原则 | 说明 |
|------|------|
| **测试驱动开发 (TDD)** | 始终先写测试，永远不跳过 |
| **系统化优于临时方案** | 流程优于猜测，过程优于假设 |
| **降低复杂度** | 简洁是首要目标，小文件优于大文件 |
| **证据优于声明** | 验证后再宣布成功，看到测试失败再写代码 |

### 技能优先级

Superpowers 技能覆盖默认系统行为，但**用户指令始终优先**：

1. **用户的显式指令** (CLAUDE.md, GEMINI.md, AGENTS.md, 直接请求) — 最高优先级
2. **Superpowers 技能** — 覆盖默认系统行为
3. **默认系统提示** — 最低优先级

## 快速入门 (5分钟)

### 前置要求

- 已安装 Claude Code、Cursor、Copilot CLI 或其他支持的编码代理
- 基本的 Git 知识

### 安装

#### Claude Code 官方市场

```bash
/plugin install superpowers@claude-plugins-official
```

#### Claude Code (Superpowers 市场)

```bash
# 先注册市场
/plugin marketplace add obra/superpowers-marketplace

# 然后安装插件
/plugin install superpowers@superpowers-marketplace
```

#### Cursor

```text
/add-plugin superpowers
```

或在插件市场中搜索 "superpowers"

#### GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

#### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

#### OpenCode

```text
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

## 详细教程：核心工作流程

Superpowers 的工作流程是一个自动触发的技能链：

```
1. brainstorming (头脑风暴)
   ↓
2. using-git-worktrees (Git 工作树)
   ↓
3. writing-plans (编写计划)
   ↓
4. subagent-driven-development (子代理驱动开发)
   ↓
5. test-driven-development (测试驱动开发)
   ↓
6. requesting-code-review (代码审查)
   ↓
7. finishing-a-development-branch (完成开发分支)
```

---

## 核心技能详解

### 1. Brainstorming（头脑风暴）

**触发时机**：在任何创造性工作之前 — 创建功能、构建组件、添加功能或修改行为。

**核心原则**：在展示设计并获得用户批准之前，**绝不**调用任何实现技能、编写任何代码、搭建任何项目或采取任何实现行动。

#### 工作流程

```dot
探索项目上下文 → 提供视觉伴侣（如需要）→ 提出澄清问题 → 提出2-3种方案 → 展示设计 → 用户批准 → 编写设计文档 → 规格自审 → 用户审核 → 调用 writing-plans 技能
```

#### 关键检查清单

1. **探索项目上下文** — 检查文件、文档、最近提交
2. **提供视觉伴侣**（如果涉及视觉问题）— 独立消息，不与其他内容混合
3. **提出澄清问题** — 一次一个问题，理解目的/约束/成功标准
4. **提出2-3种方案** — 包含权衡和你推荐的理由
5. **展示设计** — 按复杂度分段，每段后获取用户批准
6. **编写设计文档** — 保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` 并提交
7. **规格自审** — 快速检查占位符、矛盾、歧义、范围
8. **用户审核书面规格** — 请用户在继续之前审核规格文件
9. **过渡到实现** — 调用 writing-plans 技能创建实现计划

#### 反模式警告

> **"这太简单了，不需要设计"**
> 
> 每个项目都要经过这个过程。待办列表、单函数工具、配置更改 — 全部都是。"简单"项目正是未检查的假设导致最多浪费工作的地方。设计可以很短（对于真正简单的项目只需几句话），但你必须展示并获得批准。

#### 关键原则

- **一次一个问题** — 不要用多个问题淹没用户
- **优先多选题** — 比开放式问题更容易回答
- **无情应用 YAGNI** — 从所有设计中删除不必要的功能
- **探索替代方案** — 在确定之前总是提出2-3种方法
- **增量验证** — 展示设计，获得批准后再继续

---

### 2. Using Git Worktrees（Git 工作树）

**触发时机**：设计批准后。

**功能**：
- 在新分支上创建隔离工作空间
- 运行项目设置
- 验证干净的测试基线

**优势**：允许并行开发，不干扰主分支工作。

---

### 3. Writing Plans（编写计划）

**触发时机**：有规格或需求的多步骤任务，在接触代码之前。

**核心原则**：假设工程师对我们的代码库零上下文，品味存疑。记录他们需要知道的一切。

#### 计划文档头部

每个计划**必须**以此头部开始：

```markdown
# [功能名称] 实现计划

> **对于代理工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框 (`- [ ]`) 语法跟踪。

**目标：** [一句话描述这构建什么]

**架构：** [2-3句关于方法]

**技术栈：** [关键技术/库]

---
```

#### 任务粒度

**每个步骤是一个动作（2-5分钟）：**

- "编写失败的测试" - 步骤
- "运行它确保失败" - 步骤
- "实现最小代码使测试通过" - 步骤
- "运行测试确保通过" - 步骤
- "提交" - 步骤

#### 任务结构示例

```markdown
### 任务 N: [组件名称]

**文件：**
- 创建: `exact/path/to/file.py`
- 修改: `exact/path/to/existing.py:123-145`
- 测试: `tests/exact/path/to/test.py`

- [ ] **步骤 1: 编写失败的测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **步骤 2: 运行测试验证失败**

运行: `pytest tests/path/test.py::test_name -v`
预期: FAIL with "function not defined"

- [ ] **步骤 3: 编写最小实现**

```python
def function(input):
    return expected
```

- [ ] **步骤 4: 运行测试验证通过**

运行: `pytest tests/path/test.py::test_name -v`
预期: PASS

- [ ] **步骤 5: 提交**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

#### 禁止的占位符

每个步骤必须包含工程师需要的实际内容。这些是**计划失败** — 永远不要写它们：

- "TBD"、"TODO"、"稍后实现"、"填写详情"
- "添加适当的错误处理" / "添加验证" / "处理边缘情况"
- "为上述编写测试"（没有实际测试代码）
- "类似于任务 N"（重复代码 — 工程师可能乱序阅读任务）
- 描述做什么而不展示如何的步骤（代码步骤需要代码块）
- 引用任何任务中未定义的类型、函数或方法

---

### 4. Test-Driven Development（测试驱动开发）

**触发时机**：实现任何功能或修复任何 bug，在编写实现代码之前。

#### 铁律

```
没有失败测试在前，不写生产代码
```

先写代码再写测试？删除它。重新开始。

**无例外：**
- 不要保留它作为"参考"
- 不要在写测试时"改编"它
- 不要看它
- 删除意味着删除

从测试开始全新实现。句号。

#### RED-GREEN-REFACTOR 循环

```dot
RED [编写失败测试] → 验证失败正确 → GREEN [最小代码] → 验证通过 → REFACTOR [清理] → 下一个
```

##### RED - 编写失败测试

编写一个最小测试展示应该发生什么。

**好的测试：**
```typescript
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };

  const result = await retryOperation(operation);

  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```
✅ 清晰名称，测试真实行为，一件事

**坏的测试：**
```typescript
test('retry works', async () => {
  const mock = jest.fn()
    .mockRejectedValueOnce(new Error())
    .mockRejectedValueOnce(new Error())
    .mockResolvedValueOnce('success');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(3);
});
```
❌ 模糊名称，测试模拟而非代码

##### 验证 RED - 观察失败

**强制执行。永不跳过。**

```bash
npm test path/to/test.test.ts
```

确认：
- 测试失败（不是错误）
- 失败消息是预期的
- 失败是因为功能缺失（不是拼写错误）

**测试通过？** 你在测试现有行为。修复测试。

**测试错误？** 修复错误，重新运行直到正确失败。

##### GREEN - 最小代码

编写最简单的代码通过测试。

**好的实现：**
```typescript
async function retryOperation<T>(fn: () => Promise<T>): Promise<T> {
  for (let i = 0; i < 3; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === 2) throw e;
    }
  }
  throw new Error('unreachable');
}
```
✅ 刚好足够通过

**坏的实现：**
```typescript
async function retryOperation<T>(
  fn: () => Promise<T>,
  options?: {
    maxRetries?: number;
    backoff?: 'linear' | 'exponential';
    onRetry?: (attempt: number) => void;
  }
): Promise<T> {
  // YAGNI - 你不需要它
}
```
❌ 过度工程

#### 常见合理化陷阱

| 借口 | 现实 |
|------|------|
| "太简单不需要测试" | 简单代码也会出错。测试只需30秒。 |
| "我会在之后测试" | 测试立即通过证明不了什么。 |
| "之后测试达到相同目标" | 之后测试 = "这做什么？" 先测试 = "这应该做什么？" |
| "已经手动测试过" | 临时的 ≠ 系统的。没有记录，无法重跑。 |
| "删除 X 小时是浪费" | 沉没成本谬误。保留未验证的代码是技术债务。 |
| "保留作为参考，先写测试" | 你会改编它。那是之后测试。删除意味着删除。 |
| "需要先探索" | 可以。扔掉探索，从 TDD 开始。 |
| "测试困难 = 设计不清晰" | 倾听测试。难测试 = 难使用。 |
| "TDD 会让我变慢" | TDD 比调试更快。务实 = 先测试。 |

#### 红旗警告 - 停止并重新开始

- 测试前有代码
- 实现后测试
- 测试立即通过
- 无法解释为什么测试失败
- "稍后"添加测试
- 合理化"就这一次"
- "我已经手动测试过了"
- "之后测试达到相同目的"
- "这是精神而非仪式"
- "保留作为参考"或"改编现有代码"
- "已经花了 X 小时，删除是浪费"
- "TDD 是教条的，我是务实的"
- "这是不同的因为..."

**所有这些都意味着：删除代码。用 TDD 重新开始。**

---

### 5. Systematic Debugging（系统化调试）

**触发时机**：遇到任何 bug、测试失败或意外行为，在提出修复之前。

#### 铁律

```
没有先调查根本原因，不修复
```

如果你还没完成阶段 1，你就不能提出修复。

#### 四阶段流程

##### 阶段 1: 根本原因调查

**在尝试任何修复之前：**

1. **仔细阅读错误消息**
   - 不要跳过错误或警告
   - 它们通常包含确切的解决方案
   - 完整阅读堆栈跟踪
   - 记录行号、文件路径、错误代码

2. **一致地重现**
   - 你能可靠地触发它吗？
   - 确切步骤是什么？
   - 每次都发生吗？
   - 如果不可重现 → 收集更多数据，不要猜测

3. **检查最近更改**
   - 什么变化可能导致这个？
   - Git diff，最近提交
   - 新依赖，配置更改
   - 环境差异

4. **在多组件系统中收集证据**

   **当系统有多个组件时（CI → 构建 → 签名，API → 服务 → 数据库）：**

   **在提出修复之前，添加诊断工具：**
   ```bash
   对于每个组件边界：
     - 记录什么数据进入组件
     - 记录什么数据退出组件
     - 验证环境/配置传播
     - 检查每层状态

   运行一次收集证据显示哪里中断
   然后分析证据识别失败组件
   然后调查那个特定组件
   ```

5. **追踪数据流**
   - 错误值从哪里起源？
   - 什么用错误值调用了这个？
   - 继续向上追踪直到找到源头
   - 在源头修复，而不是症状

##### 阶段 2: 模式分析

**在修复之前找到模式：**

1. **找到工作示例**
   - 在同一代码库中定位类似的正常工作代码
   - 什么工作的与损坏的相似？

2. **与参考比较**
   - 如果实现模式，完整阅读参考实现
   - 不要浏览 - 阅读每一行
   - 在应用之前完全理解模式

3. **识别差异**
   - 工作的和损坏的之间有什么不同？
   - 列出每个差异，无论多小
   - 不要假设"那不可能重要"

4. **理解依赖**
   - 这需要什么其他组件？
   - 什么设置、配置、环境？
   - 它做什么假设？

##### 阶段 3: 假设和测试

**科学方法：**

1. **形成单一假设**
   - 清楚陈述："我认为 X 是根本原因因为 Y"
   - 写下来
   - 要具体，不要模糊

2. **最小测试**
   - 做最小可能的更改来测试假设
   - 一次一个变量
   - 不要一次修复多个东西

3. **验证后再继续**
   - 有效？是 → 阶段 4
   - 无效？形成新假设
   - 不要在上面添加更多修复

4. **当你不知道时**
   - 说"我不理解 X"
   - 不要假装知道
   - 寻求帮助
   - 更多研究

##### 阶段 4: 实现

**修复根本原因，不是症状：**

1. **创建失败测试用例**
   - 最简单的可能重现
   - 如果可能自动化测试
   - 修复前必须有

2. **实现单一修复**
   - 解决识别的根本原因
   - 一次一个更改
   - 没有"既然我在这里"的改进
   - 没有捆绑重构

3. **验证修复**
   - 测试现在通过？
   - 没有其他测试损坏？
   - 问题实际解决？

4. **如果修复无效**
   - 停止
   - 计数：你尝试了多少次修复？
   - 如果 < 3：返回阶段 1，用新信息重新分析
   - **如果 ≥ 3：停止并质疑架构（步骤 5）**
   - 不要在没有架构讨论的情况下尝试修复 #4

5. **如果 3+ 次修复失败：质疑架构**

   **指示架构问题的模式：**
   - 每次修复在不同地方揭示新的共享状态/耦合/问题
   - 修复需要"大规模重构"来实现
   - 每次修复在其他地方创建新症状

   **停止并质疑基础：**
   - 这个模式根本上是健全的吗？
   - 我们是否"纯粹出于惯性坚持它"？
   - 我们应该重构架构还是继续修复症状？

#### 红旗警告 - 停止并遵循流程

如果你发现自己在想：

- "暂时快速修复，稍后调查"
- "只是尝试更改 X 看看是否有效"
- "添加多个更改，运行测试"
- "跳过测试，我会手动验证"
- "可能是 X，让我修复那个"
- "我不完全理解但这可能有效"
- "模式说 X 但我会不同地适应它"
- "这是主要问题：[列出没有调查的修复]"
- 在追踪数据流之前提出解决方案
- **"再尝试一次修复"**（当已经尝试 2+ 次）
- **每次修复在不同地方揭示新问题**

**所有这些都意味着：停止。返回阶段 1。**

---

### 6. Requesting Code Review（代码审查请求）

**触发时机**：任务之间。

**功能**：
- 对照计划审查
- 按严重程度报告问题
- 关键问题阻止进度

---

### 7. Finishing a Development Branch（完成开发分支）

**触发时机**：任务完成时。

**功能**：
- 验证测试
- 展示选项（合并/PR/保留/丢弃）
- 清理工作树

---

## 技能库完整列表

| 类别 | 技能 | 说明 |
|------|------|------|
| **测试** | test-driven-development | RED-GREEN-REFACTOR 循环（包含测试反模式参考） |
| **调试** | systematic-debugging | 4 阶段根因分析流程（包含根因追踪、深度防御、基于条件的等待技术） |
| | verification-before-completion | 确保问题真正修复 |
| **协作** | brainstorming | 苏格拉底式设计提炼 |
| | writing-plans | 详细实施计划 |
| | executing-plans | 带检查点的批量执行 |
| | dispatching-parallel-agents | 并发子代理工作流 |
| | requesting-code-review | 审查前检查清单 |
| | receiving-code-review | 响应反馈 |
| | using-git-worktrees | 并行开发分支 |
| | finishing-a-development-branch | 合并/PR 决策流程 |
| | subagent-driven-development | 快速迭代 + 两阶段审查 |
| **元技能** | writing-skills | 按最佳实践创建新技能（包含测试方法论） |
| | using-superpowers | 技能系统介绍 |

---

## 使用技能的规则

### 核心规则

**在任何响应或操作之前调用相关或请求的技能。** 即使只有 1% 的可能性技能可能适用，你也应该调用技能来检查。

### 红旗思维

这些想法意味着停止 — 你在合理化：

| 想法 | 现实 |
|------|------|
| "这只是一个简单的问题" | 问题就是任务。检查技能。 |
| "我需要更多上下文" | 技能检查在澄清问题之前。 |
| "让我先探索代码库" | 技能告诉你如何探索。先检查。 |
| "我可以快速检查 git/文件" | 文件缺少对话上下文。检查技能。 |
| "让我先收集信息" | 技能告诉你如何收集信息。 |
| "这不需要正式技能" | 如果技能存在，使用它。 |
| "我记得这个技能" | 技能会演变。阅读当前版本。 |
| "这不算任务" | 行动 = 任务。检查技能。 |
| "技能是过度杀伤" | 简单的事情变得复杂。使用它。 |
| "我只做这一件事" | 在做任何事之前检查。 |
| "这感觉很有成效" | 不守纪律的行动浪费时间。技能防止这一点。 |
| "我知道那意味着什么" | 知道概念 ≠ 使用技能。调用它。 |

### 技能优先级

当多个技能可能适用时，使用此顺序：

1. **流程技能优先**（brainstorming、debugging）— 这些决定如何处理任务
2. **实现技能其次**（frontend-design、mcp-builder）— 这些指导执行

"让我们构建 X" → 先 brainstorming，然后实现技能。
"修复这个 bug" → 先 debugging，然后领域特定技能。

### 技能类型

**刚性**（TDD、debugging）：精确遵循。不要偏离纪律。

**灵活**（patterns）：根据上下文调整原则。

技能本身会告诉你属于哪种。

---

## 自定义技能

你可以使用 `writing-skills` 技能创建自己的技能：

1. 遵循技能创建最佳实践
2. 包含测试方法论
3. 确保跨编码代理兼容性

---

## 贡献指南

如果你想为 Superpowers 做贡献：

```bash
# 1. Fork 仓库
# 2. 切换到 'dev' 分支
# 3. 创建你的工作分支
# 4. 遵循 writing-skills 技能创建和测试新技能
# 5. 提交 PR，填写 PR 模板
```

> **注意**：通常不接受新技能的贡献，技能更新必须兼容所有支持的编码代理。

---

## 常见问题

### Q: Superpowers 支持哪些编码代理？

支持的主要平台：
- Claude Code
- Cursor
- GitHub Copilot CLI
- OpenAI Codex CLI / App
- Gemini CLI
- OpenCode

### Q: 更新如何进行？

更新通常是自动的，具体取决于你使用的编码代理平台。

### Q: 如何获取帮助？

- **Discord**: [加入社区](https://discord.gg/35wsABTejz)
- **Issues**: https://github.com/obra/superpowers/issues
- **发布公告**: [订阅更新](https://primeradiant.com/superpowers/)

### Q: Superpowers 是免费的吗？

是的，Superpowers 采用 MIT 许可证开源。如果它帮助你赚钱，可以考虑 [赞助作者](https://github.com/sponsors/obra)。

---

## 总结

Superpowers 为你的 AI 编码助手提供了一套完整的软件开发方法论。通过自动触发的技能链，它确保：

1. **需求先行** - 先理解再编码
2. **计划驱动** - 清晰的实施步骤
3. **质量保证** - TDD + 代码审查
4. **自主执行** - 长时间自主工作不偏离

核心理念总结：

| 原则 | 实践 |
|------|------|
| **TDD** | 没有失败测试在前，不写生产代码 |
| **系统化调试** | 没有先调查根本原因，不修复 |
| **头脑风暴** | 没有设计批准，不实现 |
| **编写计划** | 每个步骤都是 2-5 分钟的动作 |

如果你正在使用 Claude Code、Cursor 或其他支持的编码代理，Superpowers 是提升开发效率和代码质量的必备工具。

---

## 相关链接

- [GitHub 仓库](https://github.com/obra/superpowers)
- [原版发布公告](https://blog.fsck.com/2025/10/09/superpowers/)
- [作者博客](https://blog.fsck.com)
- [Prime Radiant](https://primeradiant.com)
- [Discord 社区](https://discord.gg/35wsABTejz)
