---
name: python-fastapi-skill
description: 个人 Python/FastAPI 项目初始化与编码规范。创建、初始化、重构或扩展 Python FastAPI 后端项目时使用，遵循用户偏好的项目结构、依赖、配置、日志、异步编程、API 分层、数据库、测试、文档、Docker 和编码风格。
---

# Python FastAPI 个人编码规范

本 Skill 定义了用户在 Python FastAPI 后端项目中的偏好规范。

## 1. 核心原则

- 使用 `uv` 进行 Python 依赖与项目管理。
- 使用 FastAPI 构建 HTTP API。
- I/O 密集型操作优先使用异步编程。
- 始终使用类型注解。
- 保持配置、日志、应用启动、业务逻辑与持久化等关注点相互分离。
- 优先编写简单、明确的代码，避免不必要的抽象。
- 修改已有项目时遵循现有项目风格，不要重写无关代码。

## 2. 新项目初始化

创建新的 FastAPI 项目时，使用 `uv` 初始化依赖。

除明确指定版本的依赖外，其余依赖安装最新兼容版本。

```bash
uv add fastapi
uv add uvicorn==0.21.0
uv add pydantic-settings
```

注意：
- `uvicorn` 必须固定使用 `0.21.0`。
- 除非用户明确要求或存在兼容性要求，否则不要随意固定其他依赖版本。

### 数据库依赖

数据库依赖不是默认安装的。

安装数据库依赖之前，必须先询问项目是否需要数据库。

如果需要数据库，还必须询问用户使用哪种 ORM。默认推荐 Tortoise ORM，也可以选择 SQLAlchemy、SQLModel 等其他 ORM，以用户的选择为准。

如果选择 Tortoise ORM，则安装：

```bash
uv add "tortoise-orm[accel]"
uv add aerich
uv add aiomysql
```

如果选择其他 ORM，则安装其对应的 ORM 依赖、迁移工具和异步数据库驱动。

数据库默认规范（用户未指定其他 ORM 时）：
- ORM：Tortoise ORM
- 数据库迁移：Aerich
- MySQL 驱动：aiomysql
- 优先使用异步数据库访问。

## 3. 标准项目结构

新项目创建以下标准结构：

```text
project/
├── .env.dev
├── .env.pre
├── .env.prd
├── .gitignore
├── .dockerignore
├── Dockerfile
├── pyproject.toml
├── uv.lock
├── README.md
├── docs/
├── tests/
└── src/
    ├── config.py
    ├── logger.py
    └── app/
        └── main.py
```

如果项目规模较大，再根据实际需要扩展 `src/app/`：

```text
src/
├── config.py
├── logger.py
└── app/
    ├── main.py
    ├── routers/
    ├── schemas/
    ├── services/
    ├── models/
    └── dependencies/
```

没有实际需求时，不要提前创建无意义的目录。

测试代码统一放在项目根目录的 `tests/` 目录中，不要放进 `src/`。

## 4. 环境变量文件

在项目根目录创建以下文件：

```text
.env.dev
.env.pre
.env.prd
```

分别表示：

- `.env.dev`：开发环境
- `.env.pre`：预发布环境
- `.env.prd`：生产环境

包含密钥、密码等敏感信息的环境变量文件不得提交到 Git。

示例：

```env
APP_NAME=FastAPI Application
APP_ENV=dev

HOST=127.0.0.1
PORT=8000
DEBUG=true

DATABASE_URL=mysql://user:password@127.0.0.1:3306/example
```

## 5. Git 配置

创建 `.gitignore`：

```gitignore
.idea
.venv

*.pyc
__pycache__/

migrations
```

修改已有项目时，除非用户明确要求，否则保留现有 `.gitignore`。

## 6. Docker 配置

初始化项目时先创建一个空的 `Dockerfile`。

创建 `.dockerignore`：

```dockerignore
.git
.claude
__pycache__
*.py[cod]
*.pyo
.Python
.env
.env.*
.venv
venv
build
dist
*.egg-info
.pytest_cache
.mypy_cache
.ruff_cache
.DS_Store
tests
```

不要将 `.env` 文件复制进 Docker 镜像。

## 7. 配置管理

使用 Pydantic Settings 创建 `src/config.py`。

推荐的基础实现：

