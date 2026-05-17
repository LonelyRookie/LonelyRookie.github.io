---
name: hexo-blog-writer
description: Use when writing new blog posts for Hexo. Creates documents with `hexo new`, researches topics using GitHub API and Exa search, and produces detailed tutorials with code examples. Trigger when user says "写博客", "写一篇关于...的文章", "帮我写文档", or requests blog post creation.
version: 1.0.0
---

# Hexo 博客写作技能

## 技能概述

本技能用于为 Hexo 博客撰写高质量技术文章，包含完整的创作流程：创建文档、研究主题、撰写内容、质量验证。

## 工作流程

### 第一步：创建博客文档

使用 `hexo new` 命令创建新文档：

```bash
cd <blog_root> && hexo new "<post-title>"
```

这会在 `source/_posts/YYYY-MM-DD-<post-title>.md` 创建文件。

### 第二步：研究主题

使用多种来源全面研究主题：

**GitHub API（用于开源项目）：**
```python
# 获取 README 和文档
mcp__plugin_ecc_github__get_file_contents(owner, repo, path="README.md")
mcp__plugin_ecc_github__get_file_contents(owner, repo, path="docs/")

# 获取代码结构
mcp__plugin_ecc_github__get_file_contents(owner, repo, path="src/")
```

**Exa 搜索（用于教程和对比）：**
```python
# 搜索教程
mcp__plugin_ecc_exa__web_search_exa(
    query="{topic} tutorial guide getting started",
    numResults=5
)

# 搜索对比分析
mcp__plugin_ecc_exa__web_search_exa(
    query="{topic} vs {alternative} comparison",
    numResults=5
)
```

**Context7（用于官方文档）：**
```python
# 获取库文档
mcp__plugin_ecc_context7__resolve-library-id(
    libraryName="{library}",
    query="usage guide tutorial"
)
mcp__plugin_ecc_context7__query-docs(
    libraryId="{id}",
    query="how to use {feature}"
)
```

### 第三步：撰写博客

#### Front Matter 模板

```yaml
---
title: {博客标题}
date: YYYY-MM-DD HH:MM:SS
tags:
  - tag1
  - tag2
categories:
  - Category1
keywords:
  - keyword1
  - keyword2
description: {SEO友好的描述，1-2句话}
---
```

#### 内容结构模板

**教程型文章：**

```markdown
## 一句话理解
{这是什么？为什么重要？}

## 核心价值：为什么需要它？
- 解决什么问题
- 适合谁使用
- 对比表格（适合/不适合）

## 快速开始（5 分钟）
### 安装
### 基本使用

## 详细内容
### 核心概念
### 架构设计
### API 详解

## 实战示例
{完整可运行的代码}

## 对比分析
| 特性 | 本方案 | 替代方案 |
|------|--------|----------|

## 总结
- 关键要点
- 下一步建议
```

**框架评测型文章：**

```markdown
## 前言：为什么选择 {Framework}？
- 解决什么问题
- 市场定位

## 一、架构设计理念
### 整体架构
### 核心模块

## 二、{模块 1}
### 概念
### 代码示例

## 三、{模块 2}
...

## 对比分析
### vs 替代方案 1
### vs 替代方案 2

## 总结
### 适用场景
### 下一步建议
```

### 第四步：质量检查

完成前检查以下项目：

- [ ] Front matter 完整（title, date, tags, categories, description）
- [ ] 代码示例完整可运行
- [ ] 标题层级合理（H1 → H2 → H3）
- [ ] 代码块指定语言（```python, ```bash）
- [ ] 使用表格进行对比
- [ ] 包含官方文档/仓库链接
- [ ] 内容易于扫描（列表、提示框、代码块）
- [ ] 技术术语已解释

## 代码示例规范

**好的示例：**
```python
# 完整、有上下文和注释
from library import Feature

# 初始化配置
feature = Feature(
    param1="value1",  # 参数说明
    param2=100,       # 参数说明
)

# 运行并展示预期输出
result = feature.run()
print(result)  # 输出: {...}
```

**不好的示例：**
```python
# 不完整、无上下文
f = Feature()
f.run()
```

## Markdown 格式规范

**提示框：**
```markdown
> 💡 **提示**: 有用的建议

> ⚠️ **注意**: 重要警告

> 📝 **说明**: 补充信息
```

**表格：**
```markdown
| 特性 | 选项 A | 选项 B |
|------|--------|--------|
| 速度 | 快 | 慢 |
```

**代码块：**
````markdown
```python
# Python 代码
```

```bash
# Shell 命令
```
````

## 使用的工具

| 工具 | 用途 |
|------|------|
| `Bash` | 运行 `hexo new` 命令 |
| `mcp__plugin_ecc_github__get_file_contents` | 获取 GitHub README/文档 |
| `mcp__plugin_ecc_exa__web_search_exa` | 搜索教程/对比文章 |
| `mcp__plugin_ecc_context7__query-docs` | 获取官方文档 |
| `Read` | 读取创建的博客文件 |
| `Write` | 写入博客内容 |

## 使用场景

### 何时调用本技能
1. **写技术博客**: 当用户需要撰写关于框架、库、工具的技术文章时
2. **写入门教程**: 当用户需要创建 getting-started 指南时
3. **写框架对比**: 当用户需要对比多个技术方案时
4. **写项目文档**: 当用户需要为开源项目撰写文档时

### 调用示例
1. 用户说："写一篇关于 React Hooks 的博客"
2. 用户说："帮我写一篇 Qlib 量化框架的教程"
3. 用户说："写一篇对比 Backtrader 和 VnPy 的文章"

## 文件命名规范

```
source/_posts/YYYY-MM-DD-{中文标题}.md
```

示例：
- `2026-05-05-microsoft-qlib-量化投资AI框架实战教程.md`
- `2026-04-19-graphify入门指南.md`
- `2026-04-27-superpowers入门指南.md`

## 提交消息规范

从 git 历史分析得出的提交消息模式：
- `写文档` / `写博客` - 新博客文章
- `新增文档` - 添加新文档
- `更新文档` - 更新现有文章
- `修改` - 小幅修改

## 相关技能

- `professor-synapse` - 用于复杂研究任务
- `ecc:article-writing` - 通用文章写作模式
- `ecc:docs` - 文档查询

## 注意事项

1. **先读后写**: 使用 `Read` 工具读取创建的文件后再写入内容
2. **多源研究**: 结合 GitHub、Exa、Context7 获取全面信息
3. **代码完整**: 确保所有代码示例可独立运行
4. **SEO 优化**: 在 title、headings、description 中包含关键词
5. **目标读者**: 根据读者背景调整技术术语的使用

## 更新日志

### v1.0.0 (初始版本)
- 实现完整博客写作流程
- 支持教程型和评测型文章结构
- 集成 GitHub API、Exa、Context7 研究工具
- 提供质量检查清单
