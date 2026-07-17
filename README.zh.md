# Hermes Math Template（中文说明）

[English](README.md) | [中文](README.zh.md)

一套可复现的 Hermes Agent 数学研究环境模板 —— 支持 Lean 4 形式化证明、
LaTeX 论文编译、隐私友好的网络搜索。

基于官方 [Hermes Agent](https://hermes-agent.nousresearch.com/docs/) 文档构建：
[配置指南](https://hermes-agent.nousresearch.com/docs/user-guide/configuration) ·
[技能系统](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) ·
[MCP 服务器](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp) ·
[消息网关](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/)

## 仓库结构

```
.
├── skills/                         # 11 个 agent skill（SKILL.md + references/）
│   ├── lean-4-workflow/
│   ├── lean-4-proof-writing/
│   ├── mathematical-writing/
│   ├── chinese-math-paper/
│   ├── paper-engineering/
│   ├── latex-workflow/
│   ├── web-search-strategy/
│   ├── agent-memory-architecture/
│   ├── container-python-environment/
│   ├── hermes-web-search-debugging/
│   └── development-methodologies/
├── config/
│   ├── config.yaml.template        # Hermes 配置模板
│   └── env.template                # API key 和环境变量
├── docker-compose.yml              # 容器编排
├── docker/
│   └── searxng/settings.yml        # 搜索引擎配置
├── infra/                          # 基础设施搭建指南
│   ├── python.md
│   ├── lean.md
│   ├── latex.md
│   └── search.md
├── README.md
├── README.zh.md                    # 你在这里
├── AGENTS.md                       # 给 AI agent 看的项目上下文
└── .gitignore
```

这个仓库包含搭建完整 Hermes Agent 环境所需的**配置文件、技能和基础设施文档**，
面向以下场景优化：

- **形式化数学** — Lean 4 + Mathlib 4（通过 MCP server）
- **论文写作** — Tectonic（轻量 LaTeX）+ Pandoc
- **学术搜索** — 自托管 SearXNG + ddgs 备用
- **代码与证明工程** — 化为 agent skill 的成熟工作流

克隆后按说明操作，即可获得一个能辅助形式化证明、论文编译、网络搜索和代码开发的
本地 AI agent。

## 环境要求

- Docker Engine 24+
- 6 GB 可用内存
- 模型 API key（DeepSeek / Anthropic / OpenAI / OpenRouter 均可）
- (可选) Windows 用户需要 WSL2（Lean 4 无法在 NTFS 挂载的 Windows 路径下编译）
- **端口检查：** 宿主机 `8085`（SearXNG）和 `9119`（Dashboard）必须空闲。
  如需更改，设置 `SEARXNG_PORT` 和 `HERMES_DASHBOARD_PORT` 环境变量。

## 文件部署方式

本仓库有两层独立结构，可以放在不同的位置：

```
~/.hermes/                        ← Hermes 运行时（固定位置）
├── config.yaml                   ← 来自 config/config.yaml.template
├── .env                          ← 来自 config/env.template
├── skills/                       ← 来自 skills/*
│   ├── lean-4-workflow/
│   └── ...
└── ...                           （sessions, memory, pip-packages — 自动创建）

~/hermes-math-template/           ← Docker 与 workspace（可放在任意位置）
├── docker-compose.yml
├── docker-data/                  ← SearXNG 配置（自动创建）
├── AGENTS.md                     ← Hermes 启动时自动加载
├── skills/ config/ infra/ ...    ← 源文件（复制到 ~/.hermes/）
└── （repo 根目录即 workspace；可通过 $WORKSPACE_DIR 覆盖）
```

- **`~/.hermes/`** — 固定位置。Hermes 从这里读取配置、skill 和运行时状态。
- **Docker 文件** — 可移动。把 `docker-compose.yml` 放在任何你喜欢的地方。repo 根目录就是 workspace；也可通过 `$WORKSPACE_DIR` 指向任何路径。

### 将 compose 文件移到其他目录

如果把 `docker-compose.yml` 移出 repo，需要同时复制以下文件到它旁边：

```
~/my-docker/                       ← compose 文件所在目录
├── docker-compose.yml
├── AGENTS.md                      ← 必需（Hermes 自动加载）
└── docker-data/
    └── searxng/config/
        └── settings.yml           ← 必需（SearXNG 配置）
```

然后在该目录下启动：
```bash
cd ~/my-docker
docker compose up -d
```

其他文件 —— `config.yaml`、`.env`、`skills/` —— 始终保留在 `~/.hermes/` 中，不受影响。

## 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/wu1chenghui/hermes-math-template.git
cd hermes-math-template

# 2. 创建目录
mkdir -p ~/.hermes
mkdir -p docker-data/searxng/config docker-data/searxng/cache

# 3. 配置 Hermes
cp config/config.yaml.template ~/.hermes/config.yaml
cp config/env.template ~/.hermes/.env
# 编辑 ~/.hermes/.env —— 填入 API key（至少 DEEPSEEK_API_KEY）
cp docker/searxng/settings.yml docker-data/searxng/config/

# 4. 安装 skill
cp -r skills/* ~/.hermes/skills/

# 5. 启动
# 容器执行 `sleep infinity` —— 不会自动启动 gateway。
# Hermes CLI 在第 7 步手动启动。
docker compose up -d

# 6. 安装 Python 包（仅首次）
docker compose exec hermes bash -c "
    uv pip install --target /opt/data/pip-packages ddgs scrapling playwright curl_cffi browserforge httpx
    python3 -m playwright install chromium
"

# 7. 进入 Hermes
# 注意：/opt/hermes/bin 不在容器默认 PATH 中，需使用绝对路径调用特权降级 shim
docker compose exec hermes /opt/hermes/bin/hermes
```

## 包含内容

### Skills

| Skill | 说明 |
|---|---|
| `lean-4-workflow` | Lean 4 项目搭建、编译、依赖管理 |
| `lean-4-proof-writing` | 证明策略、模式、调试 |
| `mathematical-writing` | 数学论文写作惯例 |
| `chinese-math-paper` | 中文数学论文格式（GB/T 7713.2, ctexart） |
| `paper-engineering` | Lean 形式化 → 期刊论文管线 |
| `latex-workflow` | Markdown → LaTeX → PDF 编译 |
| `web-search-strategy` | 高效网络搜索策略 |
| `agent-memory-architecture` | AI agent 持久记忆系统设计 |
| `container-python-environment` | Python 虚拟环境与依赖管理 |
| `hermes-web-search-debugging` | 搜索后端故障排查 |
| `development-methodologies` | 外部方法论参考（⚠️ 未经完全验证） |

> **注意：** 这些 skill 是在一个真实的 Lean 4 项目中开发的。
> skill 参考文件中的路径指向 `/opt/lean-home/lean-projects/e/`
> —— 请将 `e` 替换为你自己的 Lean 项目目录。

### 基础设施文档

| 文档 | 内容 |
|---|---|
| `infra/python.md` | Python 包持久化（ddgs, scrapling, playwright） |
| `infra/lean.md` | Lean 4 + Mathlib 安装，MCP server 配置 |
| `infra/latex.md` | Tectonic、Pandoc、GitHub CLI 安装 |
| `infra/search.md` | SearXNG 搭建、ddgs 备用、Playwright 浏览器 |

### 配置模板

| 文件 | 用途 |
|---|---|
| `config/config.yaml.template` | Hermes 配置（模型、工具、MCP server 等） |
| `config/env.template` | API key 和环境变量 |
| `docker-compose.yml` | 容器编排 |
| `docker/searxng/settings.yml` | 搜索引擎配置 |

### 不包含的内容

本仓库提供模板和 skill —— **不包含**运行时数据：
- 不含 API key 或密码（请在 `env.template` 中填入自己的）
- 不含会话历史（`state.db`、`sessions/`）
- 不含个人记忆或用户画像（`memories/`）
- 不含 Python 包或 Playwright 浏览器（Quick Start 第 6 步安装）

这里的一切都是源文件，需要复制到你的 `~/.hermes/` 中使用。

## 架构

```
┌─────────────────────────────────────────────────┐
│ Docker Compose                                  │
│                                                 │
│  ┌──────────────┐  ┌──────────┐  ┌──────────┐  │
│  │   Hermes     │  │ SearXNG  │  │  Valkey  │  │
│  │   Agent      │  │  :8080   │  │  (缓存)  │  │
│  │              │  │          │  │          │  │
│  │ config.yaml  │  └──────────┘  └──────────┘  │
│  │ skills/      │                               │
│  │ pip-pkgs/    │  ┌──────────────────────────┐ │
│  │ playwright/  │  │  Lean 4 + Mathlib        │ │
│  └──────────────┘  │  (named volume)          │ │
│        │           │  /opt/lean-home          │ │
│        │           └──────────────────────────┘ │
│        │                                        │
│  ~/.hermes ← bind mount                         │
│  lean-home ← named volume（WSL ext4）            │
└─────────────────────────────────────────────────┘
```

## 模型

默认使用 **DeepSeek**（`deepseek-v4-pro`），数学推理能力强且价格低。
在 `config.yaml` → `model.default` 修改模型。

## WSL 注意事项

Windows WSL2 用户：

1. **Lean 4 必须在 WSL 原生文件系统**（`/opt/lean-home/`），不能放在
   `/mnt/c/` 下。NTFS 挂载会导致 `lake build` 失败。
2. docker-compose.yml 使用 named volume（`lean-home`）解决此问题。
3. 确保 `/etc/wsl.conf` 中 `systemd=true`，否则 Docker 无法正常运行。

## 许可证

MIT — 自由使用、修改、分享。

---

*AI agent 使用：请参阅 [AGENTS.md](AGENTS.md) 获取机器可读的项目上下文。*
