# Forge 模块文档

> 📍 **位置**: [skyvern](../CLAUDE.md) → forge

## 模块概述

Forge 模块是 Skyvern 的 API 服务器层，基于 FastAPI 构建，提供完整的 REST API 和 WebSocket 支持。它作为系统的核心通信枢纽，处理所有外部请求、内部服务协调和实时通信。

## 文件结构

```
skyvern/forge/
├── __init__.py
├── api_app.py              # FastAPI 应用主入口
├── app.py                  # 应用配置
├── dependencies.py         # 依赖注入
├── middleware.py           # 中间件
├── exceptions.py           # 异常处理器
├── main.py                 # 应用启动入口
├── api/                    # API 路由定义
│   ├── __init__.py
│   ├── v1/                 # API v1 版本
│   │   ├── __init__.py
│   │   ├── router.py       # 主路由
│   │   ├── agent_protocol.py    # 代理协议端点
│   │   ├── artifacts.py         # 工件管理端点
│   │   ├── browser_sessions.py  # 浏览器会话端点
│   │   ├── credentials.py       # 凭据管理端点
│   │   ├── data_sources.py      # 数据源端点
│   │   ├── organizations.py     # 组织管理端点
│   │   ├── proxy.py            # 代理管理端点
│   │   ├── runs.py             # 执行记录端点
│   │   ├── server.py           # 服务器信息端点
│   │   ├── tasks.py            # 任务管理端点
│   │   ├── users.py            # 用户管理端点
│   │   ├── webhook.py          # Webhook 端点
│   │   ├── workflows.py        # 工作流端点
│   │   └── workflows_v2.py     # 工作流 v2 端点
│   └── public/             # 公开 API
│       ├── __init__.py
│       └── health.py         # 健康检查端点
├── sdk/                    # SDK 组件
│   ├── __init__.py
│   ├── api/                # API 客户端
│   │   ├── __init__.py
│   │   ├── llm/            # LLM 集成
│   │   │   ├── __init__.py
│   │   │   ├── base.py     # LLM 基础接口
│   │   │   ├── openai.py   # OpenAI 集成
│   │   │   ├── anthropic.py # Anthropic 集成
│   │   │   ├── azure.py    # Azure OpenAI 集成
│   │   │   ├── aws.py      # AWS Bedrock 集成
│   │   │   ├── gemini.py   # Gemini 集成
│   │   │   └── ollama.py   # Ollama 集成
│   │   └── routes/         # 路由访问器
│   │       ├── __init__.py
│   │       ├── agent_protocol.py
│   │       ├── artifacts.py
│   │       ├── browser_sessions.py
│   │       ├── credentials.py
│   │       ├── data_sources.py
│   │       ├── organizations.py
│   │       ├── proxy.py
│   │       ├── runs.py
│   │       ├── server.py
│   │       ├── tasks.py
│   │       ├── users.py
│   │       ├── webhook.py
│   │       ├── workflows.py
│   │       └── workflows_v2.py
│   ├── artifact/           # 工件管理
│   │   ├── __init__.py
│   │   ├── manager.py      # 工件管理器
│   │   └── storage.py      # 存储后端
│   ├── cache/              # 缓存系统
│   │   ├── __init__.py
│   │   ├── redis_client.py # Redis 客户端
│   │   └── cache_manager.py # 缓存管理器
│   ├── cryptocurrency/     # 加密货币支持
│   │   ├── __init__.py
│   │   └── client.py       # 加密货币客户端
│   ├── db/                 # 数据库访问
│   │   ├── __init__.py
│   │   ├── engine.py       # 数据库引擎
│   │   ├── models.py       # ORM 模型
│   │   ├── crud.py         # CRUD 操作
│   │   └── migrations.py   # 迁移工具
│   ├── encryption/         # 加密服务
│   │   ├── __init__.py
│   │   ├── encryptor.py    # 加密器
│   │   └── key_manager.py  # 密钥管理器
│   ├── forge_agent/        # Forge Agent
│   │   ├── __init__.py
│   │   ├── agent.py        # Agent 实现
│   │   └── tasks.py        # 任务处理器
│   ├── pdf/                # PDF 处理
│   │   ├── __init__.py
│   │   └── parser.py       # PDF 解析器
│   ├── scheduler/          # 任务调度
│   │   ├── __init__.py
│   │   ├── cron.py         # Cron 调度器
│   │   └── queue.py        # 队列管理
│   ├── security/           # 安全模块
│   │   ├── __init__.py
│   │   ├── auth.py         # 认证
│   │   ├── rbac.py         # 基于角色的访问控制
│   │   └── rate_limiter.py # 速率限制
│   ├── streaming/          # 流媒体处理
│   │   ├── __init__.py
│   │   └── websocket.py    # WebSocket 管理
│   ├── telemetry/          #遥测系统
│   │   ├── __init__.py
│   │   ├── metrics.py      # 指标收集
│   │   └── tracer.py       # 分布式追踪
│   └── webhook/            # Webhook 管理
│       ├── __init__.py
│       ├── manager.py      # Webhook 管理器
│       └── verifier.py     # Webhook 验证器
├── prompts.py              # 提示词管理
├── responses.py            # 响应模型
├── settings.py             # 配置设置
└── constants.py            # 常量定义
```

