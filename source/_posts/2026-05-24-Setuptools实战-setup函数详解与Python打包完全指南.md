---
title: Setuptools实战：setup()函数详解与Python打包完全指南
date: 2026-05-24 17:33:00
tags:
  - Python
  - setuptools
  - 打包
  - pyproject.toml
categories:
  - Python
description: 深入剖析setuptools的setup()函数每一个参数，覆盖包发现、入口点、依赖管理、C扩展编译、数据文件打包，以及从传统setup.py到现代pyproject.toml的完整迁移路径。
abbrlink: 882026545
keywords:
  - setuptools
  - setup.py
  - setup()
  - find_packages
  - Python打包
---

## 前言：setuptools 凭什么统治 Python 打包二十年？

打开 GitHub 上任意一个 Python 项目，你大概率会看到一个 `setup.py`，里面调用了 `setuptools.setup()`。二十年来，这几乎是 Python 打包的"唯一标准"。

即使2026年的今天，PyPA 力推 `pyproject.toml`，**setuptools 仍是构建 C 扩展的唯一成熟选择**，numpy、scipy、pandas、cryptography 等核心库全部依赖它。

本文以 [setuptools 官方 Quickstart](https://setuptools.pypa.io/en/stable/userguide/quickstart.html)（v82.0）为基础，先讲透传统的 `setup()` 函数，再过渡到现代的声明式配置，让你读懂任何 setuptools 项目。

---

## 一、快速入门：5 分钟写出第一个 setup.py

### 1.1 最小项目结构

```bash
my-package/
├── setup.py              # 构建入口
├── README.md
└── my_package/           # 源码
    ├── __init__.py
    └── core.py
```

### 1.2 第一个 setup()

```python
# setup.py
from setuptools import setup

setup(
    name="my-package",
    version="0.1.0",
    description="A short description",
    long_description=open("README.md", encoding="utf-8").read(),
    long_description_content_type="text/markdown",
    author="Your Name",
    author_email="you@example.com",
    url="https://github.com/you/my-package",
    packages=["my_package"],            # 要打包的模块列表
    python_requires=">=3.10",
    install_requires=[
        "requests>=2.28,<3.0",
        "click>=8.0",
    ],
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
    ],
)
```

### 1.3 构建和安装

```bash
# 安装构建工具
pip install build

# 构建（自动下载 setuptools）
python -m build
# 输出: dist/my_package-0.1.0-py3-none-any.whl
#         dist/my_package-0.1.0.tar.gz

# 安装
pip install dist/my_package-0.1.0-py3-none-any.whl
```

> 💡 **提示**: 不需要手动安装 setuptools，`build` 工具会从 `pyproject.toml` 的 `[build-system]` 中自动下载。

### 1.4 配一个 pyproject.toml

即使你用 `setup.py`，也建议加一个最小 `pyproject.toml` 声明构建系统：

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

有了这个文件，`pip install .` 就知道先装 setuptools，再执行构建。

---

## 二、setup() 函数完全参数手册

`setup()` 是 setuptools 的核心，你给他的每一个参数，都定义了你这个"包"的完整面貌。以下是按使用频率排序的完整参数说明。

### 2.1 元数据参数

```python
from setuptools import setup

setup(
    # === 必填 ===
    name="my-package",                    # PyPI 唯一名称（小写、短横线）
    version="1.0.0",                     # 遵循 SemVer

    # === 描述 ===
    description="一句话描述",              # PyPI 搜索结果摘要
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",  # 可选: text/x-rst

    # === 作者 ===
    author="Your Name",
    author_email="you@example.com",
    maintainer="Maintainer Name",
    maintainer_email="maint@example.com",

    # === 链接 ===
    url="https://github.com/you/my-package",
    project_urls={                        # PyPI 侧边栏链接
        "Documentation": "https://my-package.readthedocs.io",
        "Source": "https://github.com/you/my-package",
        "Tracker": "https://github.com/you/my-package/issues",
    },

    # === 分类（PyPI Trove Classifiers）===
    classifiers=[
        "Development Status :: 4 - Beta",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
        "Programming Language :: Python :: 3.12",
        "Operating System :: OS Independent",
    ],

    # === 许可证 ===
    license="MIT",

    # === 关键字（PyPI 搜索）===
    keywords="packaging setuptools demo",
)
```

### 2.2 包发现参数

```python
from setuptools import setup, find_packages

setup(
    # === 方式一: 手动声明（精确控制）===
    packages=["my_package", "my_package.utils", "my_package.plugins"],
    package_dir={"": "src"},              # 告诉 setuptools 源码在 src/ 下

    # === 方式二: find_packages() 自动发现（推荐）===
    packages=find_packages(where="src"),  # 自动扫描 src/ 下的所有 Python 包
    package_dir={"": "src"},

    # find_packages 高级过滤:
    # packages=find_packages(
    #     where="src",
    #     include=["my_package", "my_package.*"],   # 只包含
    #     exclude=["my_package.tests", "my_package._dev"],  # 排除
    # ),
)
```

### 2.3 依赖参数

```python
setup(
    # === 运行时依赖 ===
    install_requires=[
        "requests>=2.28,<3.0",            # 范围约束（推荐）
        "pyyaml>=6.0",
        "click>=8.0; sys_platform!='win32'",  # 条件依赖（PEP 508）
        "colorama; sys_platform=='win32'",
    ],

    # === Python 版本约束 ===
    python_requires=">=3.10",

    # === 可选依赖（extras）===
    extras_require={
        "cli": ["rich>=13.0", "typer>=0.15"],
        "dev": ["pytest>=8.0", "ruff>=0.8", "mypy>=1.14"],
        "all": ["my-package[cli,dev]"],   # 组合其他 extras
    },

    # === 测试依赖（已废弃，但仍广泛使用）===
    tests_require=["pytest>=8.0", "pytest-cov>=6.0"],

    # === setup.py 自身依赖 ===
    setup_requires=["setuptools>=61.0"],   # 运行 setup.py 需要的包
)
```

### 2.4 入口点参数

```python
setup(
    # === 命令行工具 ===
    entry_points={
        # 终端命令
        "console_scripts": [
            "mycmd=my_package.cli:main",
            "myadmin=my_package.admin:main",
        ],
        # GUI 应用（Windows 不弹命令行窗口）
        "gui_scripts": [
            "mygui=my_package.gui:run",
        ],
        # 插件入口（第三方工具发现你的插件）
        "pytest11": [
            "my_plugin=my_package.pytest_plugin",
        ],
        "myapp.commands": [
            "greet=my_package.commands.greet:GreetCommand",
            "report=my_package.commands.report:ReportCommand",
        ],
    },
)
```

```python
# 对应的 my_package/cli.py
def main():
    import sys
    print(f"Hello, {sys.argv[1] if len(sys.argv) > 1 else 'World'}!")
```

安装后直接运行：`mycmd Alice`

### 2.5 数据文件参数

```python
setup(
    # === 方式一: 按包声明（精确）===
    package_data={
        "my_package": ["templates/*.html", "static/*.css", "static/*.js"],
        "my_package.config": ["*.yaml", "*.json"],
    },

    # === 方式二: 全包匹配模式 ===
    package_data={
        "": ["*.txt", "*.rst", "*.md"],             # 所有包
        "my_package": ["data/*.csv"],
    },

    # === 方式三: include_package_data（配合 MANIFEST.in）===
    include_package_data=True,  # 按 MANIFEST.in 规则自动包含
    # 此时需要：
    #   MANIFEST.in:
    #     graft my_package/templates
    #     graft my_package/static
    #     include README.md LICENSE

    # === 非包数据文件（安装到系统目录）===
    data_files=[
        ("share/my_package", ["config.yaml"]),
        ("share/doc/my_package", ["README.md", "CHANGELOG.md"]),
    ],

    # === 排除文件 ===
    exclude_package_data={
        "my_package": [".DS_Store", "*.pyc"],
    },
)
```

### 2.6 C/C++ 扩展参数

这是 setuptools **不可替代**的核心能力：

```python
from setuptools import setup, Extension

setup(
    name="my-c-extension",
    ext_modules=[
        Extension(
            "my_package._math",                   # 模块名
            sources=[
                "src/_math.c",                     # C 源文件
                "src/matrix.c",
            ],
            include_dirs=["/usr/local/include"],   # 头文件路径
            library_dirs=["/usr/local/lib"],       # 库文件路径
            libraries=["gsl", "m"],                # 链接库
            extra_compile_args=["-O3", "-march=native"],  # 编译选项
            define_macros=[("DEBUG", "1")],        # 宏定义
        ),
    ],
)
```

```c
// src/_math.c
#include <Python.h>
#include <gsl/gsl_matrix.h>

static PyObject *my_add(PyObject *self, PyObject *args) {
    int a, b;
    if (!PyArg_ParseTuple(args, "ii", &a, &b)) return NULL;
    return PyLong_FromLong(a + b);
}

static PyMethodDef MyMethods[] = {
    {"add", my_add, METH_VARARGS, "Add two numbers"},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef mymodule = {
    PyModuleDef_HEAD_INIT, "_math", NULL, -1, MyMethods
};

PyMODINIT_FUNC PyInit__math(void) {
    return PyModule_Create(&mymodule);
}
```

```bash
pip install .
python -c "from my_package._math import add; print(add(1, 2))"
# 输出: 3
```

### 2.7 其他重要参数

```python
setup(
    # 命名空间包（多个发行版共享一个顶级包名）
    namespace_packages=["mycompany"],

    # zip_safe: 是否可以从 .egg 压缩包直接运行
    zip_safe=False,  # 有 C 扩展或需要访问文件的包设为 False

    # 在 PyPI 上隐藏
    obsoletes=["old-package-name"],

    # 自定义构建命令
    cmdclass={
        "build_ext": my_custom_build_ext,
    },

    # 下载 URL
    download_url="https://github.com/you/my-package/archive/v1.0.0.tar.gz",

    # 平台限制
    platforms=["Windows", "Linux"],
)
```

---

## 三、三种配置方式全面对比

setuptools 支持三种配置方式，从旧到新：

| 方式 | 文件 | 优缺点 | 现状 |
|------|------|--------|------|
| `setup.py` | Python 代码 | 灵活，但本质是"可执行脚本" | 存量项目主流，新项目**不推荐** |
| `setup.cfg` | INI 格式 | 声明式，比 setup.py 安全 | 过渡方案，逐步淘汰 |
| `pyproject.toml` | TOML 格式 | 声明式 + 标准化（PEP 621） | PyPA 力推，未来方向 |

### 3.1 三种方式的相同配置

**setup.py 方式**：

```python
from setuptools import setup, find_packages

setup(
    name="my-package",
    version="1.0.0",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    install_requires=["requests>=2.28"],
    entry_points={
        "console_scripts": ["mycmd=my_package.cli:main"],
    },
    include_package_data=True,
)
```

**setup.cfg 方式**（声明式，免去代码执行风险）：

```ini
[metadata]
name = my-package
version = 1.0.0

[options]
packages = find:
package_dir =
    = src
install_requires =
    requests>=2.28
include_package_data = True

[options.packages.find]
where = src

[options.entry_points]
console_scripts =
    mycmd = my_package.cli:main
```

```python
# setup.py 变成极简 stub
from setuptools import setup
setup()
```

**pyproject.toml 方式**（现代化、PEP 621 标准）：

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = ["requests>=2.28"]

[project.scripts]
mycmd = "my_package.cli:main"

[tool.setuptools.packages.find]
where = ["src"]

[tool.setuptools]
include-package-data = true
```

### 3.2 配置优先级

当同一个项目同时存在 `setup.py`、`setup.cfg`、`pyproject.toml` 时：

```
pyproject.toml > setup.cfg > setup.py 的硬编码值
```

后面覆盖前面。所以迁移时可以逐步替换。

---

## 四、包发现深入：find_packages() 是怎么工作的？

### 4.1 原理

```python
from setuptools import find_packages

# 等价于：
# 1. 遍历 where 目录下的所有子目录
# 2. 找到包含 __init__.py 的目录 → 认定为"包"
# 3. 根据 include/exclude 过滤
# 4. 返回包名列表

packages = find_packages(where="src")
# ['my_package', 'my_package.utils', 'my_package.plugins']
```

### 4.2 src-layout vs flat-layout

```bash
# src-layout（推荐）
my-project/
├── setup.py          # package_dir={"": "src"}
└── src/
    └── my_package/
        ├── __init__.py
        └── utils/
            ├── __init__.py
            └── helper.py

# flat-layout（简单项目）
my-project/
├── setup.py          # 不需要 package_dir
└── my_package/
    └── __init__.py
```

**为什么推荐 src-layout？**

```bash
# flat-layout 暗藏陷阱：
cd my-project
python
>>> import my_package          # 导入了当前目录的源码！
>>> my_package.__file__         # .../my-project/my_package/__init__.py
# 这掩盖了"安装不完整"的问题！

# src-layout 强制你测试"安装后的包"：
cd my-project
python
>>> import my_package
# ImportError! 必须先 pip install，确保包是完整可安装的
```

### 4.3 排查"包发现失败"

```bash
# 诊断脚本：看 setuptools 发现了哪些包
python -c "
from setuptools.discovery import ConfigDiscovery
import json
d = ConfigDiscovery('.', {'packages': {'find': {'where': ['src']}}})
print(d.find())
"
```

---

## 五、开发模式（Editable Install）

开发中最常用的功能——改代码立刻生效。

```bash
# 安装为"可编辑模式"
pip install -e .
# 或带开发依赖
pip install -e ".[dev]"
```

### 5.1 原理

```
site-packages/
└── my-package.egg-link   # 纯文本文件，指向你的项目目录
    → 内容: /home/you/projects/my-package

pip 遇到 import my_package 时：
  1. 查 site-packages
  2. 找到 .egg-link
  3. 跳转到你的源码目录
  4. 直接加载源码 → 修改即时生效
```

### 5.2 临时文件 .gitignore

```gitignore
# setup.py 产生的临时文件
build/
dist/
*.egg-info/
*.egg
__pycache__/
*.pyc
```

---

## 六、MANIFEST.in 与数据文件完整指南

### 6.1 MANIFEST.in 是什么？

`MANIFEST.in` 控制 **sdist（源码分发包）** 中包含哪些非 Python 文件。

> ⚠️ **重要区分**: `package_data` 控制 wheel 包，`MANIFEST.in` 控制 sdist 包。两者都要配置才能完整覆盖。

### 6.2 常用指令

```makefile
# MANIFEST.in

# 包含指定文件
include LICENSE
include README.md
include CHANGELOG.md

# 包含匹配模式的文件（当前目录）
include *.txt
include *.yml

# 递归包含目录下所有文件
recursive-include docs *
recursive-include tests *.py

# 包含指定目录下所有文件（只一层）
graft src/my_package/templates
graft src/my_package/static

# 排除模式
global-exclude __pycache__
global-exclude *.py[co]
global-exclude .DS_Store

# 排除整个目录
prune tests
prune .github
```

### 6.3 完整配置对照

```python
# setup.py
setup(
    include_package_data=True,     # 启用 MANIFEST.in 规则
    package_data={                 # 额外声明（与 MANIFEST.in 叠加）
        "my_package": ["py.typed"],  # PEP 561 类型标记
    },
)
```

```makefile
# MANIFEST.in
graft src/my_package/templates
graft src/my_package/static
include LICENSE
include README.md
```

### 6.4 验证打包内容

```bash
# 构建 sdist 并检查内容
python -m build --sdist
tar -tzf dist/*.tar.gz | sort

# 构建 wheel 并检查内容
python -m build --wheel
python -m zipfile -l dist/*.whl
```

---

## 七、入口点深入：entry_points 的三种用法

### 7.1 自动生成脚本

```python
setup(
    entry_points={
        "console_scripts": [
            "mycli=my_package.cli:main",
        ],
    }
)
```

安装后 setuptools 生成跨平台 wrapper：

```python
# 实际生成的脚本（简化版）
# Windows: Scripts\mycli.exe
# Linux/Mac: bin/mycli
import sys
from my_package.cli import main
sys.exit(main())
```

### 7.2 插件发现系统

```python
# 插件提供方
setup(
    name="my-pytest-plugin",
    entry_points={
        "pytest11": ["myplugin = myplugin.hooks"],
    },
)
```

```python
# 主应用方：加载所有插件
from importlib.metadata import entry_points

def load_all_plugins(group="myapp.plugins"):
    """发现并加载所有注册的插件"""
    plugins = {}
    for ep in entry_points(group=group):
        try:
            plugins[ep.name] = ep.load()
        except Exception as e:
            print(f"Failed to load plugin {ep.name}: {e}")
    return plugins
```

### 7.3 Flask / Click 的入口点模式

```python
# Flask 扩展用这种方式注册自己
setup(
    entry_points={
        "flask.commands": [
            "init-db=myapp.commands:init_db_command",
        ],
    }
)
```

---

## 八、实战案例：一个完整的 CLI 工具项目

### 8.1 项目结构

```bash
weather-cli/
├── pyproject.toml
├── setup.py
├── setup.cfg
├── MANIFEST.in
├── README.md
├── LICENSE
├── src/
│   └── weather_cli/
│       ├── __init__.py
│       ├── __version__.py
│       ├── cli.py
│       ├── api.py
│       ├── formatters.py
│       ├── config/
│       │   ├── __init__.py
│       │   └── settings.yaml
│       └── templates/
│           ├── report.html
│           └── summary.txt
├── tests/
│   ├── __init__.py
│   ├── test_cli.py
│   └── test_api.py
└── docs/
    └── index.md
```

### 8.2 setup.py

```python
"""Weather CLI - A command-line weather tool."""
from setuptools import setup, find_packages
import os

HERE = os.path.abspath(os.path.dirname(__file__))

with open(os.path.join(HERE, "README.md"), encoding="utf-8") as f:
    long_description = f.read()

# 从单独文件读取版本号（单一数据源）
about = {}
with open(os.path.join(HERE, "src", "weather_cli", "__version__.py")) as f:
    exec(f.read(), about)

setup(
    name="weather-cli",
    version=about["__version__"],
    description="A command-line weather tool",
    long_description=long_description,
    long_description_content_type="text/markdown",
    author="Your Name",
    author_email="you@example.com",
    url="https://github.com/you/weather-cli",
    project_urls={
        "Documentation": "https://weather-cli.readthedocs.io",
        "Source": "https://github.com/you/weather-cli",
        "Issues": "https://github.com/you/weather-cli/issues",
    },

    # 包发现
    package_dir={"": "src"},
    packages=find_packages(
        where="src",
        exclude=["tests", "tests.*", "docs"],
    ),

    # 版本约束
    python_requires=">=3.10",

    # 运行时依赖
    install_requires=[
        "requests>=2.28,<3.0",
        "click>=8.1,<9.0",
        "rich>=13.0",
        "pyyaml>=6.0",
    ],

    # 可选依赖
    extras_require={
        "dev": [
            "pytest>=8.0",
            "pytest-cov>=6.0",
            "ruff>=0.8",
            "mypy>=1.14",
        ],
        "docs": ["sphinx>=8.0", "furo>=2024"],
    },

    # 命令行入口
    entry_points={
        "console_scripts": [
            "weather=weather_cli.cli:main",
        ],
    },

    # 数据文件
    include_package_data=True,
    package_data={
        "weather_cli": ["config/*.yaml", "templates/*"],
    },

    # 分类
    classifiers=[
        "Development Status :: 4 - Beta",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
        "Programming Language :: Python :: 3.12",
    ],
    keywords="weather cli command-line",
    license="MIT",
)
```

### 8.3 setup.cfg（可选，用于 flake8 等不支持 pyproject.toml 的工具）

```ini
[flake8]
max-line-length = 100
exclude = .venv,build,dist,.git,__pycache__
```

### 8.4 pyproject.toml

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

### 8.5 MANIFEST.in

```makefile
graft src/weather_cli/config
graft src/weather_cli/templates
include README.md
include LICENSE
global-exclude __pycache__
global-exclude *.py[co]
prune tests
prune docs
```

### 8.6 __version__.py

```python
# src/weather_cli/__version__.py
__version__ = "0.1.0"
```

### 8.7 构建与发布

```bash
# 安装开发模式
pip install -e ".[dev]"

# 运行命令
weather --city Shanghai

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

## 九、迁移路线图：setup.py → pyproject.toml

### 9.1 三步迁移法

**第一步（当前可用）**: 添加 `pyproject.toml`，保留 `setup.py`

```toml
# pyproject.toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

**第二步（逐步过渡）**: 把配置从 `setup.py` 抽到 `setup.cfg`，`setup.py` 只留 `setup()`

```python
# setup.py — 极简 stub
from setuptools import setup
setup()
```

**第三步（最终目标）**: 全部迁移到 `pyproject.toml`，删除 `setup.py` 和 `setup.cfg`

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "weather-cli"
dynamic = ["version"]

[project.scripts]
weather = "weather_cli.cli:main"

[tool.setuptools.packages.find]
where = ["src"]
exclude = ["tests*"]

[tool.setuptools.dynamic]
version = {attr = "weather_cli.__version__.__version__"}

[tool.setuptools]
include-package-data = true
```

### 9.2 什么时候必须保留 setup.py？

即使全面迁移到 `pyproject.toml`，以下场景仍需 `setup.py`：

| 场景 | 原因 |
|------|------|
| C 扩展的自定义编译逻辑 | `ext_modules` 需要运行时计算 |
| 动态依赖（写脚本检测系统环境） | pyproject.toml 是**静态声明** |
| 老 pip 用户（< v21.1）的 editable install | 必须存在 `setup.py` |
| Cython 编译 | `cythonize()` 需要在 `setup()` 中调用 |

```python
# 必须保留 setup.py 的例子：动态 Cython 编译
from setuptools import setup, Extension
from Cython.Build import cythonize

extensions = [
    Extension("mypack._core", ["src/_core.pyx"]),
]

setup(
    ext_modules=cythonize(extensions, compiler_directives={"language_level": "3"}),
)
```

---

## 十、常见问题排错

### 10.1 "error: package directory 'X' does not exist"

```bash
# setup.py 中的 package_dir 和实际目录不匹配
# 确认: package_dir={"": "src"} + src/my_package/ 都存在
ls src/my_package/__init__.py
```

### 10.2 "ModuleNotFoundError after pip install"

```bash
# 可能包没有被包含进 wheel
python -m zipfile -l dist/*.whl | grep my_package
# 如果为空 → find_packages 没找到包
# 检查: where 参数路径是否正确
```

### 10.3 "version not found" (setuptools-scm)

```bash
# setuptools-scm 需要 git 仓库和至少一个 tag
git init && git add -A && git commit -m "init"
git tag v0.1.0
python -m build
```

### 10.4 Editable install 不生效

```bash
# 新版 pip（>=21.1）用 PEP 660
pip install -e . --force-reinstall --no-deps

# 如果是老 pip，必须保留 setup.py（哪怕是空的 setup()）
```

### 10.5 数据文件没被包含

```bash
# 确认 include_package_data = True
# 确认 MANIFEST.in 存在且规则正确
# 构建后检查
python -m build --sdist
tar -tzf dist/*.tar.gz | grep "yaml"
```

---

## 十一、什么时候用 setuptools？

| 场景 | 推荐 | 备选 |
|------|------|------|
| 包含 C/C++ 扩展 | **setuptools**（唯一选择） | 无 |
| 纯 Python 新项目 | hatchling | flit |
| 维护大量存量项目 | **setuptools** | — |
| 需要 Cython 编译 | **setuptools** | — |
| 只需最简单的打包 | flit | hatchling |

---

## 总结

### setup() 核心参数速查

| 参数 | 用途 | 示例 |
|------|------|------|
| `name` | PyPI 唯一名称 | `"my-package"` |
| `version` | 语义化版本 | `"1.0.0"` |
| `packages` | 要打包的模块 | `find_packages(where="src")` |
| `package_dir` | 源码根目录映射 | `{"": "src"}` |
| `install_requires` | 运行时依赖 | `["requests>=2.28"]` |
| `extras_require` | 可选功能依赖 | `{"dev": ["pytest"]}` |
| `entry_points` | CLI 入口 + 插件 | `{"console_scripts": [...]}` |
| `package_data` | 包内数据文件 | `{"mypkg": ["*.html"]}` |
| `include_package_data` | 按 MANIFEST.in 打包 | `True` |
| `ext_modules` | C 扩展模块 | `[Extension(...)]` |
| `python_requires` | Python 版本约束 | `">=3.10"` |

### 三种配置方式建议

- **新项目** → `pyproject.toml` 为主，必要时保留 `setup.py` 做动态逻辑
- **老项目维护** → 保持 `setup.py`，逐步向 `setup.cfg` 或 `pyproject.toml` 迁移
- **C 扩展项目** → `setup.py` + `pyproject.toml`（声明 build-system）

> 📋 **参考资料**:
> - [Setuptools Quickstart](https://setuptools.pypa.io/en/stable/userguide/quickstart.html)
> - [Setuptools User Guide](https://setuptools.pypa.io/en/stable/userguide/)
> - [Configuring setuptools using pyproject.toml](https://setuptools.pypa.io/en/stable/userguide/pyproject_config.html)
> - [Python Packaging User Guide](https://packaging.python.org/en/latest/guides/)
> - [PEP 517 — Build system independent format](https://peps.python.org/pep-0517/)
> - [PEP 621 — Project metadata in pyproject.toml](https://peps.python.org/pep-0621/)
