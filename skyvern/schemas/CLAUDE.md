# Schemas 模块文档

> 📍 **位置**: [skyvern](../CLAUDE.md) → schemas

## 模块概述

Schemas 模块定义了 Skyvern 系统中使用的所有数据模型和 Pydantic 模式。该模块提供了完整的数据验证、序列化和反序列化功能，确保 API 请求/响应的数据一致性和类型安全。

## 文件结构

```
skyvern/schemas/
├── __init__.py
├── agent_schemas.py        # 代理相关模式
├── artifact_schemas.py     # 工件相关模式
├── auth_schemas.py         # 认证相关模式
├── browser_schemas.py      # 浏览器相关模式
├── credential_schemas.py   # 凭据相关模式
├── data_source_schemas.py  # 数据源相关模式
├── enums.py               # 枚举定义
├── organization_schemas.py # 组织相关模式
├── proxy_schemas.py       # 代理相关模式
├── response_schemas.py    # 响应相关模式
├── run_schemas.py         # 执行记录相关模式
├── server_schemas.py      # 服务器相关模式
├── task_schemas.py        # 任务相关模式
├── user_schemas.py        # 用户相关模式
├── webhook_schemas.py     # Webhook 相关模式
├── workflow_schemas.py    # 工作流相关模式
├── workflow_v2_schemas.py # 工作流 v2 相关模式
└── common_schemas.py      # 通用模式定义
```

## 核心模式定义

### 1. 通用模式 (`common_schemas.py`)
```python
from pydantic import BaseModel, Field, validator
from typing import Optional, Dict, Any, List
from datetime import datetime
from uuid import UUID

class BaseSchema(BaseModel):
    """基础模式类"""
    class Config:
        allow_population_by_field_name = True
        use_enum_values = True

class TimestampedSchema(BaseSchema):
    """带时间戳的基础模式"""
    created_at: datetime
    updated_at: datetime

class IdentifiableSchema(BaseSchema):
    """可识别的基础模式"""
    id: UUID = Field(..., description="唯一标识符")

class PaginatedResponse(BaseSchema):
    """分页响应模式"""
    items: List[Any]
    total: int
    page: int
    size: int
    pages: int

class ErrorResponse(BaseSchema):
    """错误响应模式"""
    error: str
    message: str
    details: Optional[Dict[str, Any]] = None

class SuccessResponse(BaseSchema):
    """成功响应模式"""
    success: bool = True
    message: Optional[str] = None
    data: Optional[Any] = None
```

### 2. 任务模式 (`task_schemas.py`)
```python
from typing import Optional, List, Dict, Any
from pydantic import Field, validator

class TaskCreate(BaseSchema):
    """创建任务请求"""
    title: str = Field(..., min_length=1, max_length=255, description="任务标题")
    url: str = Field(..., description="目标URL")
    goal: str = Field(..., description="任务目标描述")
    webhook_callback_url: Optional[str] = Field(None, description="Webhook回调URL")
    proxy: Optional[ProxySettings] = Field(None, description="代理设置")
    proxy_location: Optional[str] = Field(None, description="代理位置")
    org_id: Optional[str] = Field(None, description="组织ID")

    @validator('url')
    def validate_url(cls, v):
        """验证URL格式"""
        if not v.startswith(('http://', 'https://')):
            raise ValueError('URL must start with http:// or https://')
        return v

class TaskResponse(TimestampedSchema, IdentifiableSchema):
    """任务响应"""
    task_id: str
    user_id: str
    organization_id: Optional[str]
    title: str
    url: str
    goal: str
    status: TaskStatus
    webhook_callback_url: Optional[str]
    proxy_location: Optional[str]
    max_steps_per_run: Optional[int]
    error_message: Optional[str]
    extracted_information: Optional[Dict[str, Any]]
    retry_count: int = 0
    last_run_id: Optional[str]

class TaskExecution(BaseSchema):
    """任务执行信息"""
    execution_id: str
    task_id: str
    status: TaskStatus
    prompt_adjustment: Optional[str]
    started_at: datetime
    completed_at: Optional[datetime]
    error_message: Optional[str]
    result: Optional[Dict[str, Any]]
    steps: List[StepResponse]
```