## 核心组件

### 1. FastAPI 应用 (`api_app.py`)
```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期管理"""
    # 启动时执行
    await initialize_services()
    yield
    # 关闭时执行
    await cleanup_services()

app = FastAPI(
    title="Skyvern API",
    description="AI-powered browser automation platform",
    version="2.0.0",
    lifespan=lifespan
)

# 中间件配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 2. 依赖注入 (`dependencies.py`)
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from sqlalchemy.ext.asyncio import AsyncSession

security = HTTPBearer()

async def get_current_user(
    token: str = Depends(security)
) -> User:
    """获取当前用户"""
    user = await authenticate_token(token.credentials)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )
    return user

async def get_db_session() -> AsyncSession:
    """获取数据库会话"""
    async with get_async_session() as session:
        try:
            yield session
        finally:
            await session.close()
```

### 3. 中间件 (`middleware.py`)
```python
import time
import uuid
from fastapi import Request, Response
from starlette.middleware.base import BaseHTTPMiddleware

class RequestIDMiddleware(BaseHTTPMiddleware):
    """请求ID中间件"""

    async def dispatch(self, request: Request, call_next):
        request_id = str(uuid.uuid4())
        request.state.request_id = request_id

        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response

class LoggingMiddleware(BaseHTTPMiddleware):
    """日志中间件"""

    async def dispatch(self, request: Request, call_next):
        start_time = time.time()

        response = await call_next(request)

        process_time = time.time() - start_time
        logger.info(
            "Request processed",
            method=request.method,
            url=str(request.url),
            status_code=response.status_code,
            process_time=process_time
        )

        return response
```

## API 路由详解

### 1. 任务管理 (`api/v1/tasks.py`)
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sdk.db.models import Task, TaskStatus

router = APIRouter(prefix="/tasks", tags=["tasks"])

@router.post("/", response_model=TaskResponse)
async def create_task(
    task_request: CreateTaskRequest,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db_session)
) -> TaskResponse:
    """创建新任务"""
    task = await task_service.create_task(
        db=db,
        user_id=current_user.user_id,
        request=task_request
    )
    return TaskResponse.from_task(task)

@router.get("/{task_id}", response_model=TaskResponse)
async def get_task(
    task_id: str,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db_session)
) -> TaskResponse:
    """获取任务详情"""
    task = await task_service.get_task(
        db=db,
        task_id=task_id,
        user_id=current_user.user_id
    )
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found"
        )
    return TaskResponse.from_task(task)

@router.post("/{task_id}/execute")
async def execute_task(
    task_id: str,
    execute_request: ExecuteTaskRequest,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db_session)
) -> ExecuteTaskResponse:
    """执行任务"""
    execution_id = await task_service.execute_task(
        db=db,
        task_id=task_id,
        user_id=current_user.user_id,
        request=execute_request
    )
    return ExecuteTaskResponse(execution_id=execution_id)
```

### 2. 工作流管理 (`api/v1/workflows.py`)
```python
from fastapi import APIRouter, Depends, HTTPException
from sdk.db.models import Workflow, WorkflowRun

router = APIRouter(prefix="/workflows", tags=["workflows"])

