# Services 模块文档

> 📍 **位置**: [skyvern](../CLAUDE.md) → services

## 模块概述

Services 模块是 Skyvern 的业务逻辑层，包含所有核心业务服务。该模块负责处理任务执行、工作流编排、浏览器会话管理、凭据管理等核心业务功能，是系统的业务中枢。

## 文件结构

```
skyvern/services/
├── __init__.py
├── auth_service.py          # 认证服务
├── task_service.py          # 任务服务
├── task_v1_service.py       # 任务服务 v1
├── task_v2_service.py       # 任务服务 v2
├── workflow_service.py      # 工作流服务
├── workflow_v2_service.py   # 工作流服务 v2
├── action_service.py        # 动作服务
├── artifact_service.py      # 工件服务
├── browser_service.py       # 浏览器服务
├── browser_session_service.py  # 浏览器会话服务
├── credential_service.py    # 凭据服务
├── data_source_service.py   # 数据源服务
├── organization_service.py  # 组织服务
├── proxy_service.py         # 代理服务
├── run_service.py           # 执行记录服务
├── user_service.py          # 用户服务
├── webhook_service.py       # Webhook 服务
├── agent_service.py         # 代理服务
├── server_service.py        # 服务器服务
├── browser_recording/       # 浏览器录制功能
│   ├── __init__.py
│   ├── recorder.py          # 录制器
│   ├── storage.py           # 存储管理
│   └── player.py            # 播放器
├── json_to_workflow/        # JSON转工作流
│   ├── __init__.py
│   ├── converter.py         # 转换器
│   └── validator.py         # 验证器
├── agent_v1/                # 代理系统 v1（已弃用）
│   ├── __init__.py
│   ├── agent.py
│   ├── prompts.py
│   └── types.py
├── auxiliate/               # 辅助服务
│   ├── __init__.py
│   ├── llm.py               # LLM 辅助服务
│   ├── token_counter.py     # Token 计数器
│   └── email_sender.py      # 邮件发送器
├── metrics/                 # 指标服务
│   ├── __init__.py
│   ├── collector.py         # 指标收集器
│   └── reporter.py          # 指标报告器
└── exceptions.py            # 服务异常
```

## 核心服务详解

### 1. 任务服务 (`task_service.py`)
```python
from typing import Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from forge.sdk.db.models import Task, TaskStatus, TaskExecution
from forge.sdk.db.crud import CRUDBase

class TaskService:
    """任务管理服务"""

    def __init__(self):
        self.task_crud = CRUDBase(Task)
        self.execution_crud = CRUDBase(TaskExecution)

    async def create_task(
        self,
        db: AsyncSession,
        user_id: str,
        title: str,
        url: str,
        goal: str,
        webhook_callback_url: Optional[str] = None,
        proxy: Optional[ProxySettings] = None,
        proxy_location: Optional[str] = None,
        organization_id: Optional[str] = None
    ) -> Task:
        """创建新任务"""
        task = Task(
            task_id=generate_uuid(),
            user_id=user_id,
            organization_id=organization_id,
            title=title,
            url=url,
            goal=goal,
            webhook_callback_url=webhook_callback_url,
            proxy_location=proxy_location,
            status=TaskStatus.PENDING,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )
        return await self.task_crud.create(db, task.dict())

    async def execute_task(
        self,
        db: AsyncSession,
        task_id: str,
        user_id: str,
        prompt_adjustment: Optional[str] = None
    ) -> TaskExecution:
        """执行任务"""
        task = await self.task_crud.get(db, task_id)
        if not task or task.user_id != user_id:
            raise TaskNotFoundException(task_id)

        execution = TaskExecution(
            execution_id=generate_uuid(),
            task_id=task_id,
            user_id=user_id,
            status=TaskStatus.RUNNING,
            prompt_adjustment=prompt_adjustment,
            started_at=datetime.utcnow()
        )

        # 异步执行任务
        asyncio.create_task(self._run_task_async(execution))

        return await self.execution_crud.create(db, execution.dict())

    async def _run_task_async(self, execution: TaskExecution):
        """异步执行任务逻辑"""
        try:
            # 初始化浏览器
            browser = await browser_manager.get_browser(execution.task_id)
            page = await browser.new_page()

            # 导航到目标URL
            await page.goto(execution.task.url)

            # 创建并运行代理
            agent = WebAgent(
                browser=browser,
                page=page,
                task=execution.task
            )
            result = await agent.execute()

            # 更新执行状态
            execution.status = TaskStatus.COMPLETED if result.success else TaskStatus.FAILED
            execution.completed_at = datetime.utcnow()
            execution.result = result.to_dict()

        except Exception as e:
            execution.status = TaskStatus.FAILED
            execution.error_message = str(e)
            execution.completed_at = datetime.utcnow()

        finally:
            # 清理资源
            await browser.close()
```

