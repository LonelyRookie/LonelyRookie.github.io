---
title: Hatchling实战：Python现代构建后端完全指南
date: 2026-05-24 18:00:00
tags:
  - Python
  - hatchling
  - 打包
categories:
  - Python
description: 全面掌握PyPA官方推荐的现代构建后端hatchling：内置版本管理、包发现、多构建目标、插件系统、构建钩子，以及从setuptools迁移的完整路径。
abbrlink: 882026546
---

## 前言：为什么 PyPA 要自己造一个构建后端？

Python 打包世界很长一段时间只有 setuptools。它功能强大，但问题也明显：

- `setup.py` 本质是**可执行代码**，有安全风险
- 配置分散在 `setup.py` + `setup.cfg` + `MANIFEST.in` 中
- 构建速度慢（每次都要执行 Python 代码来解析配置）

PyPA 的目标是：**一个纯声明式的、PEP 621 原生的、快速且可扩展的构建后端**。这就是 hatchling。

> **hatchling 的定位**：setuptools 的现代化替代品。不是"又一个新的打包工具"，而是"PyPA 认为 Python 打包该有的样子"。

它与 [Hatch](https://hatch.pypa.io/) 的关系：Hatch 是项目管理器（类似 Poetry），hatchling 是其中的构建引擎。但你**完全不需要安装 Hatch**，hatchling 可以作为独立构建后端使用。

---

## 一、快速开始：3 行配置跑起来

### 1.1 最小 pyproject.toml

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-package"
version = "0.1.0"
```

```bash
pip install build
python -m build
# 输出: dist/my_package-0.1.0-py3-none-any.whl
#         dist/my_package-0.1.0.tar.gz
```

就这三行——不需要 `setup.py`，不需要 `setup.cfg`，不需要 `MANIFEST.in`。hatchling 自动发现 `src/` 或根目录下的包。

### 1.2 带依赖和入口点的完整配置

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-cli"
version = "0.1.0"
description = "A hatchling-powered CLI"
readme = "README.md"
requires-python = ">=3.10"
license = {text = "MIT"}
authors = [{name = "You", email = "you@example.com"}]
dependencies = [
    "click>=8.1",
    "rich>=13.0",
]

[project.optional-dependencies]
dev = ["pytest>=8.0", "ruff>=0.8"]

[project.scripts]
mycli = "my_cli.main:cli"

[project.urls]
Repository = "https://github.com/you/my-cli"
```

```bash
pip install .
mycli --help
```

---

## 二、版本管理：hatchling 最亮眼的内置功能

setuptools 需要额外安装 `setuptools-scm` 来管理版本，hatchling **内置了多种版本方案**。

### 2.1 静态版本（最简单）

```toml
[project]
version = "1.0.0"
```

### 2.2 从文件读取（推荐）

```toml
[project]
dynamic = ["version"]          # 声明版本是动态的

[tool.hatch.version]
path = "src/my_package/__init__.py"
```

```python
# src/my_package/__init__.py
__version__ = "1.0.0"
```

### 2.3 正则提取（灵活）

```toml
[tool.hatch.version]
path = "src/my_package/__init__.py"
pattern = "VERSION = '(?P<version>[^']+)'"
```

```python
# src/my_package/__init__.py
VERSION = '1.0.0'
```

### 2.4 从 Git tag 自动推断

```toml
[tool.hatch.version]
source = "vcs"                # 从 git tag 读取
```

```bash
git tag v1.0.0
python -m build
# 自动识别版本: 1.0.0

# 未打 tag 的 commit 自动生成 dev 版本
git commit -m "wip"
python -m build
# 输出: my_package-1.0.1.dev2+g3a8b2f1...
```

### 2.5 版本方案对比

| 方案 | 配置 | 适用场景 |
|------|------|---------|
| 静态 | `version = "1.0.0"` | 简单项目 |
| 文件读取 | `path = "xxx/__init__.py"` | 大多数项目 |
| 正则提取 | `path + pattern` | 非标准格式 |
| Git tag | `source = "vcs"` | CI/CD 自动化发布 |
| 自定义 | `source = "code"` + 回调 | 复杂场景 |

---

## 三、包发现：零配置的艺术

### 3.1 默认行为

hatchling 默认**自动发现**，你什么都不用配：

```bash
my-project/
├── pyproject.toml
└── src/
    └── my_package/          # ← 自动被发现
        └── __init__.py
```

### 3.2 手动指定（src-layout）

```toml
[tool.hatch.build.targets.wheel]
packages = ["src/my_package"]
```

```toml
# 多个包
[tool.hatch.build.targets.wheel]
packages = [
    "src/my_core",
    "src/my_plugins",
]
```

### 3.3 强制只打包纯 Python 包

```toml
[tool.hatch.build.targets.wheel]
only-packages = true
```

### 3.4 排除文件

```toml
[tool.hatch.build.targets.wheel]
exclude = ["src/my_package/tests", "src/my_package/_dev"]

[tool.hatch.build.targets.sdist]
exclude = ["tests", "docs", ".github"]
```

---

## 四、构建目标：wheel 和 sdist 的精细控制

### 4.1 独立配置两种目标

```toml
# wheel 配置（用户 pip install 时用）
[tool.hatch.build.targets.wheel]
packages = ["src/my_package"]
only-packages = true

# sdist 配置（源码分发）
[tool.hatch.build.targets.sdist]
include = [            # 额外包含
    "/tests",
    "/docs",
    "/.github",
]
exclude = [
    "*.pyc",
    "__pycache__",
]
```

### 4.2 数据文件自动包含

hatchling 默认使用 VCS（git）来确定哪些文件属于项目，会自动包含**被 git 跟踪的所有文件**。这也意味着：

```bash
# 被 gitignore 的文件不会进入 sdist
git status  # 确认哪些文件会被包含

# 强制包含（覆盖 gitignore）
[tool.hatch.build.targets.sdist]
include = ["/data/important.csv"]

# 强制排除
[tool.hatch.build.targets.sdist]
exclude = ["/tests/fixtures/large_files"]
```

### 4.3 构建时强制版本检查

```toml
[tool.hatch.build]
require-runtime-dependencies = true  # 构建时检查运行时依赖是否可用
skip-excluded-dirs = true            # 跳过被 VCS 忽略的目录
```

---

## 五、构建钩子（Build Hooks）：在构建流程中注入自定义逻辑

这是 hatchling 的高级特性，允许你在构建的特定阶段执行自定义代码。

### 5.1 钩子类型

| 钩子 | 触发时机 | 用途 |
|------|---------|------|
| `version` | 读取版本时 | 自定义版本获取逻辑 |
| `initialize` | 构建开始前 | 准备工作（生成文件、下载资源） |
| `metadata` | 元数据收集后 | 动态修改项目元数据 |
| `finalize` | 构建完成后 | 后处理（压缩、校验） |

### 5.2 实战：构建前自动生成版本文件

```toml
[tool.hatch.build.hooks.custom]
path = "hatch_build.py"     # 指向你的钩子脚本
```

```python
# hatch_build.py
import datetime
from hatchling.builders.hooks.plugin.interface import BuildHookInterface

class CustomHook(BuildHookInterface):
    def initialize(self, version, build_data):
        """构建前：生成 _version.py"""
        output_path = self.root / "src" / "my_package" / "_version.py"
        today = datetime.date.today().isoformat()
        output_path.write_text(
            f'__version__ = "{version}"\n'
            f'__build_date__ = "{today}"\n'
        )

    def finalize(self, version, build_data, artifact_path):
        """构建后：打印构建信息"""
        print(f"Built {artifact_path} (version {version})")
```

### 5.3 使用第三方钩子

```toml
[build-system]
requires = ["hatchling", "hatch-odoo"]

[tool.hatch.build.hooks.odoo-addons]
```

安装 `hatch-odoo` 后，它自动注册了 `odoo-addons` 钩子，会在构建时处理 Odoo 模块的特殊需求。

---

## 六、插件系统

hatchling 的插件架构非常灵活：

```
构建钩子（Build Hooks）
├── CustomHook    # 自定义 Python 脚本
├── hatch-vcs     # 增强 VCS 集成
├── hatch-odoo    # Odoo 模块支持
└── hatch-mypyc   # mypyc 编译支持

版本源（Version Sources）
├── static    # 内置：静态版本
├── code      # 内置：从代码文件读取
├── regex     # 内置：正则提取
├── vcs       # 内置：从 Git tag 读取
└── 自定义    # 实现 VersionSourceInterface

元数据钩子（Metadata Hooks）
└── 自定义    # 实现 MetadataHookInterface
```

---

## 七、高级配置

### 7.1 共享库 / 数据目录

```toml
[tool.hatch.build.targets.wheel.shared-data]
"extra-scripts/my-helper" = "share/my_package/helper"
```

构建后，`extra-scripts/my-helper` 会被复制到 wheel 的 `share/my_package/helper` 位置。

### 7.2 强制包含特定文件到 wheel

```toml
[tool.hatch.build.targets.wheel.force-include]
"src/my_package/py.typed" = "my_package/py.typed"  # PEP 561 类型标记
"README.md" = "my_package/README.md"
```

### 7.3 环境标记

```toml
[tool.hatch.build.targets.wheel]
bypass-selection = true    # 跳过环境标记筛选

# 仅在特定环境下包含
[[tool.hatch.build.targets.wheel.force-include]]
path = "src/my_package/_win32.pyd"
dest = "my_package/_win32.pyd"
condition = "sys_platform == 'win32'"
```

### 7.4 严格模式

```toml
[tool.hatch.build]
strict-naming = true       # 包名不符合规范时报错而非警告
```

---

## 八、实战案例：一个完整的 CLI 项目

### 8.1 项目结构

```bash
weather-cli/
├── pyproject.toml         # 唯一配置文件
├── README.md
├── LICENSE
├── src/
│   └── weather_cli/
│       ├── __init__.py    # __version__ = "0.1.0"
│       ├── cli.py
│       ├── api.py
│       ├── formatters.py
│       ├── py.typed       # PEP 561 类型标记
│       └── templates/
│           ├── report.html
│           └── summary.txt
└── tests/
    ├── __init__.py
    ├── test_cli.py
    └── test_api.py
```

### 8.2 pyproject.toml

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "weather-cli"
description = "A command-line weather tool built with hatchling"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.10"
authors = [{name = "You", email = "you@example.com"}]
dynamic = ["version"]

dependencies = [
    "requests>=2.28,<3.0",
    "click>=8.1,<9.0",
    "rich>=13.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=6.0",
    "ruff>=0.8",
    "mypy>=1.14",
]

[project.scripts]
weather = "weather_cli.cli:main"

[project.urls]
Repository = "https://github.com/you/weather-cli"
Documentation = "https://weather-cli.readthedocs.io"
Issues = "https://github.com/you/weather-cli/issues"

# === hatchling 配置 ===

[tool.hatch.version]
path = "src/weather_cli/__init__.py"

[tool.hatch.build.targets.wheel]
packages = ["src/weather_cli"]

[tool.hatch.build.targets.wheel.force-include]
"src/weather_cli/py.typed" = "weather_cli/py.typed"

[tool.hatch.build.targets.sdist]
exclude = ["tests", ".github"]

# === 工具配置（全部在这里） ===

[tool.ruff]
line-length = 100
target-version = "py310"

[tool.mypy]
python_version = "3.10"
strict = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

### 8.3 构建与发布

```bash
# 开发模式
pip install -e ".[dev]"

# 运行测试
pytest tests/ -v

# 构建
python -m build

# 检查
twine check dist/*

# 发布
twine upload dist/*
```

---

## 九、从 setuptools 迁移到 hatchling

### 9.1 对照表

| setuptools | hatchling |
|-----------|-----------|
| `find_packages(where="src")` | `packages = ["src/my_package"]` |
| `package_dir={"": "src"}` | 自动处理，无需声明 |
| `setuptools-scm` | `[tool.hatch.version] source = "vcs"` (内置) |
| `include_package_data=True` | VCS 自动跟踪（默认） |
| `MANIFEST.in` | 不再需要（VCS 自动发现） |
| `ext_modules=[Extension(...)]` | ❌ 不支持，请留在 setuptools |
| `entry_points={"console_scripts": ...}` | `[project.scripts]`（PEP 621） |

### 9.2 迁移步骤

```bash
# 1. 确保你的项目是 src-layout
# 2. 删除 setup.py, setup.cfg, MANIFEST.in
# 3. 在 pyproject.toml 中替换配置

# 检查迁移效果
python -m build
pip install dist/*.whl
# 一切正常即可提交
```

> ⚠️ **C 扩展项目不能迁移**。hatchling 不处理 C 编译，这种情况必须留在 setuptools。

---

## 十、与其他构建后端对比

| 特性 | hatchling | setuptools | flit | PDM backend |
|------|-----------|------------|------|-------------|
| PEP 621 原生 | ✅ | ⚠️ | ✅ | ✅ |
| 构建速度 | ✅ 快 | ⚠️ 慢 | ✅ 最快 | ✅ 快 |
| 版本管理 | ✅ **内置 4 种** | ⚠️ 需 setuptools-scm | ✅ 内置 | ✅ 内置 |
| 构建钩子 | ✅ **一流** | ✅ cmdclass | ❌ | ⚠️ |
| VCS 集成 | ✅ **默认** | ⚠️ MANIFEST.in | ✅ | ✅ |
| C 扩展 | ❌ | ✅ | ❌ | ⚠️ |
| 配置复杂度 | ✅ 低 | ⚠️ 中 | ✅ 最低 | ⚠️ 中 |
| 插件生态 | ✅ 增长中 | ✅ 最丰富 | ❌ | ✅ |

---

## 十一、常见问题

### 11.1 "hatchling 找不到我的包"

```bash
# hatchling 的自动发现基于 VCS（Git）
# 确保你的包目录被 git 跟踪
git ls-files src/

# 如果不想依赖 VCS，手动声明
[tool.hatch.build.targets.wheel]
packages = ["src/my_package"]
```

### 11.2 "构建后 wheel 是空的"

```toml
# 手动指定的包路径不是目录
[tool.hatch.build.targets.wheel]
packages = ["src/my_package"]  # 必须是目录路径，不是模块名
```

### 11.3 "数据文件没被包含"

```bash
# hatchling 只包含被 git 跟踪的文件
git add src/my_package/templates/
git commit -m "add templates"
python -m build
```

### 11.4 "动态版本不生效"

```toml
# 必须先声明 dynamic = ["version"]
[project]
dynamic = ["version"]

# 再配置 hatchling 版本源
[tool.hatch.version]
path = "src/my_package/__init__.py"
```

---

## 总结

### hatchling 的核心优势

- **零配置**: 开箱即用，VCS 自动发现文件
- **内置版本管理**: 静态 / 文件 / 正则 / Git tag，四种方案无需额外插件
- **构建钩子**: 在构建流程任意阶段注入自定义逻辑
- **PEP 621 原生**: 从一开始就遵循 PyPA 标准
- **快速**: 纯声明式解析，比 setuptools 快数倍

### 什么时候用 hatchling？

| 场景 | 建议 |
|------|------|
| 纯 Python 新项目 | ✅ **首选用 hatchling** |
| 需要 C 扩展 | ❌ 必须用 setuptools |
| 存量 setuptools 项目 | ⚠️ 看情况，纯 Python 可迁 |
| 需要自定义构建逻辑 | ✅ 用 hatchling 的 build hooks |

> 📋 **参考资料**:
> - [Hatchling 官方文档](https://hatch.pypa.io/latest/config/build/)
> - [Hatch 项目管理器](https://hatch.pypa.io/)
> - [PEP 621 — pyproject.toml 项目元数据](https://peps.python.org/pep-0621/)
> - [PyPA Build 工具](https://github.com/pypa/build)