@router.post("/", response_model=WorkflowResponse)
async def create_workflow(
    workflow_request: CreateWorkflowRequest,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db_session)
) -> WorkflowResponse:
    """创建工作流"""
    workflow = await workflow_service.create_workflow(
        db=db,
        user_id=current_user.user_id,
        request=workflow_request
    )
    return WorkflowResponse.from_workflow(workflow)

@router.post("/{workflow_id}/run")
async def run_workflow(
    workflow_id: str,
    run_request: RunWorkflowRequest,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db_session)
) -> WorkflowRunResponse:
    """运行工作流"""
    run = await workflow_service.run_workflow(
        db=db,
        workflow_id=workflow_id,
        user_id=current_user.user_id,
        request=run_request
    )
    return WorkflowRunResponse.from_run(run)
```

### 3. 浏览器会话 (`api/v1/browser_sessions.py`)
```python
from fastapi import APIRouter, Depends, WebSocket, WebSocketDisconnect
from sdk.webeye.browser_manager import BrowserManager

router = APIRouter(prefix="/browser-sessions", tags=["browser-sessions"])

@router.post("/", response_model=BrowserSessionResponse)
async def create_browser_session(
    session_request: CreateBrowserSessionRequest,
    current_user: User = Depends(get_current_user)
) -> BrowserSessionResponse:
    """创建浏览器会话"""
    session = await browser_manager.create_session(
        user_id=current_user.user_id,
        request=session_request
    )
    return BrowserSessionResponse.from_session(session)

@router.websocket("/{session_id}/stream")
async def websocket_stream(
    websocket: WebSocket,
    session_id: str
):
    """WebSocket 实时流"""
    await websocket.accept()

    try:
        async for message in stream_manager.get_stream(session_id):
            await websocket.send_json(message)
    except WebSocketDisconnect:
        await stream_manager.disconnect(session_id)
```

## SDK 组件详解

### 1. LLM 集成 (`sdk/api/llm/`)
```python
# base.py
from abc import ABC, abstractmethod
from typing import Optional, Dict, Any, List

class BaseLLMClient(ABC):
    """LLM 客户端基类"""

    @abstractmethod
    async def generate_text(
        self,
        prompt: str,
        max_tokens: Optional[int] = None,
        temperature: Optional[float] = None,
        **kwargs
    ) -> str:
        """生成文本"""
        pass

    @abstractmethod
    async def generate_vision_response(
        self,
        prompt: str,
        image_data: bytes,
        max_tokens: Optional[int] = None,
        **kwargs
    ) -> str:
        """生成视觉响应"""
        pass

# openai.py
class OpenAIClient(BaseLLMClient):
    """OpenAI 客户端实现"""

    def __init__(self, api_key: str, model: str = "gpt-4-vision-preview"):
        self.client = AsyncOpenAI(api_key=api_key)
        self.model = model

    async def generate_vision_response(
        self,
        prompt: str,
        image_data: bytes,
        max_tokens: Optional[int] = None,
        **kwargs
    ) -> str:
        response = await self.client.chat.completions.create(
            model=self.model,
            messages=[
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": prompt},
                        {
                            "type": "image_url",
                            "image_url": {
                                "url": f"data:image/png;base64,{base64.b64encode(image_data).decode()}"
                            }
                        }
                    ]
                }
            ],
            max_tokens=max_tokens
        )
        return response.choices[0].message.content
```

### 2. 数据库访问 (`sdk/db/`)
```python
# engine.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class DatabaseEngine:
    def __init__(self, database_url: str):
        self.engine = create_async_engine(
            database_url,
            echo=settings.DEBUG,
            pool_pre_ping=True,
            pool_recycle=300
        )
        self.async_session = async_sessionmaker(
            self.engine,
            class_=AsyncSession,
            expire_on_commit=False
        )

    async def get_session(self) -> AsyncSession:
        async with self.async_session() as session:
            try:
                yield session
            finally:
                await session.close()

# crud.py
class CRUDBase:
    """基础 CRUD 操作"""

    def __init__(self, model):
        self.model = model

    async def get(
        self,
        db: AsyncSession,
        id: Any
    ) -> Optional[ModelType]:
        """获取单个记录"""
        result = await db.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()

    async def get_multi(
        self,
        db: AsyncSession,
        skip: int = 0,
        limit: int = 100
    ) -> List[ModelType]:
        """获取多个记录"""
        result = await db.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return result.scalars().all()

    async def create(
        self,
        db: AsyncSession,
        obj_in: Dict[str, Any]
    ) -> ModelType:
        """创建记录"""
        db_obj = self.model(**obj_in)
        db.add(db_obj)
        await db.commit()
        await db.refresh(db_obj)
        return db_obj