### 2. 工作流服务 (`workflow_service.py`)
```python
from typing import Dict, Any, List
from forge.sdk.db.models import Workflow, WorkflowRun, WorkflowBlock, BlockType

class WorkflowService:
    """工作流管理服务"""

    def __init__(self):
        self.workflow_crud = CRUDBase(Workflow)
        self.run_crud = CRUDBase(WorkflowRun)
        self.block_crud = CRUDBase(WorkflowBlock)

    async def create_workflow(
        self,
        db: AsyncSession,
        user_id: str,
        title: str,
        description: str,
        blocks: List[Dict[str, Any]],
        organization_id: Optional[str] = None
    ) -> Workflow:
        """创建工作流"""
        workflow = Workflow(
            workflow_id=generate_uuid(),
            user_id=user_id,
            organization_id=organization_id,
            title=title,
            description=description,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )

        # 创建工作流
        workflow = await self.workflow_crud.create(db, workflow.dict())

        # 创建工作流块
        for block_data in blocks:
            block = WorkflowBlock(
                block_id=generate_uuid(),
                workflow_id=workflow.workflow_id,
                block_type=BlockType(block_data["type"]),
                label=block_data.get("label"),
                data=block_data.get("data", {}),
                parameters=block_data.get("parameters", {}),
                output_schema=block_data.get("output_schema"),
                created_at=datetime.utcnow()
            )
            await self.block_crud.create(db, block.dict())

        return workflow

    async def run_workflow(
        self,
        db: AsyncSession,
        workflow_id: str,
        user_id: str,
        data: Optional[Dict[str, Any]] = None,
        proxy_location: Optional[str] = None
    ) -> WorkflowRun:
        """运行工作流"""
        workflow = await self.workflow_crud.get(db, workflow_id)
        if not workflow or workflow.user_id != user_id:
            raise WorkflowNotFoundException(workflow_id)

        # 获取工作流块
        blocks = await self.block_crud.get_multi(
            db,
            filters={"workflow_id": workflow_id},
            order_by={"created_at": "asc"}
        )

        # 创建运行实例
        run = WorkflowRun(
            run_id=generate_uuid(),
            workflow_id=workflow_id,
            user_id=user_id,
            status="running",
            data=data or {},
            proxy_location=proxy_location,
            started_at=datetime.utcnow()
        )

        run = await self.run_crud.create(db, run.dict())

        # 异步执行工作流
        asyncio.create_task(self._execute_workflow(run, blocks))

        return run

    async def _execute_workflow(
        self,
        run: WorkflowRun,
        blocks: List[WorkflowBlock]
    ):
        """执行工作流逻辑"""
        execution_context = ExecutionContext(
            run=run,
            data=run.data,
            variables={}
        )

        try:
            for block in blocks:
                # 执行块
                executor = self._get_block_executor(block.block_type)
                result = await executor.execute(block, execution_context)

                # 更新执行上下文
                execution_context.variables[block.block_id] = result
                execution_context.data.update(result.get("output", {}))

            # 标记为完成
            run.status = "completed"
            run.completed_at = datetime.utcnow()
            run.output = execution_context.data

        except Exception as e:
            run.status = "failed"
            run.error_message = str(e)
            run.completed_at = datetime.utcnow()

    def _get_block_executor(self, block_type: BlockType):
        """获取块执行器"""
        executors = {
            BlockType.NAVIGATION: NavigationBlockExecutor(),
            BlockType.EXTRACTION: ExtractionBlockExecutor(),
            BlockType.VALIDATION: ValidationBlockExecutor(),
            BlockType.LOOP: LoopBlockExecutor(),
            BlockType.CONDITIONAL: ConditionalBlockExecutor(),
            BlockType.CODE: CodeBlockExecutor(),
            BlockType.WAIT: WaitBlockExecutor(),
            BlockType.UPLOAD: UploadBlockExecutor(),
            BlockType.DOWNLOAD: DownloadBlockExecutor(),
            BlockType.EMAIL: EmailBlockExecutor(),
            BlockType.WEBHOOK: WebhookBlockExecutor(),
            BlockType.TERMINATE: TerminateBlockExecutor(),
        }
        return executors.get(block_type)
```

