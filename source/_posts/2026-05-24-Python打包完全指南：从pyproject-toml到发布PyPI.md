---
title: Python打包完全指南：从pyproject.toml到发布PyPI
date: 2026-05-24 16:06:01
tags:
  - Python
  - 打包
  - pyproject.toml
categories:
  - Python
description: 一篇全面的Python打包实战指南，从pyproject.toml配置、构建后端选择(setuptools/hatchling/flit/PDM/Poetry)、到发布PyPI和版本管理，帮助开发者掌握现代标准化的Python打包流程。
abbrlink: 662026244
---

## 前言：为什么需要现代化Python打包？

如果你写过Python项目，你一定遇到过这些问题：

- `setup.py` / `setup.cfg` / `requirements.txt` / `MANIFEST.in` 蔚为壮观，到底用哪个？
- 依赖管理器混战：pip、pip-tools、Poetry、Pipenv各不相同
- 想发布到PyPI，但不知道该怎么开始
- 换电脑后项目不能一键安装，环境搭建痛苦不堪
- 项目维护者要维护好几个配置文件，头大如斗

这些问题的根源是：**Python打包生态系统长期碎片化**。2018年，Python社区通过 [PEP 517](https://peps.python.org/pep-0517/) 和 [PEP 518](https://peps.python.org/pep-0518/) 开启了现代化改革，随后 [PEP 621](https://peps.python.org/pep-0621/) 正式将 `pyproject.toml` 确立为**唯一、统一的项目元数据文件**，从此告别了多文件混战的时代。

> 📋 **核心变化**: 从 `setup.py` + `setup.cfg` + `requirements.txt` → **`pyproject.toml` 一文档统治**

本文基于 [Python Packaging Authority (PyPA) 官方指南](https://packaging.python.org/en/latest/guides/)，将带你从零开始，完成从项目配置、构建后端选择、到发布PyPI的全流程。

---

## 一、一句话理解：setup.py 为什么被 pyproject.toml 取代？

假设你的项目是一部汽车：

| 角色 | 旧世界 | 新世界 |
|------|--------|--------|
| 厂商说明书 | `setup.py` / `setup.cfg` 多份文件 | `pyproject.toml` 唯一文档 |
| 发动机 | 必须指定 setuptools | 自选构建后端(hatchling/flit/PDM/Poetry) |
| 装配清单 | `requirements.txt` / `Pipfile` | `[project.dependencies]` |
| 质检单 | 无 | 内置 `[project.scripts]` 或者插件入口 |
| 安全帶 | 必须手动检查 | `[build-system]` 明确声明 |

`pyproject.toml` 不是简单格式变更，而是 **架构性改进**：从"工具锁定"到"厂商中立"。

```toml
# 时代的眼泪：旧 setup.py
# from setuptools import setup
# setup(
#     name="my-project",
#     version="0.1.0",  # 硬编码版本
#     install_requires=["requests>=2.28"],
#     entry_points={"console_scripts": ["mycli=my_project.cli:main"]},
# )

# 新世界：pyproject.toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = ["requests>=2.28"]

[project.scripts]
mycli = "my_project.cli:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

> 💡 **提示**: 如果你还在用 `setup.py`，现在就是迁移的最佳时机。Python 3.12+ 甚至开始发布 DeprecationWarning。

---

## 二、核心价值：从"能跑"到"好用"

`pyproject.toml` 解决的不只是"打包"问题，而是面向**开发者**、**发布者**、**用户**三方面的体验问题：

### 2.1 面向开发者：一个文件搞定所有

`pyproject.toml` 统一管理：
- 项目元数据(名称、版本、作者、描述)
- 依赖声明(runtime + build + dev + optional)
- 构建系统配置(后端、前置命令、插件)
- 工具配置(linter、test、type checker、tools一站式)

### 2.2 面向发布者：标准化流程

- 不再需要执行 `setup.py`，避免任意代码执行风险
- `python -m build` 统一构建入口，支持多种构建后端
- `twine upload` 一键发布到PyPI

### 2.3 面向用户：开箱即用

`pip install` 无需先解读 `setup.py`，也不需要安装构建依赖（如setuptools）。用户只需要Python和pip，其余全部自动处理。

> 📋 **说明**: 如果你的项目仅仅是一个脚本，不需要发布，可能一个 `requirements.txt` 就够用了。但只要你计划分享项目或发布包，`pyproject.toml` 就是最佳的起点。

---

## 三、快速开始：5分钟创建你的第一个包

### 3.1 项目初始化

我们从零开始创建一个完整的项目。

```bash
# 创建项目目录
mkdir my-awesome-tool && cd my-awesome-tool

# 创建虚拟环境
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 创建源代码目录
mkdir -p src/my_awesome_tool
```

### 3.2 核心源代码

```python
# src/my_awesome_tool/__init__.py
"""
my_awesome_tool - A demonstration package for modern Python packaging.
"""
__version__ = "0.1.0"

# src/my_awesome_tool/core.py
def greet(name: str = "World") -> str:
    """Say hello to someone."""
    return f"Hello, {name}! 👋"

# src/my_awesome_tool/cli.py
import argparse
from .core import greet

def main():
    parser = argparse.ArgumentParser(description="My Awesome Tool")
    parser.add_argument("--name", default="World", help="Who to greet")
    args = parser.parse_args()
    print(greet(args.name))

if __name__ == "__main__":
    main()
```

### 3.3 写下 `pyproject.toml`

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-awesome-tool"
version = "0.1.0"
description = "A demonstration package for modern Python packaging."
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.10"
authors = [{name = "Your Name", email = "you@example.com"}]
keywords = ["demo", "packaging"]
classifiers = [
    "Development Status :: 3 - Alpha",
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]
dependencies = [
    "rich>=13.0",  # 为了漂亮的终端输出
]

[project.optional-dependencies]
dev = ["pytest>=7.0", "ruff>=0.8", "mypy>=1.14"]

[project.scripts]
mycli = "my_awesome_tool.cli:main"

[tool.hatch.build.targets.wheel]
packages = ["src/my_awesome_tool"]

[tool.ruff]
line-length = 100
target-version = "py310"

[tool.mypy]
python_version = "3.10"
strict = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

### 3.4 构建、安装与测试

```bash
# 安装构建工具
pip install build

# 构建包
python -m build
# 输出: dist/my_awesome_tool-0.1.0-py3-none-any.whl
#         dist/my_awesome_tool-0.1.0.tar.gz

# 安装测试
pip install dist/my_awesome_tool-0.1.0-py3-none-any.whl

# 测试命令行工具
mycli --name Codex
# 输出: Hello, Codex! 👋
```

> ⚠️ **注意**: 始终在**虚拟环境**中测试，避免污染全局Python环境。

---

## 四、pyproject.toml 详解

### 4.1 [build-system] — 构建系统声明

这是整个文件的引擎声明。告诉pip你需要什么工具来构建项目：

```toml
[build-system]
requires = ["hatchling"]             # 构建依赖
build-backend = "hatchling.build"    # 构建后端入口
```

| 字段 | 必须 | 说明 |
|------|------|------|
| `requires` | 是 | 构建工具的PyPI依赖列表 |
| `build-backend` | 是 | 后端入口字符串（`module:object`格式） |
| `backend-path` | 否 | 如果后端在项目内部 |

**常见构建后端配置**：

```toml
# hatchling（推荐）
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# setuptools（兼容性最强）
[build-system]
requires = ["setuptools>=77.0", "wheel"]
build-backend = "setuptools.build_meta"

# flit（最简单）
[build-system]
requires = ["flit_core>=3.12"]
build-backend = "flit_core.buildapi"

# PDM（PEP 621原生支持）
[build-system]
requires = ["pdm-backend"]
build-backend = "pdm.backend"
```

### 4.2 [project] — 项目元数据

这是 `pyproject.toml` 的**核心**，遵循PEP 621规范。

```toml
[project]
name = "my-package"                 # 必填：PyPI唯一标识
version = "1.0.0"                   # 必填：遵循SemVer
description = "A short summary"     # 必填：一句话描述
readme = "README.md"                # 描述文件路径
license = {text = "MIT"}            # 许可证声明
requires-python = ">=3.10"          # Python版本约束
authors = [
    {name = "Author Name", email = "author@example.com"},
]
maintainers = [
    {name = "Maintainer Name", email = "maint@example.com"},
]
keywords = ["keyword1", "keyword2"]
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]

# core dependencies
dependencies = [
    "requests>=2.28,<3.0",
    "click>=8.1",
]

# extra features
[project.optional-dependencies]
all = ["mypackage[cli,dev]"]
cli = ["rich>=13.0", "typer>=0.15"]
dev = ["pytest>=8.0", "ruff>=0.8", "mypy>=1.14"]

# entry points
[project.scripts]
mycli = "my_package.cli:main"

[project.gui-scripts]
mygui = "my_package.gui:main"

# dynamic fields (if not declared in [project], must be in dynamic)
dynamic = ["version"]  # 允许构建时动态生成

# URLs for PyPI page
[project.urls]
Homepage = "https://github.com/user/my-package"
Documentation = "https://my-package.readthedocs.io"
Repository = "https://github.com/user/my-package.git"
Issues = "https://github.com/user/my-package/issues"
```

**`dynamic` 字段的含义**：

如果你不想在 `pyproject.toml` 中硬编码版本号，可以声明为动态：

```toml
[project]
name = "my-package"
dynamic = ["version", "readme"]  # 由构建后端动态生成
```

```toml
# hatchling 从文件读取版本
[tool.hatch.version]
path = "src/my_package/__init__.py"

# setuptools-scm 从 git tag 自动生成版本
[tool.setuptools_scm]
```

### 4.3 [tool] — 工具配置命名空间

所有第三方工具的配置都应该放在 `[tool.xxx]` 下，这是约定俗成的规范：

```toml
# Ruff linter
[tool.ruff]
line-length = 100
target-version = "py310"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

# mypy type checker
[tool.mypy]
python_version = "3.10"
strict = true

# pytest
[tool.pytest.ini_options]
minversion = "7.0"
testpaths = ["tests"]

# coverage
[tool.coverage.run]
branch = true
source = ["my_package"]

# Black formatter
[tool.black]
line-length = 100
target-version = ["py310"]
```

> 💡 **提示**: 把 `.flake8`、`mypy.ini`、`pytest.ini`、`.coveragerc`等配置全部迁移到 `pyproject.toml` 中，一个文件统治所有工具配置。

---

## 五、构建后端对比：谁适合你？

现代Python构建后端的职责很简单：把 `pyproject.toml` 转化为Sdist (`.tar.gz`) 和Wheel (`.whl`)。但是，背后的工作量却差异巨大。

| 特性 | hatchling | setuptools | flit | PDM | Poetry |
|------|-----------|------------|------|-----|--------|
| PEP 621 | ✅ 原生 | ✅ | ✅ | ✅ | ⚠️ 自定义 |
| 构建速度 | ✅ 很快 | ⚠️ 慢 | ✅ 最快 | ✅ 快 | ⚠️ 慢 |
| C扩展 | ✅ | ✅ 最佳 | ❌ | ✅ | ❌ |
| 插件 | ✅ | ✅ | ⚠️ 少 | ✅ | ✅ |
| 版本管理 | ✅ | ✅ setuptools-scm | ✅ | ✅ | ✅ |
| 依赖解析 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 学习成本 | ✅ 低 | ⚠️ 中 | ✅ 低 | ⚠️ 中 | ✅ 低 |

### 5.1 实战建议

> 🎨 **默认推荐**: **hatchling** — 速度快、PEP 621原生支持、活跃社区、插件生态丰富。
>
> 🔧 **需要C扩展**: **setuptools** — 唯一对C扩展有完整支持的后端。
>
> ⚡ **最简单**: **flit** — 仅需3行声明，适合纯Python包。
>
> 📦 **想PDM/Poetry体验**: **PDM** / **Poetry** — 依赖管理+构建一体化，但PEP 621支持待完善。

### 5.2 hatchling 配置详解

```toml
[tool.hatch.version]
path = "src/my_package/__init__.py"

[tool.hatch.build.targets.wheel]
packages = ["src/my_package"]

# 排除特定文件
[tool.hatch.build.targets.sdist]
exclude = ["tests/", "docs/", ".github/"]

# 只包含纯Python包
[tool.hatch.build.targets.wheel]
only-packages = true
```

### 5.3 setuptools 配置（兼容向）

```toml
[build-system]
requires = ["setuptools>=77.0", "setuptools-scm>=8.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-c-extension"
dynamic = ["version"]

[tool.setuptools]
packages = ["my_extension"]
package-dir = {"" = "src"}

[tool.setuptools_scm]
```

### 5.4 Poetry 迁移示例

如果你用Poetry，需要做一些转换：

```toml
# Poetry 格式 → PEP 621 格式
# [tool.poetry.dependencies]
# python = "^3.10"
# requests = "^2.28"

# → 转换为:
[project]
requires-python = ">=3.10"
dependencies = ["requests>=2.28"]
```

---

## 六、项目结构最佳实践

### 6.1 src layout vs flat layout

Python项目主要有两种目录结构：

```
# src layout (推荐)
my-project/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── core.py
│       └── cli.py
└── tests/
    └── test_core.py

# flat layout (简单项目可用)
my-project/
├── pyproject.toml
├── README.md
├── my_package/
│   └── __init__.py
└── tests/
```

| 对比维度 | src layout | flat layout |
|-----------|------------|-------------|
| 安装测试 | 强制测试构建后的包 | 可能导入源代码同名包 |
| 多包 | ✅ 自然支持 | ❌ 易混淆 |
| 简单项目 | 增加一层目录 | ✅ 直观 |
| PyPA推荐 | ✅ | ❌ |

**建议**: 新项目起手就用**src layout**，避免将来迁移的麻烦。

### 6.2 完整项目模板

```bash
my-project/
├── pyproject.toml          # 唯一配置文件
├── README.md               # 项目介绍
├── LICENSE                 # 许可证
├── CHANGELOG.md            # 变更日志
├── .gitignore
├── src/
│   └── my_package/
│       ├── __init__.py     # 公开API + __version__
│       ├── py.typed         # PEP 561 标记文件
│       ├── core.py
│       ├── cli.py
│       └── _internal/       # 私有实现
│           └── _helper.py
├── tests/
│   ├── __init__.py
│   └── test_core.py
└── docs/
    └── index.md
```

---

## 七、发布到PyPI：让世界用上你的包

### 7.1 发布前检查清单

发布前确保以下准备就绪：

- [ ] `pyproject.toml` 中 `name` 在PyPI上唯一（先去 [pypi.org](https://pypi.org) 查询）
- [ ] `version` 与已发布版本不冲突
- [ ] `python -m build` 编译无错误
- [ ] `twine check dist/*` 通过校验
- [ ] `README.md` 内容完整且格式正确

### 7.2 发布流程

```bash
# 1. 安装构建和发布工具
pip install build twine

# 2. 清理旧包
rm -rf dist/

# 3. 构建
python -m build

# 4. 校验包
twine check dist/*
# 合格输出: Checking dist/my_package-1.0.0-py3-none-any.whl: PASSED
#              Checking dist/my_package-1.0.0.tar.gz: PASSED

# 5. 上传到TestPyPI（先测试）
twine upload -r testpypi dist/*

# 6. 测试安装
pip install -i https://test.pypi.org/simple/ my-package

# 7. 一切正常后，上传到PyPI
twine upload dist/*
```

### 7.3 设置PyPI凭证

```bash
# ~/.pypirc
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-xxxxxxxxxxxx

[testpypi]
username = __token__
password = pypi-xxxxxxxxxxxx
```

> ⚠️ **安全**: 始终使用 **API Token** 而非密码。在 [pypi.org/manage/account](https://pypi.org/manage/account/) 生成。

### 7.4 CI/CD 自动发布 (GitHub Actions)

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # trusted publishing
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install build
      - run: python -m build
      - uses: pypa/gh-action-pypi-publish@release/v1
```

> 🎨 **Trusted Publishing**: PyPI 支持 GitHub Actions OIDC，不需要密码或Token，只需在PyPI项目设置中添加 Trusted Publisher。

---

## 八、版本管理：Single-sourcing the Version

**切忌**在多个地方硬编码版本号。PyPA提供了几种优雅的解决方案：

### 方案一：setuptools-scm（推荐）

从 `git tag` 自动生成版本：

```toml
[build-system]
requires = ["setuptools>=77.0", "setuptools-scm>=8.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
dynamic = ["version"]

[tool.setuptools_scm]
```

```bash
git tag v1.0.0
python -m build
# 版本自动设置为 1.0.0
```

### 方案二：从 __init__.py 读取

```toml
# hatchling
[tool.hatch.version]
path = "src/my_package/__init__.py"
```

```python
# src/my_package/__init__.py
__version__ = "1.0.0"
```

### 方案三：importlib.metadata (运行时读取)

```python
from importlib.metadata import version, PackageNotFoundError

try:
    __version__ = version("my-package")
except PackageNotFoundError:
    __version__ = "unknown"
```

> ✅ **最佳实践**: 使用 `setuptools-scm` 或 `hatchling` 的动态版本，从单一源头生成版本，避免多处硬编码。

---

## 九、插件和入口点：让你的包可扩展

### 9.1 console_scripts — 命令行入口

最常见的用例：

```toml
[project.scripts]
mycli = "my_package.cli:main"
```

安装后直接可用 `mycli` 命令。

### 9.2 插件系统 — Entry Points

这是Python插件体系的基础，以pytest为例：

```toml
# 你的插件项目
[project.entry-points.pytest11]
my-plugin = "my_plugin.module"
```

```python
# 定义插件
def pytest_addoption(parser):
    parser.addoption("--my-option", action="store_true")
```

安装后，pytest会自动发现并加载你的插件。

### 9.3 创建可定制插件的项目

```toml
[project.entry-points."myapp.plugins"]
json-exporter = "myapp_json.plugin:JSONExporter"
csv-exporter = "myapp_csv.plugin:CSVExporter"
```

```python
# 在主项目中加载插件
from importlib.metadata import entry_points

def load_plugins():
    eps = entry_points(group="myapp.plugins")
    return {ep.name: ep.load() for ep in eps}
```

> 📋 **说明**: Entry points是Python插件生态的核心。pytest、sphinx、flake8、pre-commit等所有支持插件的工具都基于此机制。

---

## 十、依赖管理进阶

### 10.1 依赖类型完整对比

```toml
[build-system]
requires = ["hatchling"]           # 构建依赖（仅构建时需要）

[project]
dependencies = [                     # runtime dependencies
    "requests>=2.28,<3.0",
    "click>=8.1",
]

[project.optional-dependencies]      # extra features
cli = ["rich>=13.0", "typer>=0.15"]
dev = ["pytest>=8.0", "ruff>=0.8", "mypy>=1.14"]
docs = ["sphinx>=8.0", "furo>=2024"]
test = ["pytest>=8.0", "pytest-cov>=6.0"]
```

### 10.2 版本约束语法

| 表达式 | 含义 | 示例 |
|------|------|------|
| `>=2.28` | 最低 | `requests>=2.28` |
| `<3.0` | 最高 | `requests<3.0` |
| `>=2.28,<3.0` | 范围 | 推荐组合使用 |
| `~=2.28.1` | 兼容版 | `~=2.28.1` ≈ `>=2.28.1, <2.29` |
| `==2.28.1` | 精确 | 应用锁定版本 |
| `!=2.28.1` | 排除 | 排除特定版本 |

> ⚠️ **最佳实践**: 始终使用 `>=X,<Y` 范围约束，确保兼容性，同时防止无限升级。

---

## 十一、常见问题与排错

### 11.1 "Why is my package empty after install?"

最常见的原因是构建后端不知道你的源码在哪里：

```toml
# hatchling: 明确指定包路径
[tool.hatch.build.targets.wheel]
packages = ["src/my_package"]

# setuptools: 明确指定 src 布局
[tool.setuptools]
package-dir = {"" = "src"}
packages = ["my_package"]
```

### 11.2 "pip install -e . 失败"

确保 `build-backend` 支持可编辑安装：

```bash
pip install -e ".[dev]"
```

### 11.3 "ModuleNotFoundError: No module named 'my_package'"

检查是否使用了 `src layout`但没有给 build backend 指定包路径。

### 11.4 "Command not found: mycli"

检查 `[project.scripts]` 中的入口是否正确，并重新安装：

```bash
pip install --force-reinstall --no-deps dist/*.whl
```

---

## 十二、工具链推荐：开箱即用的技术栈

| 工具 | 用途 | 配置调用 |
|------|------|----------|
| [build](https://github.com/pypa/build) | 构建包 | `python -m build` |
| [twine](https://twine.readthedocs.io/) | 上传PyPI | `twine upload dist/*` |
| [hatch](https://hatch.pypa.io/) | 项目管理 | `hatch new` / `hatch build` |
| [ruff](https://docs.astral.sh/ruff/) | Lint & Format | `ruff check` / `ruff format` |
| [mypy](https://mypy-lang.org/) | 类型检查 | `mypy src/` |
| [pytest](https://docs.pytest.org/) | 测试 | `pytest` |
| [check-manifest](https://github.com/mgedmin/check-manifest) | 检查sdist包 | `check-manifest` |
| [pip-audit](https://github.com/pypa/pip-audit) | 安全审计 | `pip-audit` |

---

## 总结

### 关键要点

- **一个文件统治**: `pyproject.toml` 统一管理项目元数据、依赖、构建、工具配置
- **构建后端自选**: new project 用 `hatchling`，C扩展用 `setuptools`，简单项目用 `flit`
- **始终虚拟环境**: 隔离开发环境，避免依赖冲突
- **API Token 而非密码**: 发布PyPI应使用API Token
- **src layout**: 避免导入污染，新项目起手就用
- **>=X, <Y 范围兼容**: 所有依赖使用范围约束

### 下一步建议

1. 把你现有的 `setup.py` 项目迁移到 `pyproject.toml`
2. 探索 [Hatch](https://hatch.pypa.io/) 的环境管理
3. 设置 CI/CD 自动发布流水线
4. 深入阅读 [PyPA官方指南](https://packaging.python.org/en/latest/guides/)

> 📋 **参考资料**:
> - [Python Packaging User Guide](https://packaging.python.org/en/latest/guides/)
> - [PEP 621 — Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/)
> - [PEP 517 — A build-system independent format for source trees](https://peps.python.org/pep-0517/)
> - [Hatch Documentation](https://hatch.pypa.io/)
> - [Setuptools Documentation](https://setuptools.pypa.io/)