```

### 3. 缓存系统 (`sdk/cache/`)
```python
# cache_manager.py
from typing import Optional, Any, Union
import json
import pickle

class CacheManager:
    """缓存管理器"""

    def __init__(self, redis_client: Redis):
        self.redis = redis_client

    async def get(
        self,
        key: str,
        default: Any = None
    ) -> Any:
        """获取缓存值"""
        value = await self.redis.get(key)
        if value is None:
            return default
        try:
            return pickle.loads(value)
        except:
            return value.decode()

    async def set(
        self,
        key: str,
        value: Any,
        expire: Optional[int] = None
    ) -> None:
        """设置缓存值"""
        if isinstance(value, (dict, list, tuple)):
            value = pickle.dumps(value)
        await self.redis.set(key, value, ex=expire)

    async def delete(self, key: str) -> None:
        """删除缓存"""
        await self.redis.delete(key)

    async def invalidate_pattern(self, pattern: str) -> None:
        """按模式删除缓存"""
        keys = await self.redis.keys(pattern)
        if keys:
            await self.redis.delete(*keys)
```

### 4. WebSocket 流媒体 (`sdk/streaming/websocket.py`)
```python
from fastapi import WebSocket, WebSocketDisconnect
from typing import Dict, List
import json
import asyncio

class ConnectionManager:
    """WebSocket 连接管理器"""

    def __init__(self):
        self.active_connections: Dict[str, List[WebSocket]] = {}

    async def connect(self, websocket: WebSocket, session_id: str):
        """建立连接"""
        await websocket.accept()
        if session_id not in self.active_connections:
            self.active_connections[session_id] = []
        self.active_connections[session_id].append(websocket)

    def disconnect(self, websocket: WebSocket, session_id: str):
        """断开连接"""
        if session_id in self.active_connections:
            self.active_connections[session_id].remove(websocket)
            if not self.active_connections[session_id]:
                del self.active_connections[session_id]

    async def send_personal_message(
        self,
        message: Dict,
        websocket: WebSocket
    ):
        """发送个人消息"""
        await websocket.send_json(message)

    async def broadcast_to_session(
        self,
        message: Dict,
        session_id: str
    ):
        """向会话广播消息"""
        if session_id in self.active_connections:
            for connection in self.active_connections[session_id]:
                try:
                    await connection.send_json(message)
                except:
                    # 移除无效连接
                    self.disconnect(connection, session_id)
```

## 提示词管理 (`prompts.py`)

```python
from typing import Dict, Any
import jinja2

class PromptManager:
    """提示词管理器"""

    def __init__(self):
        self.env = jinja2.Environment(
            loader=jinja2.DictLoader({
                'task_analysis': TASK_ANALYSIS_PROMPT,
                'action_planning': ACTION_PLANNING_PROMPT,
                'element_detection': ELEMENT_DETECTION_PROMPT,
                'error_recovery': ERROR_RECOVERY_PROMPT,
            })
        )

    def render_prompt(
        self,
        template_name: str,
        context: Dict[str, Any]
    ) -> str:
        """渲染提示词模板"""
        template = self.env.get_template(template_name)
        return template.render(**context)

# 提示词模板
TASK_ANALYSIS_PROMPT = """
你是一个智能网页自动化助手。请分析以下任务：

任务描述：{{ task_description }}
页面信息：{{ page_info }}

请提供：
1. 任务分解步骤
2. 需要的操作类型
3. 可能的困难点
4. 建议的执行策略
"""

ACTION_PLANNING_PROMPT = """
基于当前页面状态和任务目标，规划下一步操作：

当前页面：{{ current_state }}
任务目标：{{ task_goal }}
历史操作：{{ history }}

请决定：
1. 最优操作类型
2. 目标元素选择器
3. 操作参数
4. 预期结果
"""
```

## 配置管理 (`settings.py`)

```python
from pydantic_settings import BaseSettings
from typing import List, Optional