### 3. 浏览器会话服务 (`browser_session_service.py`)
```python
from typing import Optional, Dict, Any
from forge.sdk.db.models import BrowserSession, SessionStatus

class BrowserSessionService:
    """浏览器会话管理服务"""

    def __init__(self):
        self.session_crud = CRUDBase(BrowserSession)
        self.active_sessions: Dict[str, BrowserContext] = {}

    async def create_session(
        self,
        db: AsyncSession,
        user_id: str,
        title: str,
        url: Optional[str] = None,
        proxy_location: Optional[str] = None,
        user_data_dir: Optional[str] = None,
        organization_id: Optional[str] = None
    ) -> BrowserSession:
        """创建浏览器会话"""
        session = BrowserSession(
            session_id=generate_uuid(),
            user_id=user_id,
            organization_id=organization_id,
            title=title,
            url=url,
            proxy_location=proxy_location,
            user_data_dir=user_data_dir,
            status=SessionStatus.ACTIVE,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )

        # 初始化浏览器上下文
        browser = await browser_manager.get_browser(
            session_id=session.session_id,
            proxy_location=proxy_location
        )
        context = await browser.new_context(
            user_data_dir=user_data_dir,
            viewport={"width": 1920, "height": 1080}
        )

        self.active_sessions[session.session_id] = context

        # 如果提供了URL，导航到该页面
        if url:
            page = await context.new_page()
            await page.goto(url)

        return await self.session_crud.create(db, session.dict())

    async def execute_action_in_session(
        self,
        db: AsyncSession,
        session_id: str,
        user_id: str,
        action_type: str,
        parameters: Dict[str, Any]
    ) -> Dict[str, Any]:
        """在会话中执行动作"""
        session = await self.session_crud.get(db, session_id)
        if not session or session.user_id != user_id:
            raise SessionNotFoundException(session_id)

        context = self.active_sessions.get(session_id)
        if not context:
            raise SessionNotActiveException(session_id)

        # 获取当前页面
        pages = context.pages
        page = pages[-1] if pages else await context.new_page()

        # 执行动作
        executor = ActionExecutor()
        action = Action(
            action_id=generate_uuid(),
            action_type=action_type,
            parameters=parameters
        )

        result = await executor.execute_action(page, action)

        # 更新会话时间
        session.updated_at = datetime.utcnow()
        await self.session_crud.update(db, session_id, session.dict())

        return result.to_dict()

    async def close_session(
        self,
        db: AsyncSession,
        session_id: str,
        user_id: str
    ) -> None:
        """关闭浏览器会话"""
        session = await self.session_crud.get(db, session_id)
        if not session or session.user_id != user_id:
            raise SessionNotFoundException(session_id)

        # 关闭浏览器上下文
        context = self.active_sessions.pop(session_id, None)
        if context:
            await context.close()

        # 更新状态
        session.status = SessionStatus.CLOSED
        session.updated_at = datetime.utcnow()
        await self.session_crud.update(db, session_id, session.dict())
```

