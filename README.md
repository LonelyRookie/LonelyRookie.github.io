
## 部署

```bash
hexo clean
hexo generate
hexo deploy
```

或简写：

```bash
hexo cl && hexo g && hexo d
```

## 日常发布流程

```bash
# 1. 创建文章
hexo new "文章标题"

# 2. 编辑文章（使用 Markdown 编辑器）

# 3. 本地预览
hexo clean && hexo generate && hexo server

# 4. 确认无误后部署
hexo clean && hexo generate && hexo deploy
```

注：使用`hexo deploy`命令后，会把静态资源文件推送到`master`分支。

```yml
deploy:
  type: git
  repo: git@github.com:LonelyRookie/LonelyRookie.github.io.git
  branch: master
```

--- 

`master`分支存放的是打包后的静态资源文件

`feature/dev`分支存放的是工程代码


## 将本地工程推送到远程仓库

```shell
git push origin HEAD:feature/dev
```


## 提示词

```text

  使用 professor-synapse 技能，为 GitHub 项目 {{document}} 撰写一篇高质量技术博客。

  ## 任务流程

  1. **创建文档**
     - 运行 `hexo new "<博客标题>"` 创建 Markdown 文件
     - 标题建议：突出项目核心价值或技术亮点

  2. **研究阶段**（使用 professor-synapse）
     - 阅读 GitHub 项目的 README、文档、核心源码
     - 分析项目架构、技术栈、设计模式
     - 搜索相关技术背景和同类项目对比

  3. **博客写作**

     **目标读者**：[初学者/中级开发者/高级开发者]

     **技术角度**：[入门教程 / 源码分析 / 架构设计 / 最佳实践 / 项目实战]

     **语言**：[中文]

     **结构要求**：
     - 引言：项目背景、解决什么问题、为什么值得关注
     - 核心概念：关键技术点解释
     - 实战/源码分析：带代码示例的深入讲解
     - 最佳实践：使用建议和避坑指南
     - 总结：核心收获和适用场景

     **内容要求**：
     - 包含可运行的代码片段（带注释）
     - 包含架构图或流程图（如适用）
     - 引用官方文档或源码链接

  4. **质量检查**
     - 代码示例是否可运行
     - 技术术语是否有解释
     - 逻辑是否连贯
     - 是否有清晰的 takeaways

  ## 验收标准
  - 博客结构完整，层次清晰
  - 代码示例准确且有注释
  - 技术深度匹配目标读者
  - 无事实性错误
```