class Settings(BaseSettings):
    """应用配置"""

    # 应用基础配置
    APP_NAME: str = "Skyvern API"
    VERSION: str = "2.0.0"
    DEBUG: bool = False

    # 数据库配置
    DATABASE_URL: str

    # Redis 配置
    REDIS_URL: str

    # LLM 配置
    LLM_PROVIDER: str = "openai"
    OPENAI_API_KEY: Optional[str] = None
    ANTHROPIC_API_KEY: Optional[str] = None

    # 安全配置
    SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    # CORS 配置
    ALLOWED_ORIGINS: List[str] = ["http://localhost:3000"]

    # 浏览器配置
    BROWSER_HEADLESS: bool = True
    BROWSER_TIMEOUT: int = 30000

    # 文件存储配置
    STORAGE_TYPE: str = "local"
    STORAGE_PATH: str = "./storage"
    AWS_S3_BUCKET: Optional[str] = None

    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()
```

## 异常处理 (`exceptions.py`)

```python
from fastapi import HTTPException, Request, status
from fastapi.responses import JSONResponse
from typing import Dict, Any

class SkyvernException(Exception):
    """Skyvern 基础异常"""
    def __init__(
        self,
        message: str,
        error_code: str,
        status_code: int = status.HTTP_500_INTERNAL_SERVER_ERROR,
        details: Dict[str, Any] = None
    ):
        self.message = message
        self.error_code = error_code
        self.status_code = status_code
        self.details = details or {}
        super().__init__(message)

class TaskNotFoundException(SkyvernException):
    """任务未找到异常"""
    def __init__(self, task_id: str):
        super().__init__(
            message=f"Task {task_id} not found",
            error_code="TASK_NOT_FOUND",
            status_code=status.HTTP_404_NOT_FOUND
        )

class WorkflowExecutionError(SkyvernException):
    """工作流执行异常"""
    def __init__(self, workflow_id: str, reason: str):
        super().__init__(
            message=f"Workflow {workflow_id} execution failed: {reason}",
            error_code="WORKFLOW_EXECUTION_ERROR",
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            details={"workflow_id": workflow_id, "reason": reason}
        )

# 异常处理器
async def skyvern_exception_handler(
    request: Request,
    exc: SkyvernException
) -> JSONResponse:
    """Skyvern 异常处理器"""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.error_code,
                "message": exc.message,
                "details": exc.details
            }
        }
    )
```

## 性能优化

### 1. 数据库优化
- 使用连接池
- 实现查询缓存
- 优化索引策略
- 批量操作优化

### 2. API 响应优化
- 实现响应压缩
- 使用分页查询
- 缓存频繁查询
- 异步处理长时间任务

### 3. 内存管理
- 限制并发请求数
- 实现请求队列
- 监控内存使用
- 及时释放资源

## 安全最佳实践

### 1. 认证与授权
- JWT 令牌认证
- 基于角色的访问控制
- API 速率限制
- 请求签名验证

### 2. 数据保护
- 敏感数据加密
- 输入验证和清理
- SQL 注入防护
- XSS 攻击防护

### 3. 网络安全
- HTTPS 强制使用
- CORS 策略配置
- 安全头部设置
- DDoS 防护

## 监控与日志

### 1. 日志管理
```python
import structlog

logger = structlog.get_logger()

# 结构化日志
logger.info(
    "Task executed",
    task_id=task_id,
    user_id=user_id,
    execution_time=execution_time,
    status=status
)
```

### 2. 指标收集
- API 响应时间
- 请求成功率
- 错误率统计
- 资源使用情况

### 3. 健康检查
```python
@router.get("/health")
async def health_check():
    """健康检查端点"""
    checks = {
        "database": await check_database_health(),
        "redis": await check_redis_health(),
        "llm": await check_llm_health()
    }

    status = "healthy" if all(checks.values()) else "unhealthy"

    return {
        "status": status,
        "checks": checks,
        "timestamp": datetime.utcnow().isoformat()
    }
```

## 部署配置

### 1. Docker 部署
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 2. Kubernetes 部署
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: skyvern-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: skyvern-api
  template:
    metadata:
      labels:
        app: skyvern-api
    spec:
      containers:
      - name: api
        image: skyvern/api:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: skyvern-secrets
              key: database-url
```

---

> 📖 **返回**: [Skyvern 根文档](../CLAUDE.md) | [WebEye 模块](../webeye/CLAUDE.md) | [Services 模块](../services/CLAUDE.md)