### 4. 凭据服务 (`credential_service.py`)
```python
from cryptography.fernet import Fernet
from forge.sdk.db.models import Credential, CredentialType

class CredentialService:
    """凭据管理服务"""

    def __init__(self):
        self.credential_crud = CRUDBase(Credential)
        self.cipher = Fernet(settings.ENCRYPTION_KEY.encode())

    async def create_credential(
        self,
        db: AsyncSession,
        user_id: str,
        credential_type: CredentialType,
        name: str,
        data: Dict[str, Any],
        organization_id: Optional[str] = None
    ) -> Credential:
        """创建凭据"""
        # 加密敏感数据
        encrypted_data = self._encrypt_data(data)

        credential = Credential(
            credential_id=generate_uuid(),
            user_id=user_id,
            organization_id=organization_id,
            credential_type=credential_type,
            name=name,
            encrypted_data=encrypted_data,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )

        return await self.credential_crud.create(db, credential.dict())

    async def get_credential_data(
        self,
        db: AsyncSession,
        credential_id: str,
        user_id: str
    ) -> Dict[str, Any]:
        """获取解密后的凭据数据"""
        credential = await self.credential_crud.get(db, credential_id)
        if not credential or credential.user_id != user_id:
            raise CredentialNotFoundException(credential_id)

        return self._decrypt_data(credential.encrypted_data)

    def _encrypt_data(self, data: Dict[str, Any]) -> str:
        """加密数据"""
        json_data = json.dumps(data)
        return self.cipher.encrypt(json_data.encode()).decode()

    def _decrypt_data(self, encrypted_data: str) -> Dict[str, Any]:
        """解密数据"""
        decrypted = self.cipher.decrypt(encrypted_data.encode())
        return json.loads(decrypted.decode())

    async def auto_fill_credentials(
        self,
        db: AsyncSession,
        page: Page,
        domain: str,
        user_id: str
    ) -> bool:
        """自动填充凭据"""
        # 查找匹配的凭据
        credentials = await self.credential_crud.get_multi(
            db,
            filters={"user_id": user_id, "domain": domain}
        )

        if not credentials:
            return False

        # 获取第一个匹配的凭据
        credential = credentials[0]
        data = await self.get_credential_data(db, credential.credential_id, user_id)

        # 填充用户名和密码
        if "username" in data and "password" in data:
            await page.fill('input[name="username"], input[type="email"]', data["username"])
            await page.fill('input[name="password"], input[type="password"]', data["password"])
            return True

        return False
```

### 5. Webhook 服务 (`webhook_service.py`)
```python
import aiohttp
from typing import Dict, Any
from forge.sdk.db.models import Webhook, WebhookEvent

class WebhookService:
    """Webhook 管理服务"""

    def __init__(self):
        self.webhook_crud = CRUDBase(Webhook)
        self.session = aiohttp.ClientSession()

    async def create_webhook(
        self,
        db: AsyncSession,
        user_id: str,
        url: str,
        events: List[WebhookEvent],
        secret: Optional[str] = None,
        organization_id: Optional[str] = None
    ) -> Webhook:
        """创建 Webhook"""
        webhook = Webhook(
            webhook_id=generate_uuid(),
            user_id=user_id,
            organization_id=organization_id,
            url=url,
            events=events,
            secret=secret or generate_secret(),
            active=True,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )

        return await self.webhook_crud.create(db, webhook.dict())

    async def trigger_webhook(
        self,
        db: AsyncSession,
        event: WebhookEvent,
        data: Dict[str, Any]
    ) -> List[bool]:
        """触发 Webhook"""
        # 查找匹配的 Webhook
        webhooks = await self.webhook_crud.get_multi(
            db,
            filters={
                "events": event,
                "active": True
            }
        )

        results = []
        for webhook in webhooks:
            success = await self._send_webhook(webhook, event, data)
            results.append(success)

        return results

    async def _send_webhook(
        self,
        webhook: Webhook,
        event: WebhookEvent,
        data: Dict[str, Any]
    ) -> bool:
        """发送 Webhook 请求"""
        try:
            payload = {
                "event": event.value,
                "timestamp": datetime.utcnow().isoformat(),
                "data": data
            }

            # 生成签名
            signature = self._generate_signature(webhook.secret, payload)

            headers = {
                "Content-Type": "application/json",
                "X-Skyvern-Webhook-Signature": signature,
                "X-Skyvern-Event": event.value
            }

            async with self.session.post(
                webhook.url,
                json=payload,
                headers=headers,
                timeout=aiohttp.ClientTimeout(total=30)
            ) as response:
                return response.status < 400

        except Exception:
            return False

    def _generate_signature(self, secret: str, payload: Dict) -> str:
        """生成签名"""
        import hmac
        import hashlib

        payload_str = json.dumps(payload, sort_keys=True)
        return hmac.new(
            secret.encode(),
            payload_str.encode(),
            hashlib.sha256
        ).hexdigest()
```

