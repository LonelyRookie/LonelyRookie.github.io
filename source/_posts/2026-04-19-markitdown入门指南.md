---
title: markitdown入门指南
date: 2026-04-19 22:21:02
tags:
  - python
  - 文档处理
categories:
  - python
---

## 简介

[MarkItDown](https://github.com/microsoft/markitdown) 是微软开源的轻量级 Python 工具，用于将各种文件格式转换为 Markdown。它专为 LLM（大语言模型）和文本分析管道设计，能够保留重要的文档结构（标题、列表、表格、链接等）。

### 为什么选择 Markdown？

Markdown 接近纯文本，标记最少，但仍能表示重要的文档结构。主流 LLM（如 GPT-4o）原生"讲"Markdown，在响应中经常自发使用 Markdown 格式。这表明它们在大量 Markdown 格式文本上进行了训练，能够很好地理解它。作为额外好处，Markdown 约定也非常节省 token。

### 支持的格式

- **PDF** - 便携式文档格式
- **PowerPoint** - 演示文稿
- **Word** - 文档
- **Excel** - 电子表格
- **Images** - 图片（EXIF 元数据和 OCR）
- **Audio** - 音频（EXIF 元数据和语音转录）
- **HTML** - 网页
- **Text-based formats** - CSV、JSON、XML
- **ZIP files** - 遍历内容
- **Youtube URLs** - YouTube 视频
- **EPubs** - 电子书
- 以及更多！

## 快速入门（5分钟）

### 环境要求

- Python 3.10 或更高版本
- 推荐使用虚拟环境

### 安装

```bash
# 安装所有可选依赖
pip install 'markitdown[all]'

# 或仅安装特定格式的依赖
pip install 'markitdown[pdf, docx, pptx]'
```

### 基本使用

**命令行方式：**

```bash
# 转换文件并输出到标准输出
markitdown path-to-file.pdf > document.md

# 使用 -o 指定输出文件
markitdown path-to-file.pdf -o document.md

# 管道方式
cat path-to-file.pdf | markitdown
```

**Python API 方式：**

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False)
result = md.convert("test.xlsx")
print(result.text_content)
```

## 详细教程

### 创建虚拟环境

**标准 Python：**

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# 或 .venv\Scripts\activate  # Windows
```

**使用 uv：**

```bash
uv venv --python=3.12 .venv
source .venv/bin/activate
# 注意：使用 'uv pip install' 而非 'pip install'
```

**使用 Anaconda：**

```bash
conda create -n markitdown python=3.12
conda activate markitdown
```

### 可选依赖说明

| 依赖组 | 说明 |
| ------ | ------------------------------------ |
| `[all]` | 安装所有可选依赖 |
| `[pptx]` | PowerPoint 文件支持 |
| `[docx]` | Word 文件支持 |
| `[xlsx]` | Excel 文件支持 |
| `[xls]` | 旧版 Excel 文件支持 |
| `[pdf]` | PDF 文件支持 |
| `[outlook]` | Outlook 邮件支持 |
| `[az-doc-intel]` | Azure Document Intelligence 支持 |
| `[audio-transcription]` | 音频转录支持（wav、mp3） |
| `[youtube-transcription]` | YouTube 视频转录支持 |

### 插件系统

MarkItDown 支持第三方插件扩展功能：

```bash
# 列出已安装的插件
markitdown --list-plugins

# 启用插件
markitdown --use-plugins path-to-file.pdf
```

**搜索插件：** 在 GitHub 上搜索 `#markitdown-plugin` 标签。

## 进阶用法

### 使用 LLM 进行图片描述

为图片和演示文稿中的图片提供智能描述：

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(
    llm_client=client,
    llm_model="gpt-4o",
    llm_prompt="可选的自定义提示词"
)
result = md.convert("example.jpg")
print(result.text_content)
```

### Azure Document Intelligence 集成

使用微软 Azure 文档智能服务进行转换：

**命令行：**

```bash
markitdown path-to-file.pdf -o document.md -d -e "<document_intelligence_endpoint>"
```

**Python API：**

```python
from markitdown import MarkItDown

md = MarkItDown(docintel_endpoint="<document_intelligence_endpoint>")
result = md.convert("test.pdf")
print(result.text_content)
```

> Azure Document Intelligence 资源设置详见[官方文档](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/how-to-guides/create-document-intelligence-resource)。

### markitdown-ocr 插件

为 PDF、DOCX、PPTX 和 XLSX 添加 OCR 支持，从嵌入图片中提取文本：

```bash
pip install markitdown-ocr
pip install openai  # 或任何 OpenAI 兼容客户端
```

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.text_content)
```

### Docker 支持

```bash
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

### MCP 服务器集成

MarkItDown 提供 MCP（Model Context Protocol）服务器，可与 Claude Desktop 等 LLM 应用集成。详见 [markitdown-mcp](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp)。

## 常见问题

### 版本升级注意事项（0.0.1 → 0.1.0）

1. **依赖组织变化**：依赖现在按可选功能组组织，使用 `pip install 'markitdown[all]'` 获得向后兼容行为
2. **convert_stream() 变化**：现在需要二进制文件对象（如以二进制模式打开的文件或 io.BytesIO）
3. **DocumentConverter 接口变化**：从文件路径读取改为文件流读取，不再创建临时文件

### 与 textract 的区别

MarkItDown 与 [textract](https://github.com/deanmalmgren/textract) 类似，但专注于保留重要的文档结构为 Markdown（标题、列表、表格、链接等）。输出通常适合人类阅读，但主要用于文本分析工具消费。

### 开发与测试

```bash
# 克隆仓库
git clone git@github.com:microsoft/markitdown.git
cd markitdown/packages/markitdown

# 安装 hatch 并运行测试
pip install hatch
hatch shell
hatch test

# 运行 pre-commit 检查
pre-commit run --all-files
```

## 总结

MarkItDown 是微软开源的强大文档转换工具，特别适合：

- **LLM 应用开发**：将各种文档转换为 LLM 友好的 Markdown 格式
- **文档分析管道**：保留结构化信息的文本提取
- **RAG 系统**：为检索增强生成准备文档数据
- **内容迁移**：批量转换文档格式

核心优势：

- 支持 10+ 种文件格式
- 保留文档结构（标题、列表、表格）
- 可选 LLM 增强的图片描述
- Azure Document Intelligence 集成
- 插件系统扩展
- MCP 服务器支持

**GitHub 仓库：** <https://github.com/microsoft/markitdown>
