# xiaoye-skills

个人 Agent Skills 集合，遵循 Agent Skills 开放标准（`SKILL.md` 格式），可用于 Codex、Cursor、Claude Code 等支持该标准的工具。

每个 Skill 目录只需包含 `SKILL.md`，Agent 会根据其中的 `name` 和 `description` 自动加载。

## 目录

```text
xiaoye-skills/
├── python-fastapi-skill/
│   └── SKILL.md
└── vue-coding-skill/
    └── SKILL.md
```

## Skill 列表

| Skill | 说明 |
|-------|------|
| `python-fastapi-skill` | Python / FastAPI 后端项目初始化与编码规范 |
| `vue-coding-skill` | Vue 3 前端项目初始化与编码规范 |

两个 Skill 的多环境命名一致，适合前后端分离项目一起使用。

## 安装

### 前提

一个 Skill 就是一个包含 `SKILL.md` 的目录，`SKILL.md` 的 YAML frontmatter 必须包含 `name` 和 `description`。

### Codex（用户级，推荐）

Windows（PowerShell，在本仓库根目录执行）：

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "$env:USERPROFILE\.agents\skills\"
Copy-Item -Recurse -Force ".\vue-coding-skill" "$env:USERPROFILE\.agents\skills\"
```

macOS / Linux：

```bash
mkdir -p ~/.agents/skills
cp -r ./python-fastapi-skill ~/.agents/skills/
cp -r ./vue-coding-skill ~/.agents/skills/
```

### Cursor（用户级，推荐）

Windows（PowerShell）：

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "$env:USERPROFILE\.cursor\skills\"
Copy-Item -Recurse -Force ".\vue-coding-skill" "$env:USERPROFILE\.cursor\skills\"
```

macOS / Linux：

```bash
mkdir -p ~/.cursor/skills
cp -r ./python-fastapi-skill ~/.cursor/skills/
cp -r ./vue-coding-skill ~/.cursor/skills/
```

### 项目级安装（跟随项目，可团队共享）

复制到目标项目对应目录下：

- Codex：`.agents/skills/`
- Cursor：`.cursor/skills/`

```powershell
Copy-Item -Recurse -Force ".\python-fastapi-skill" "你的项目路径\.cursor\skills\"
Copy-Item -Recurse -Force ".\vue-coding-skill" "你的项目路径\.cursor\skills\"
```

## 使用方式

### 显式调用

- Codex：运行 `/skills` 选择，或用 `$python-fastapi-skill`、`$vue-coding-skill` 提及
- Cursor：Agent 会在任务与 `description` 匹配时自动读取并遵循

### 隐式调用

直接描述任务即可，例如：

- "帮我初始化一个 FastAPI 项目"
- "帮我初始化一个 Vue 3 项目"

## 验证是否生效

- Codex：运行 `/skills`，查看列表中是否出现对应 Skill
- Cursor：开启新 Agent 会话，观察 Agent 是否遵循规范
- 新增或修改后若没有生效，重启工具或开启新会话

## 注意事项

1. **`name` 与目录名保持一致**：`SKILL.md` 的 `name` 与所在目录名必须一致。重命名时需同时修改。
2. **`SKILL.md` 不要带 UTF-8 BOM**：文件第一个字节必须是 `---`。
3. **避免重名**：用户级和项目级不要放置同名 Skill。
4. **Codex 目录位置**：官方推荐 `.agents/skills`；旧路径 `~/.codex/skills` 仍兼容。
5. **临时禁用**：在 `~/.codex/config.toml` 中配置 `enabled = false`。

## 其他工具

同一目录结构可用于其他支持 Agent Skills 标准的工具，例如 Claude Code：`~/.claude/skills/`。
