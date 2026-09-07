# Python FastAPI 个人编码规范 Skill

这是个人 Python / FastAPI 开发规范 Skill，遵循 Agent Skills 开放标准（`SKILL.md` 格式），可用于 Codex、Cursor、Claude Code 等支持该标准的工具。

## 内容

- uv + FastAPI 项目初始化
- uvicorn 版本规范（固定 0.21.0）
- Pydantic Settings 配置管理
- 数据库可选：默认 Tortoise ORM + Aerich + aiomysql，支持 SQLAlchemy 等其他 ORM
- 标准项目目录结构（含 `tests/`、`docs/`）
- dev / pre / prd 多环境变量文件
- Git / Docker 忽略规则
- 日志规范
- FastAPI lifespan
- Router / Service 分层（含 `summary` 与 docstring 注释规范）
- Pydantic Schema
- Python 编码风格（类型注解、命名、import、类与方法中文注释）
- 异步编程规范
- 异常处理与依赖注入
- pytest 测试规范
- README 与 API 文档编写规范
- 新项目初始化流程 / 已有项目修改规则

## 在 Codex 中使用

### 前提

Codex 遵循 Agent Skills 开放标准：一个 Skill 就是一个包含 `SKILL.md` 的目录，`SKILL.md` 的 YAML frontmatter 必须包含 `name` 和 `description`。本 Skill 已满足该格式，直接复制到 Codex 的技能目录即可。

### 方式一：用户级安装（推荐，跨项目复用）

把整个 `python-fastapi-skill` 文件夹复制到用户级技能目录 `$HOME/.agents/skills/`。

Windows（PowerShell，在本 README 所在目录的上一级执行）：

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "$env:USERPROFILE\.agents\skills\"
```

macOS / Linux：

```bash
mkdir -p ~/.agents/skills
cp -r ./python-fastapi-skill ~/.agents/skills/
```

复制后的结构应为：

```text
~/.agents/skills/
└── python-fastapi-skill/
    ├── SKILL.md
    └── README.md
```

### 方式二：仓库级安装（跟随项目，可团队共享）

复制到项目仓库内的 `.agents/skills/` 目录，并提交到 Git：

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "你的项目路径\.agents\skills\"
```

Codex 会从当前工作目录向上扫描至仓库根目录，加载沿途每个 `.agents/skills` 中的技能。

### 使用方式

1. **显式调用**：在 Codex CLI / IDE 中运行 `/skills` 选择本技能，或输入 `$` 加技能名提及，例如 `$python-fastapi-skill`（即 `SKILL.md` 中的 `name`）。
2. **隐式调用**：直接描述任务，例如"帮我初始化一个 FastAPI 项目"，当任务与 `description` 匹配时 Codex 会自动选用本技能。

### 验证是否生效

- 运行 `/skills`，查看列表中是否出现本技能。
- 新增或修改技能后若没有生效，重启 Codex 或开启新会话。

### 注意事项

1. **`name` 与目录名保持一致**：规范要求 `SKILL.md` 的 `name` 与所在目录名一致（当前均为 `python-fastapi-skill`）。如需重命名技能，`name` 与目录名必须同时修改。
2. **`SKILL.md` 不要带 UTF-8 BOM**：文件第一个字节必须是 `---`。Windows 上部分编辑器保存时会自动添加 BOM，会导致解析失败。
3. **避免重名**：用户级和仓库级不要放置同名 Skill，Codex 不会合并同名技能。
4. **目录位置**：官方推荐 `.agents/skills`；旧路径 `~/.codex/skills` 仍会被扫描（兼容保留），内置的 `$skill-installer` 安装的技能也在旧路径下，排查时两个位置都要看。
5. **临时禁用而不删除**：在 `~/.codex/config.toml` 中添加：

```toml
[[skills.config]]
path = "C:/Users/<用户名>/.agents/skills/python-fastapi-skill/SKILL.md"
enabled = false
```

## 在 Cursor 中使用

### 方式一：用户级安装（推荐，所有项目可用）

把整个 `python-fastapi-skill` 文件夹复制到 Cursor 用户级技能目录 `~/.cursor/skills/`。

Windows（PowerShell）：

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "$env:USERPROFILE\.cursor\skills\"
```

macOS / Linux：

```bash
mkdir -p ~/.cursor/skills
cp -r ./python-fastapi-skill ~/.cursor/skills/
```

复制后的结构应为：

```text
~/.cursor/skills/
└── python-fastapi-skill/
    ├── SKILL.md
    └── README.md
```

### 方式二：项目级安装（跟随项目，可团队共享）

复制到项目根目录的 `.cursor/skills/` 下，并提交到 Git：

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "你的项目路径\.cursor\skills\"
```

### 使用方式

Cursor 的 Agent 会在任务与 `description` 匹配时**自动读取并遵循**该 Skill，无需手动调用。例如直接对 Agent 说"帮我初始化一个 FastAPI 项目"，它就会按照 SKILL.md 中的规范执行。

### 验证是否生效

- 开启新的 Agent 会话，提出相关任务（如"新建一个 FastAPI 项目"），观察 Agent 是否遵循规范（使用 uv、创建标准目录结构、询问数据库等）。
- 新增或修改后若没有生效，重开会话或重启 Cursor。

## 其他工具

同一目录可直接用于其他支持 Agent Skills 标准的工具，放置到对应技能目录即可，例如 Claude Code：`~/.claude/skills/`。