### 3. 工作流模式 (`workflow_schemas.py`)
```python
from typing import Optional, List, Dict, Any, Union
from pydantic import Field, validator

class WorkflowBlockBase(BaseSchema):
    """工作流块基础模式"""
    block_type: BlockType
    label: Optional[str] = None
    description: Optional[str] = None
    data: Dict[str, Any] = Field(default_factory=dict)
    parameters: Dict[str, Any] = Field(default_factory=dict)
    output_schema: Optional[Dict[str, Any]] = None
    condition: Optional[str] = None
    continue_on_error: bool = False

class WorkflowBlockCreate(WorkflowBlockBase):
    """创建工作流块"""
    pass

class WorkflowBlockResponse(WorkflowBlockBase, IdentifiableSchema, TimestampedSchema):
    """工作流块响应"""
    workflow_id: str
    order: int

class WorkflowCreate(BaseSchema):
    """创建工作流"""
    title: str = Field(..., min_length=1, max_length=255)
    description: Optional[str] = Field(None, max_length=1000)
    blocks: List[WorkflowBlockCreate]
    workflow_definition_id: Optional[str]
    proxy_location: Optional[str]
    is_locked: bool = False
    is_public: bool = False
    organization_id: Optional[str]

class WorkflowResponse(TimestampedSchema, IdentifiableSchema):
    """工作流响应"""
    workflow_id: str
    user_id: str
    organization_id: Optional[str]
    title: str
    description: Optional[str]
    status: WorkflowStatus
    blocks: List[WorkflowBlockResponse]
    proxy_location: Optional[str]
    is_locked: bool
    is_public: bool
    last_run_id: Optional[str]

class WorkflowRunCreate(BaseSchema):
    """创建工作流运行"""
    workflow_id: str
    data: Dict[str, Any] = Field(default_factory=dict)
    proxy_location: Optional[str]
    max_iterations: Optional[int] = Field(None, gt=0)
    webhook_callback_url: Optional[str]

class WorkflowRunResponse(TimestampedSchema, IdentifiableSchema):
    """工作流运行响应"""
    run_id: str
    workflow_id: str
    user_id: str
    organization_id: Optional[str]
    status: WorkflowRunStatus
    data: Dict[str, Any]
    output: Optional[Dict[str, Any]]
    proxy_location: Optional[str]
    webhook_callback_url: Optional[str]
    error_message: Optional[str]
    modified_blocks: List[Dict[str, Any]]
    terminated_reason: Optional[str]
    max_iterations: Optional[int]
    current_iteration: int
```

### 4. 浏览器会话模式 (`browser_schemas.py`)
```python
from typing import Optional, Dict, Any, List

class BrowserSessionCreate(BaseSchema):
    """创建浏览器会话"""
    title: str = Field(..., min_length=1, max_length=255)
    url: Optional[str] = None
    proxy_location: Optional[str]
    user_data_dir: Optional[str]
    organization_id: Optional[str]

class BrowserSessionResponse(TimestampedSchema, IdentifiableSchema):
    """浏览器会话响应"""
    session_id: str
    user_id: str
    organization_id: Optional[str]
    title: str
    url: Optional[str]
    status: SessionStatus
    proxy_location: Optional[str]
    user_data_dir: Optional[str]
    last_activity_at: Optional[datetime]

class BrowserSessionAction(BaseSchema):
    """浏览器会话动作"""
    action_type: ActionType
    parameters: Dict[str, Any]

class BrowserSessionActionResponse(BaseSchema):
    """浏览器会话动作响应"""
    success: bool
    message: Optional[str]
    data: Optional[Dict[str, Any]]
    screenshot_url: Optional[str]
    elements_found: Optional[List[ElementInfo]]
```