```python
from functools import lru_cache
from pathlib import Path

from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict


BASE_DIR = Path(__file__).resolve().parent.parent


class Settings(BaseSettings):
    app_name: str = Field(default="FastAPI Application")
    app_env: str = Field(default="dev")

    host: str = Field(default="127.0.0.1")
    port: int = Field(default=8000)
    debug: bool = Field(default=True)

    model_config = SettingsConfigDict(
        env_file=BASE_DIR / ".env.dev",
        env_file_encoding="utf-8",
        extra="ignore",
    )


@lru_cache
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

规范：
- 项目配置统一通过 `settings` 访问。
- 当默认值或字段元信息有助于提高可读性时使用 `Field`。
- 不要在业务代码中到处调用 `os.getenv()`。
- 密钥和环境相关配置放在环境变量文件中。
- 实现多环境加载时，根据当前环境选择 `.env.dev`、`.env.pre` 或 `.env.prd`，不要把某个环境文件硬编码到底。

## 8. 日志规范

创建 `src/logger.py`。

推荐实现：

```python
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path


BASE_DIR = Path(__file__).resolve().parent.parent
LOG_DIR = BASE_DIR / "logs"

LOG_DIR.mkdir(parents=True, exist_ok=True)


def setup_logger() -> logging.Logger:
    logger = logging.getLogger("app")
    logger.setLevel(logging.INFO)

    if logger.handlers:
        return logger

    formatter = logging.Formatter(
        fmt="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

    console_handler = logging.StreamHandler()
    console_handler.setFormatter(formatter)

    file_handler = RotatingFileHandler(
        LOG_DIR / "app.log",
        maxBytes=10 * 1024 * 1024,
        backupCount=5,
        encoding="utf-8",
    )
    file_handler.setFormatter(formatter)

    logger.addHandler(console_handler)
    logger.addHandler(file_handler)

    return logger


logger = setup_logger()
```

日志规范：
- 使用统一的应用 Logger，不要每个模块单独配置日志。
- 记录重要的启动、关闭事件以及有意义的异常。
- 需要保留异常堆栈时使用 `logger.exception()`。
- 不要记录密码、API Key、Token 等敏感信息。
- 使用滚动日志文件，避免日志无限增长。

## 9. FastAPI 应用

创建 `src/app/main.py`。

推荐的基础实现：

```python
from contextlib import asynccontextmanager

import uvicorn
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware


@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动逻辑
    try:
        yield
    finally:
        # 关闭逻辑
        pass


app = FastAPI(
    title="测试",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)


if __name__ == "__main__":
    uvicorn.run(
        "src.app.main:app",
        host="127.0.0.1",
        port=8000,
        reload=True,
    )
```

规范：
- 使用 FastAPI `lifespan` 管理应用启动和关闭资源。
- `main.py` 只负责应用组装，不堆积业务逻辑。
- Router、业务逻辑、数据库 Model 和复杂逻辑不要直接写在 `main.py` 中。
- 本地开发启动可以使用 `reload=True`。
- 生产环境启动配置应与本地开发配置分离。

## 10. API 分层

对于非简单 API，优先采用：

```text
Router
  ↓
Service
  ↓
Database
```

各层职责：

### Router
- 解析 HTTP 请求输入。
- 使用 Pydantic Schema 校验请求数据。
- 调用 Service。
- 返回 Response Model。
- 不在 Router 中堆积业务逻辑。
- 路由装饰器必须添加 `summary`，简要说明当前接口的作用。
- 路由函数必须编写文档字符串（docstring），说明函数作用、请求参数含义和返回值含义。

示例：

```python
from fastapi import APIRouter, Body, HTTPException, status

from src.app.schemas.user import UserCreate, UserResponse
from src.app.services.user_service import create_user
from src.logger import logger

router = APIRouter(prefix="/users", tags=["用户管理"])


@router.post(path="/add-user", summary="创建用户", response_model=UserResponse)
async def add_user(data: UserCreate = Body(description="创建用户请求体")):
    """创建用户。

    接收用户注册信息，调用 Service 层完成用户创建。

    Args:
        data: 创建用户请求体，包含 username（用户名）和 email（邮箱）。

    Returns:
        UserResponse: 创建成功的用户信息，包含 id、username 和 email。
    """
    try:
        return await create_user(data.model_dump())
    except Exception:
        logger.exception("创建用户失败")
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="用户创建失败",
        )
```

以上示例仅用于演示格式规范，具体业务命名以实际需求为准。

### Service
- 实现业务规则。
- 协调多个 Model 操作或外部服务。
- 在合理情况下避免 Service 与 HTTP 细节强耦合。

## 11. Pydantic Schema

请求和响应结构统一使用 Pydantic Model。

优先使用明确的 Model：

```python
from pydantic import BaseModel


class UserCreate(BaseModel):
    username: str
    email: str


class UserResponse(BaseModel):
    id: int
    username: str
    email: str