## 浏览器录制功能 (`browser_recording/`)

### 录制器 (`recorder.py`)
```python
class BrowserRecorder:
    """浏览器录制器"""

    def __init__(self, session_id: str, storage: RecordingStorage):
        self.session_id = session_id
        self.storage = storage
        self.actions: List[RecordedAction] = []
        self.screenshots: List[RecordedScreenshot] = []

    async def start_recording(self, browser: Browser):
        """开始录制"""
        # 注入录制脚本
        await browser.add_init_script("""
            window.skyvernRecording = true;
            window.skyvernActions = [];

            // 监听点击事件
            document.addEventListener('click', function(e) {
                window.skyvernActions.push({
                    type: 'click',
                    selector: getSelector(e.target),
                    timestamp: Date.now()
                });
            });

            // 生成选择器
            function getSelector(element) {
                if (element.id) return '#' + element.id;
                if (element.className) return '.' + element.className.split(' ').join('.');
                return element.tagName.toLowerCase();
            }
        """)

    async def record_action(
        self,
        action_type: str,
        selector: str,
        parameters: Dict[str, Any]
    ):
        """记录动作"""
        recorded_action = RecordedAction(
            action_id=generate_uuid(),
            action_type=action_type,
            selector=selector,
            parameters=parameters,
            timestamp=datetime.utcnow()
        )
        self.actions.append(recorded_action)

    async def take_screenshot(self, page: Page, action_id: str):
        """录制截图"""
        screenshot_bytes = await page.screenshot(full_page=True)

        screenshot = RecordedScreenshot(
            screenshot_id=generate_uuid(),
            action_id=action_id,
            data=screenshot_bytes,
            timestamp=datetime.utcnow()
        )
        self.screenshots.append(screenshot)

    async def save_recording(self) -> RecordingSession:
        """保存录制"""
        session = RecordingSession(
            session_id=self.session_id,
            actions=self.actions,
            screenshots=self.screenshots,
            created_at=datetime.utcnow()
        )
        return await self.storage.save(session)
```

## 辅助服务 (`auxiliate/`)

### Token 计数器 (`token_counter.py`)
```python
import tiktoken

class TokenCounter:
    """Token 计数器"""

    def __init__(self, model: str = "gpt-4"):
        try:
            self.encoder = tiktoken.encoding_for_model(model)
        except KeyError:
            self.encoder = tiktoken.get_encoding("cl100k_base")

    def count_tokens(self, text: str) -> int:
        """计算文本的 token 数量"""
        return len(self.encoder.encode(text))

    def count_message_tokens(self, messages: List[Dict[str, str]]) -> int:
        """计算消息列表的 token 数量"""
        total = 0
        for message in messages:
            total += self.count_tokens(message.get("content", ""))
            total += self.count_tokens(message.get("role", ""))
            # 添加每个消息的固定开销
            total += 4
        # 添加整个消息的固定开销
        total += 3
        return total

    def estimate_cost(
        self,
        tokens: int,
        model: str,
        token_type: str = "input"
    ) -> float:
        """估算 token 成本"""
        pricing = {
            "gpt-4": {"input": 0.03, "output": 0.06},
            "gpt-4-turbo": {"input": 0.01, "output": 0.03},
            "gpt-3.5-turbo": {"input": 0.0015, "output": 0.002},
        }

        if model not in pricing:
            return 0.0

        cost_per_1k = pricing[model][token_type]
        return (tokens / 1000) * cost_per_1k
```