### 5. 凭据模式 (`credential_schemas.py`)
```python
from typing import Dict, Any, Optional

class CredentialCreate(BaseSchema):
    """创建凭据"""
    credential_type: CredentialType
    name: str = Field(..., min_length=1, max_length=255)
    data: Dict[str, Any]
    organization_id: Optional[str]

    @validator('data')
    def validate_credential_data(cls, v, values):
        """验证凭据数据"""
        credential_type = values.get('credential_type')

        if credential_type == CredentialType.BASIC_AUTH:
            required_fields = ['username', 'password']
        elif credential_type == CredentialType.API_KEY:
            required_fields = ['api_key']
        elif credential_type == CredentialType.BEARER_TOKEN:
            required_fields = ['token']
        elif credential_type == CredentialType.CUSTOM:
            required_fields = []
        else:
            required_fields = []

        for field in required_fields:
            if field not in v:
                raise ValueError(f"Missing required field: {field}")

        return v

class CredentialResponse(TimestampedSchema, IdentifiableSchema):
    """凭据响应"""
    credential_id: str
    user_id: str
    organization_id: Optional[str]
    credential_type: CredentialType
    name: str
    # 注意：不返回加密的敏感数据
    masked_data: Dict[str, Any]
    domain: Optional[str]
    auto_fill_enabled: bool

    def __init__(self, **data):
        """初始化时屏蔽敏感数据"""
        if 'encrypted_data' in data:
            # 屏蔽敏感数据
            encrypted_data = data.pop('encrypted_data')
            data['masked_data'] = self._mask_sensitive_data(encrypted_data)
        super().__init__(**data)

    @staticmethod
    def _mask_sensitive_data(data: Dict[str, Any]) -> Dict[str, Any]:
        """屏蔽敏感数据"""
        sensitive_fields = ['password', 'token', 'api_key', 'secret']
        masked = {}

        for key, value in data.items():
            if any(field in key.lower() for field in sensitive_fields):
                masked[key] = '*' * 8 + (str(value)[-4:] if value else '')
            else:
                masked[key] = value

        return masked
```

### 6. Webhook 模式 (`webhook_schemas.py`)
```python
from typing import List, Dict, Any, Optional

class WebhookCreate(BaseSchema):
    """创建 Webhook"""
    url: str = Field(..., description="Webhook URL")
    events: List[WebhookEvent] = Field(..., description="订阅的事件")
    secret: Optional[str] = Field(None, description="签名密钥")
    active: bool = Field(True, description="是否激活")
    organization_id: Optional[str]

    @validator('url')
    def validate_webhook_url(cls, v):
        """验证 Webhook URL"""
        if not v.startswith(('http://', 'https://')):
            raise ValueError('Webhook URL must start with http:// or https://')
        return v

class WebhookResponse(TimestampedSchema, IdentifiableSchema):
    """Webhook 响应"""
    webhook_id: str
    user_id: str
    organization_id: Optional[str]
    url: str
    events: List[WebhookEvent]
    secret: str
    active: bool
    last_triggered_at: Optional[datetime]
    trigger_count: int = 0

class WebhookEventPayload(BaseSchema):
    """Webhook 事件负载"""
    event: WebhookEvent
    timestamp: datetime
    data: Dict[str, Any]

class WebhookTriggerResult(BaseSchema):
    """Webhook 触发结果"""
    webhook_id: str
    event: WebhookEvent
    success: bool
    status_code: Optional[int]
    response_time_ms: Optional[int]
    error_message: Optional[str]
```

### 7. 用户模式 (`user_schemas.py`)
```python
from typing import Optional

class UserCreate(BaseSchema):
    """创建用户"""
    email: str = Field(..., regex=r'^[^@]+@[^@]+\.[^@]+$')
    password: str = Field(..., min_length=8, max_length=128)
    first_name: Optional[str] = Field(None, max_length=100)
    last_name: Optional[str] = Field(None, max_length=100)
    organization_name: Optional[str] = Field(None, max_length=255)

    @validator('password')
    def validate_password(cls, v):
        """验证密码强度"""
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain at least one uppercase letter')
        if not any(c.islower() for c in v):
            raise ValueError('Password must contain at least one lowercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain at least one digit')
        return v

class UserResponse(TimestampedSchema, IdentifiableSchema):
    """用户响应"""
    user_id: str
    email: str
    first_name: Optional[str]
    last_name: Optional[str]
    is_active: bool
    is_superuser: bool
    last_login_at: Optional[datetime]
    organization_id: Optional[str]

class UserUpdate(BaseSchema):
    """更新用户"""
    first_name: Optional[str] = Field(None, max_length=100)
    last_name: Optional[str] = Field(None, max_length=100)
    is_active: Optional[bool]

class UserLogin(BaseSchema):
    """用户登录"""
    email: str
    password: str

class UserLoginResponse(BaseSchema):
    """登录响应"""
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    user: UserResponse
```