```

如果类型化 Model 能让接口契约更清晰，就不要让原始字典在多个业务层之间传递。

## 12. Python 编码风格

通用规范：

- 使用 Python 类型注解。
- 优先使用清晰完整的名称，避免不必要的缩写。
- 函数和变量使用 `snake_case`。
- 类使用 `PascalCase`。
- 常量使用 `UPPER_SNAKE_CASE`。
- 函数保持小而专一。
- 避免过深的条件嵌套。
- 避免不必要的全局状态。
- 能让流程更清晰时，优先使用提前返回。
- 字符串插值使用 f-string。
- 项目代码统一优先使用双引号。
- import 按以下顺序分组：
  1. Python 标准库
  2. 第三方依赖
  3. 项目内部模块
- 不使用 `import *`。
- 定义类时，必须为类本身和它的每个方法编写中文文档字符串（docstring），说明类/方法的作用、请求参数含义和返回值含义。
- 行内注释用于说明意图或不明显的设计决策，不要给显而易见的代码写废话注释。

示例：

```python
from pathlib import Path

from fastapi import APIRouter

from src.app.services.user_service import UserService
```

类注释示例：

```python
class UserService:
    """用户服务类，封装用户相关的业务逻辑。"""

    async def create_user(self, data: dict) -> User:
        """创建用户。

        Args:
            data: 用户信息，包含 username（用户名）和 email（邮箱）。

        Returns:
            User: 创建成功的用户 Model 实例。
        """
        return await User.create(**data)
```

以上示例仅用于演示格式规范，具体业务命名以实际需求为准。

## 13. 异步编程规范

FastAPI 中涉及 I/O 的操作优先使用异步 API。

推荐写法：

```python
async def get_user(user_id: int):
    return await User.get(id=user_id)
```

不要在 async 函数中执行阻塞操作。

规范：
- 数据库 I/O 使用异步方式。
- HTTP 请求使用异步客户端。
- 文件和网络操作不应阻塞事件循环。
- 已经处于异步上下文时，不要调用 `asyncio.run()`。
- 不要 `await` 异步生成器，应使用 `async for` 消费。
- CPU 密集型同步任务可能阻塞请求时，应避免直接运行在事件循环中。

## 14. 数据库与 ORM

启用数据库支持后，无论使用哪种 ORM，都遵循以下通用规范：

- 数据库配置集中管理。
- Model 定义放在 `app/models/`。
- 优先使用异步 ORM 操作。
- 需要原子性的操作使用事务。
- 数据库迁移与正常应用启动逻辑分离。
- 使用所选 ORM 配套的迁移工具管理数据库结构迁移。

不要只手动修改生产数据库而忽略 Model 与迁移历史的一致性。

### Tortoise ORM（默认推荐）

用户未指定其他 ORM 时，默认使用 Tortoise ORM：

- 使用 Tortoise ORM Model。
- 使用 Aerich 管理数据库结构迁移。

典型迁移流程：

```bash
aerich migrate
aerich upgrade
```

### 其他 ORM

如果用户选择 SQLAlchemy、SQLModel 等其他 ORM，安装对应依赖并遵循该 ORM 的官方最佳实践，同时遵守上述通用规范。

## 15. 异常处理规范

规范：
- 不要静默吞掉异常。
- 抛出有意义的异常。
- 在 HTTP 边界层适当使用 FastAPI `HTTPException`。
- 不要向 API 客户端暴露内部堆栈、凭据、SQL 细节或基础设施信息。
- 对未预期异常记录带堆栈信息的日志。

示例：

```python
from fastapi import HTTPException, status


if user is None:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="用户不存在",
    )
```

## 16. 依赖注入

对于可复用的请求级或应用级依赖，使用 FastAPI Dependency。

例如：
- 当前登录用户
- 数据库 / Session 依赖
- Service 构建
- 请求上下文

不要把无关的全局状态塞进 Dependency 函数。

## 17. 测试规范

使用 pytest 编写测试，测试代码统一放在项目根目录的 `tests/` 目录中。

安装测试依赖（开发依赖）：

```bash
uv add --dev pytest pytest-asyncio httpx
```

在 `pyproject.toml` 中配置 pytest：

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

规范：
- 测试文件以 `test_` 开头命名，测试函数以 `test_` 开头命名。
- 异步接口使用 pytest-asyncio 测试，配置 `asyncio_mode = "auto"` 后无需手动添加标记。
- API 测试使用 httpx `AsyncClient` + `ASGITransport`，直接针对 FastAPI 应用测试，不依赖真实启动的服务。
- 公共测试夹具（fixture）统一放在 `tests/conftest.py` 中。
- 测试函数使用中文 docstring 说明测试意图。
- 优先覆盖核心业务逻辑和 API 契约，不为追求覆盖率给简单代码写无意义测试。
- 测试不得依赖生产环境配置和真实外部服务；需要数据库时使用独立的测试数据库。

示例：

```python
from httpx import ASGITransport, AsyncClient

