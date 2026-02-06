# mu-repo 项目指南

## 项目概述

**mu-repo** (Multiple Repositories) 是一个用于处理多个 git 仓库的命令行工具。它提供了便捷的功能来管理和同步多个 git 仓库，包括批量克隆、分组管理、分支操作、差异比较等。

- **版本**: 1.9.0
- **作者**: Fabio Zadrozny
- **许可证**: GNU General Public License v3 (GPLv3)
- **主要语言**: Python 3.7+
- **项目状态**: 成熟稳定 (Mature)
- **官方网站**: http://fabioz.github.io/mu-repo/

## 核心功能

- **批量克隆**: 从基础 URL 克隆多个仓库
- **仓库分组**: 创建和管理仓库组
- **差异比较**: 使用 WinMerge 或 meld 比较更改
- **分支操作**: 通过部分名称匹配检出分支
- **预览更改**: 预览当前分支的传入更改
- **快捷命令**: 常用 git 操作的快捷方式
- **批量执行**: 在所有注册的仓库上运行任意命令
- **Pull Request**: 在浏览器中打开创建 PR 的页面

## 项目结构

```
mu-repo/
├── mu.py                      # 主入口脚本
├── mu_repo/                   # 核心源代码目录
│   ├── __init__.py           # 主入口点和命令路由逻辑
│   ├── config.py             # 配置管理 (.mu_repo 文件)
│   ├── __docs__.py           # 命令文档
│   ├── action_*.py           # 各个命令的实现模块
│   │   ├── action_clone.py
│   │   ├── action_register.py
│   │   ├── action_list.py
│   │   ├── action_diff.py
│   │   ├── action_checkout.py
│   │   ├── action_sync.py
│   │   ├── action_add_commit_push.py
│   │   └── ...
│   ├── execute_*.py          # 命令执行相关模块
│   ├── tests/                # 测试文件
│   └── stat_server/          # 统计服务器
├── setup.py                   # Python 包配置
├── Dockerfile                 # Docker 镜像配置
├── docker-compose.yaml        # Docker Compose 配置
├── tox.ini                    # 测试配置
├── .travis.yml               # Travis CI 配置
└── README.rst                # 项目文档
```

## 配置文件

项目使用 `.mu_repo` 配置文件来管理仓库设置，支持以下配置项：

```
repo=<repository_path>        # 注册仓库路径
serial=0|1                   # 是否串行执行命令（默认并行）
git=/path/to/git             # Git 可执行文件路径
current_group=<group_name>   # 当前使用的组
group=<name>,<repo1>,<repo2> # 定义仓库组
```

## 构建和运行

### 安装

```bash
# 从源码安装
pip install -e .

# 从 PyPI 安装
pip install mu-repo
```

### 运行测试

```bash
# 使用 pytest
pytest mu_repo

# 使用 tox
tox

# 使用 pytest 并指定目录
pytest mu_repo/tests/
```

### Docker 使用

```bash
# 构建镜像
docker build -t mu-repo .

# 使用 docker-compose
docker-compose up
```

## 主要命令

### 仓库管理

```bash
mu register <repo1> <repo2>              # 注册仓库
mu register --all                         # 注册所有子目录（非递归）
mu unregister <repo1> <repo2>             # 取消注册
mu list                                   # 列出已注册的仓库
mu group add <name>                       # 创建新组
mu group switch <name>                    # 切换到指定组
mu group reset                            # 重置组（使用所有仓库）
```

### Git 操作快捷键

```bash
mu st                                      # 显示所有仓库状态（并行）
mu co <branch>                             # 检出分支
mu up                                      # 获取当前分支
mu up --all                                # 获取所有分支
mu sync / upd                              # 更新并显示传入更改
mu a                                       # git add -A
mu c <msg>                                 # git commit
mu ac <msg>                                # git add + commit
mu acp <msg>                               # git add + commit + push
mu p                                       # git push
mu rb                                      # rebase origin/current_branch
```

### 其他命令

```bash
mu dd                                      # 创建差异目录并打开 WinMerge
mu find-branch <pattern>                   # 查找匹配的分支
mu clone <base_url>                        # 从基础 URL 克隆多个仓库
mu sh <command>                            # 在所有仓库上执行命令
mu open-url                                # 在浏览器中打开 PR 页面
mu fix-eol                                 # 修复行尾符
mu set-var git=/path/to/git                # 设置 git 路径
mu get-vars                                # 打印配置
mu auto-update                             # 自动更新 mu-repo
```

## 开发约定

### 代码组织

- **命令实现**: 每个命令对应一个独立的 `action_*.py` 模块
- **命令路由**: 主入口点在 `mu_repo/__init__.py` 的 `main()` 函数中
- **配置管理**: 所有配置相关代码在 `config.py` 中
- **并行执行**: 默认并行执行命令，可通过 `serial=1` 改为串行

### 命令模式

- 所有命令函数都接受 `Params` 对象作为参数
- `Params` 包含：`config`、`args`、`config_file`
- 命令应返回 `Status` 对象或 `None`

### 测试

- 使用 pytest 作为测试框架
- 测试文件位于 `mu_repo/tests/` 目录
- 每个主要功能模块都有对应的测试文件

### 版本管理

- 版本号在 `setup.py` 和 `mu_repo/__init__.py` 中同步维护
- 发布时创建 tag 并推送，部署会自动触发

### 环境变量

- `MU_REPO_SERIAL`: 覆盖配置文件中的 `serial` 设置

## 开发环境设置

```bash
# 克隆仓库
git clone https://github.com/jieraltjp/mu-repo.git
cd mu-repo

# 创建虚拟环境（推荐）
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/Mac

# 安装开发依赖
pip install -e .
pip install pytest

# 运行测试
pytest
```

## 注意事项

- Windows 系统需要使用 `git.exe` 路径设置
- 项目支持 Python 3.7+
- 默认并行执行命令以提高效率
- `.mu_repo` 配置文件在当前目录或父目录中向上搜索
- 命令支持使用 `repo:` 或 `@` 前缀指定特定仓库
- 支持使用 `--timeit` 参数测量命令执行时间