### 8. 组织模式 (`organization_schemas.py`)
```python
from typing import Optional, List

class OrganizationCreate(BaseSchema):
    """创建组织"""
    name: str = Field(..., min_length=1, max_length=255)
    description: Optional[str] = Field(None, max_length=1000)
    domain: Optional[str] = Field(None, max_length=255)

class OrganizationResponse(TimestampedSchema, IdentifiableSchema):
    """组织响应"""
    organization_id: str
    name: str
    description: Optional[str]
    domain: Optional[str]
    is_active: bool
    owner_id: str
    member_count: int
    max_members: int
    subscription_plan: str

class OrganizationMember(BaseSchema):
    """组织成员"""
    user_id: str
    role: OrganizationRole
    joined_at: datetime

class OrganizationUpdate(BaseSchema):
    """更新组织"""
    name: Optional[str] = Field(None, min_length=1, max_length=255)
    description: Optional[str] = Field(None, max_length=1000)
    domain: Optional[str] = Field(None, max_length=255)
    is_active: Optional[bool]
```

## 枚举定义 (`enums.py`)

```python
from enum import Enum

class TaskStatus(str, Enum):
    """任务状态"""
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELLED = "cancelled"
    TIMEOUT = "timeout"

class WorkflowStatus(str, Enum):
    """工作流状态"""
    DRAFT = "draft"
    ACTIVE = "active"
    PAUSED = "paused"
    ARCHIVED = "archived"

class WorkflowRunStatus(str, Enum):
    """工作流运行状态"""
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELLED = "cancelled"
    TERMINATED = "terminated"

class BlockType(str, Enum):
    """工作流块类型"""
    NAVIGATION = "navigation"
    EXTRACTION = "extraction"
    VALIDATION = "validation"
    LOOP = "loop"
    CONDITIONAL = "conditional"
    CODE = "code"
    WAIT = "wait"
    UPLOAD = "upload"
    DOWNLOAD = "download"
    EMAIL = "email"
    WEBHOOK = "webhook"
    TERMINATE = "terminate"

class ActionType(str, Enum):
    """浏览器动作类型"""
    CLICK = "click"
    INPUT = "input"
    SELECT = "select"
    SCROLL = "scroll"
    NAVIGATE = "navigate"
    WAIT = "wait"
    UPLOAD = "upload"
    DOWNLOAD = "download"
    SCREENSHOT = "screenshot"
    EXTRACT = "extract"

class SessionStatus(str, Enum):
    """浏览器会话状态"""
    ACTIVE = "active"
    IDLE = "idle"
    CLOSED = "closed"
    ERROR = "error"

class CredentialType(str, Enum):
    """凭据类型"""
    BASIC_AUTH = "basic_auth"
    API_KEY = "api_key"
    BEARER_TOKEN = "bearer_token"
    OAUTH2 = "oauth2"
    CUSTOM = "custom"

class WebhookEvent(str, Enum):
    """Webhook 事件"""
    TASK_CREATED = "task.created"
    TASK_STARTED = "task.started"
    TASK_COMPLETED = "task.completed"
    TASK_FAILED = "task.failed"
    WORKFLOW_RUN_STARTED = "workflow_run.started"
    WORKFLOW_RUN_COMPLETED = "workflow_run.completed"
    WORKFLOW_RUN_FAILED = "workflow_run.failed"

class OrganizationRole(str, Enum):
    """组织角色"""
    OWNER = "owner"
    ADMIN = "admin"
    MEMBER = "member"
    VIEWER = "viewer"
```

## 数据验证与转换