from src.app.main import app


async def test_add_user():
    """测试创建用户接口：传入合法请求体时应返回 200。"""
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test",
    ) as client:
        response = await client.post(
            "/users/add-user",
            json={"username": "test", "email": "test@example.com"},
        )

    assert response.status_code == 200
```

运行测试：

```bash
uv run pytest
```

## 18. 文档规范

### README.md

新项目必须在项目根目录创建 `README.md`，并按以下顺序组织内容：

1. **项目背景**：说明项目是什么、解决什么问题、有哪些主要功能。
2. **目录结构**：介绍项目的目录文件结构及各目录的职责。
3. **依赖安装与运行**：Python 版本要求、使用 uv 安装依赖、环境变量配置、启动命令、测试命令。
4. **API 概览**：如果项目提供 API，用表格简要列出接口（方法、路径、作用），并链接到详细文档。

推荐结构：

````markdown
# 项目名称

## 项目背景

简要说明项目的背景、目标和主要功能。

## 目录结构

```text
project/
├── ...
└── ...
```

## 快速开始

### 环境要求

- Python 3.12+
- uv

### 安装依赖

```bash
uv sync
```

### 配置环境变量

根据当前环境准备 `.env.dev` 等环境变量文件。

### 启动项目

```bash
uv run python -m src.app.main
```

### 运行测试

```bash
uv run pytest
```

## API 概览

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/users/add-user` | 创建用户 |

详细接口说明见 [docs/API说明文档.md](docs/API说明文档.md)。
````

### API 说明文档

如果项目提供 API，在 `docs/` 目录下创建 `API说明文档.md`，逐个接口详细说明：

- 接口方法、路径和作用。
- 请求参数（路径参数、查询参数、请求体）及含义。
- 响应结构及字段含义。
- 错误情况和状态码说明。

单个接口的推荐格式：

````markdown
## 创建用户

- **方法**：POST
- **路径**：`/users/add-user`
- **说明**：创建新用户

### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名 |
| email | string | 是 | 邮箱 |

### 响应

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 用户 ID |
| username | string | 用户名 |
| email | string | 邮箱 |
````

规范：
- 文档统一使用中文编写。
- README 保持简洁，详细内容放到 `docs/` 目录。
- 接口变更时，必须同步更新 README 的 API 概览和 `docs/API说明文档.md`。
- API 文档中的接口说明应与 Router 函数的 `summary` 和 docstring 保持一致。

## 19. 开发流程

新项目按照以下流程：

1. 初始化 uv 项目。
2. 安装基础依赖与测试依赖（pytest、pytest-asyncio、httpx）。
3. 询问是否需要数据库。
4. 如果需要数据库，询问使用哪种 ORM（默认推荐 Tortoise ORM，也可选择其他 ORM）。
5. 根据选择的 ORM 安装数据库依赖。
6. 创建标准目录结构。
7. 创建环境变量文件。
8. 创建 `.gitignore`。
9. 创建 `.dockerignore`。
10. 创建 `Dockerfile`。
11. 创建 `config.py`。
12. 创建 `logger.py`。
13. 创建 `app/main.py`。
14. 仅在实际需要时添加 Router、Service、Model 等目录和文件。
15. 创建 `README.md`；如有 API，在 `docs/` 下创建 `API说明文档.md`。
16. 验证应用能够正常启动，`uv run pytest` 可以正常运行。
17. 如果使用数据库，配置并验证数据库迁移。

## 20. 修改已有项目

不要在已有项目中盲目重建整个目录结构。

首先检查：
- `pyproject.toml`
- 已有的 `src/` 目录
- 已有配置
- 已有日志配置
- 已有 Docker 文件
- 已有数据库迁移配置

然后结合现有代码结构，在新增或大幅修改的代码中应用本规范。

除非用户明确要求重构，否则不要破坏现有可用功能。

## 21. 决策规则

当需求存在歧义时：

- 是否需要数据库？先询问。
- 使用哪种 ORM？需要数据库时先询问，默认推荐 Tortoise ORM。
- 小型项目？避免不必要的 Service 分层。
- 已有项目？除非要求统一规范，否则保留已有结构。
- 新项目？使用本 Skill 定义的标准结构。
- 环境相关配置？不要硬编码生产环境密钥。
- 外部 I/O？优先使用异步。
- Schema / API 契约？优先使用 Pydantic Model。
- 启动 / 关闭资源？使用 FastAPI lifespan。