## 异常处理 (`exceptions.py`)

```python
class ServiceException(Exception):
    """服务基础异常"""
    def __init__(
        self,
        message: str,
        error_code: str,
        details: Dict[str, Any] = None
    ):
        self.message = message
        self.error_code = error_code
        self.details = details or {}
        super().__init__(message)

class TaskExecutionError(ServiceException):
    """任务执行异常"""
    def __init__(self, task_id: str, reason: str):
        super().__init__(
            message=f"Task {task_id} execution failed: {reason}",
            error_code="TASK_EXECUTION_ERROR",
            details={"task_id": task_id, "reason": reason}
        )

class WorkflowExecutionError(ServiceException):
    """工作流执行异常"""
    def __init__(self, workflow_id: str, block_id: str, reason: str):
        super().__init__(
            message=f"Workflow {workflow_id} block {block_id} failed: {reason}",
            error_code="WORKFLOW_EXECUTION_ERROR",
            details={
                "workflow_id": workflow_id,
                "block_id": block_id,
                "reason": reason
            }
        )

class BrowserSessionError(ServiceException):
    """浏览器会话异常"""
    def __init__(self, session_id: str, reason: str):
        super().__init__(
            message=f"Browser session {session_id} error: {reason}",
            error_code="BROWSER_SESSION_ERROR",
            details={"session_id": session_id, "reason": reason}
        )
```

## 性能优化策略

### 1. 异步处理
- 所有 I/O 操作使用 async/await
- 并发执行多个任务
- 使用异步队列处理长时间运行的任务

### 2. 缓存策略
- 缓存频繁查询的数据
- 使用 Redis 缓存任务状态
- 实现智能缓存失效机制

### 3. 资源管理
- 及时释放浏览器资源
- 限制并发任务数量
- 实现资源池化

### 4. 批量操作
- 批量数据库操作
- 批量 Webhook 触发
- 批量消息处理

## 监控与日志

### 1. 服务监控
```python
from prometheus_client import Counter, Histogram, Gauge

# 指标定义
task_counter = Counter('skyvern_tasks_total', 'Total tasks', ['status', 'user_id'])
task_duration = Histogram('skyvern_task_duration_seconds', 'Task execution duration')
active_sessions = Gauge('skyvern_active_sessions', 'Active browser sessions')
```

### 2. 结构化日志
```python
import structlog

logger = structlog.get_logger()

# 任务日志
logger.info(
    "Task executed",
    task_id=task_id,
    user_id=user_id,
    duration=duration,
    status=status,
    steps_count=steps_count
)

# 工作流日志
logger.info(
    "Workflow executed",
    workflow_id=workflow_id,
    run_id=run_id,
    blocks_executed=blocks_executed,
    total_duration=total_duration
)
```

## 测试策略

### 1. 单元测试
- 测试每个服务的核心功能
- Mock 外部依赖
- 验证异常处理

### 2. 集成测试
- 测试服务间的交互
- 使用测试数据库
- 验证端到端流程

### 3. 性能测试
- 负载测试
- 压力测试
- 资源使用测试

---

> 📖 **返回**: [Skyvern 根文档](../CLAUDE.md) | [Forge 模块](../forge/CLAUDE.md) | [Schemas 模块](../schemas/CLAUDE.md)