### 1. 自定义验证器
```python
from pydantic import validator, root_validator

class ComplexSchema(BaseSchema):
    """复杂模式示例"""
    start_date: datetime
    end_date: datetime
    budget: float
    currency: str

    @validator('end_date')
    def end_date_after_start_date(cls, v, values):
        """确保结束日期晚于开始日期"""
        if 'start_date' in values and v <= values['start_date']:
            raise ValueError('end_date must be after start_date')
        return v

    @validator('currency')
    def validate_currency(cls, v):
        """验证货币代码"""
        valid_currencies = ['USD', 'EUR', 'GBP', 'JPY', 'CNY']
        if v.upper() not in valid_currencies:
            raise ValueError(f'currency must be one of {valid_currencies}')
        return v.upper()

    @root_validator
    def validate_budget(cls, values):
        """验证预算与货币的一致性"""
        if values.get('budget', 0) < 0:
            raise ValueError('budget must be non-negative')
        return values
```

### 2. 数据转换器
```python
from pydantic import Field, validator

class DataTransformSchema(BaseSchema):
    """数据转换示例"""
    tags: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None

    @validator('tags', pre=True)
    def transform_tags(cls, v):
        """将标签字符串转换为列表"""
        if isinstance(v, str):
            return [tag.strip() for tag in v.split(',') if tag.strip()]
        return v

    @validator('metadata', pre=True)
    def transform_metadata(cls, v):
        """将JSON字符串转换为字典"""
        if isinstance(v, str):
            try:
                return json.loads(v)
            except json.JSONDecodeError:
                raise ValueError('metadata must be valid JSON')
        return v
```

## API 响应模式 (`response_schemas.py`)

```python
from typing import Generic, TypeVar, Optional, List
from pydantic import Field

T = TypeVar('T')

class APIResponse(BaseSchema, Generic[T]):
    """通用API响应"""
    success: bool = True
    message: Optional[str] = None
    data: Optional[T] = None
    errors: Optional[List[str]] = None

class ListResponse(BaseSchema, Generic[T]):
    """列表响应"""
    items: List[T]
    total: int = Field(..., ge=0)
    page: int = Field(..., ge=1)
    size: int = Field(..., ge=1)
    pages: int = Field(..., ge=0)

    @classmethod
    def create(
        cls,
        items: List[T],
        total: int,
        page: int,
        size: int
    ) -> "ListResponse[T]":
        """创建分页响应"""
        pages = (total + size - 1) // size
        return cls(
            items=items,
            total=total,
            page=page,
            size=size,
            pages=pages
        )

class DetailResponse(BaseSchema):
    """详情响应"""
    id: str
    created_at: datetime
    updated_at: datetime
    created_by: Optional[str]
    updated_by: Optional[str]

class BulkOperationResponse(BaseSchema):
    """批量操作响应"""
    total: int
    success: int
    failed: int
    errors: List[Dict[str, Any]] = Field(default_factory=list)
```

## 使用最佳实践

### 1. 模式设计原则
- **明确性**: 每个字段都应该有清晰的描述
- **验证性**: 使用 validators 确保数据有效性
- **可扩展性**: 设计时要考虑未来的扩展
- **类型安全**: 使用强类型定义

### 2. 性能优化
- 使用 `orm_mode = True` 支持ORM模型转换
- 合理使用 `Optional` 字段
- 避免深度嵌套的响应结构
- 实现响应数据缓存

### 3. 安全考虑
- 敏感数据不在响应中返回
- 使用输入验证防止注入攻击
- 实现数据脱敏机制
- 验证用户权限

### 4. 文档生成
```python
from pydantic import Field

class ExampleSchema(BaseSchema):
    """带有详细文档的示例"""
    name: str = Field(
        ...,
        description="用户名称",
        example="John Doe",
        min_length=1,
        max_length=100
    )
    age: int = Field(
        ...,
        description="用户年龄",
        example=30,
        ge=0,
        le=150
    )
```

---

> 📖 **返回**: [Skyvern 根文档](../CLAUDE.md) | [Services 模块](../services/CLAUDE.md) | [CLI 模块](../cli/CLAUDE.md)