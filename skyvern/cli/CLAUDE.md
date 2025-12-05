# CLI 模块文档

> 📍 **位置**: [skyvern](../CLAUDE.md) → cli

## 模块概述

CLI 模块是 Skyvern 的命令行工具，提供了完整的命令行界面用于管理 Skyvern 服务、执行任务、配置系统等。它是开发者与 Skyvern 交互的主要工具之一。

## 文件结构

```
skyvern/cli/
├── __init__.py
├── main.py                 # CLI 主入口
├── commands.py             # 命令定义
├── config.py               # 配置管理
├── utils.py                # 工具函数
├── api_client.py           # API 客户端
├── auth.py                 # 认证管理
├── output.py               # 输出格式化
├── exceptions.py           # CLI 异常
├── __main__.py             # 包执行入口
└── commands/               # 命令实现
    ├── __init__.py
    ├── agent.py            # 代理相关命令
    ├── auth.py             # 认证相关命令
    ├── browser.py          # 浏览器相关命令
    ├── config.py           # 配置相关命令
    ├── credential.py       # 凭据相关命令
    ├── init.py             # 初始化命令
    ├── organization.py     # 组织相关命令
    ├── quickstart.py       # 快速启动命令
    ├── run.py              # 运行相关命令
    ├── server.py           # 服务器相关命令
    ├── status.py           # 状态相关命令
    ├── task.py             # 任务相关命令
    ├── user.py             # 用户相关命令
    ├── workflow.py         # 工作流相关命令
    └── webhook.py          # Webhook 相关命令
```

## 核心组件

### 1. CLI 主入口 (`main.py`)
```python
import click
from rich.console import Console
from rich.table import Table
from typing import Optional

console = Console()

@click.group()
@click.option('--config', '-c', type=click.Path(), help='配置文件路径')
@click.option('--verbose', '-v', is_flag=True, help='详细输出')
@click.option('--format', type=click.Choice(['json', 'table', 'yaml']), default='table', help='输出格式')
@click.pass_context
def cli(ctx, config, verbose, format):
    """Skyvern CLI - AI 浏览器自动化平台"""
    ctx.ensure_object(dict)
    ctx.obj['config'] = config
    ctx.obj['verbose'] = verbose
    ctx.obj['format'] = format

    # 加载配置
    config_manager = ConfigManager(config_path=config)
    ctx.obj['config_manager'] = config_manager

    # 初始化 API 客户端
    api_client = APIClient(
        base_url=config_manager.get('api.base_url', 'http://localhost:8000'),
        api_key=config_manager.get('api.key'),
        timeout=config_manager.get('api.timeout', 30)
    )
    ctx.obj['api_client'] = api_client

# 注册命令组
cli.add_command(commands.agent_commands)
cli.add_command(commands.auth_commands)
cli.add_command(commands.browser_commands)
cli.add_command(commands.config_commands)
cli.add_command(commands.credential_commands)
cli.add_command(commands.init_commands)
cli.add_command(commands.organization_commands)
cli.add_command(commands.quickstart_commands)
cli.add_command(commands.run_commands)
cli.add_command(commands.server_commands)
cli.add_command(commands.status_commands)
cli.add_command(commands.task_commands)
cli.add_command(commands.user_commands)
cli.add_command(commands.workflow_commands)
cli.add_command(commands.webhook_commands)

if __name__ == '__main__':
    cli()
```

### 2. 命令定义 (`commands.py`)
```python
from click import group
from typing import Optional

# 命令组定义
@group()
def agent():
    """代理相关命令"""
    pass

@group()
def auth():
    """认证相关命令"""
    pass

@group()
def browser():
    """浏览器相关命令"""
    pass

@group()
def config():
    """配置相关命令"""
    pass

@group()
def credential():
    """凭据相关命令"""
    pass

@group()
def init():
    """初始化相关命令"""
    pass

@group()
def organization():
    """组织相关命令"""
    pass

@group()
def quickstart():
    """快速启动命令"""
    pass

@group()
def run():
    """运行相关命令"""
    pass

@group()
def server():
    """服务器相关命令"""
    pass

@group()
def status():
    """状态相关命令"""
    pass

@group()
def task():
    """任务相关命令"""
    pass

@group()
def user():
    """用户相关命令"""
    pass

@group()
def workflow():
    """工作流相关命令"""
    pass

@group()
def webhook():
    """Webhook 相关命令"""
    pass
```

### 3. 配置管理 (`config.py`)
```python
import os
import yaml
import json
from typing import Any, Dict, Optional
from pathlib import Path

class ConfigManager:
    """配置管理器"""

    DEFAULT_CONFIG = {
        'api': {
            'base_url': 'http://localhost:8000',
            'timeout': 30,
            'verify_ssl': True
        },
        'auth': {
            'token_file': '~/.skyvern/token',
            'refresh_token_file': '~/.skyvern/refresh_token'
        },
        'browser': {
            'headless': True,
            'timeout': 30000,
            'user_data_dir': '~/.skyvern/browser_data'
        },
        'logging': {
            'level': 'INFO',
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        },
        'output': {
            'format': 'table',
            'pager': True
        }
    }

    def __init__(self, config_path: Optional[str] = None):
        self.config_path = Path(config_path) if config_path else self._get_default_config_path()
        self._config = self._load_config()

    def _get_default_config_path(self) -> Path:
        """获取默认配置文件路径"""
        return Path.home() / '.skyvern' / 'config.yaml'

    def _load_config(self) -> Dict[str, Any]:
        """加载配置"""
        config = self.DEFAULT_CONFIG.copy()

        if self.config_path.exists():
            with open(self.config_path, 'r') as f:
                user_config = yaml.safe_load(f) or {}
                config.update(user_config)

        # 从环境变量加载配置
        self._load_from_env(config)

        return config

    def _load_from_env(self, config: Dict[str, Any]) -> None:
        """从环境变量加载配置"""
        env_mappings = {
            'SKYVERN_API_BASE_URL': ('api', 'base_url'),
            'SKYVERN_API_KEY': ('api', 'key'),
            'SKYVERN_BROWSER_HEADLESS': ('browser', 'headless'),
            'SKYVERN_LOG_LEVEL': ('logging', 'level'),
        }

        for env_var, (section, key) in env_mappings.items():
            value = os.getenv(env_var)
            if value is not None:
                # 类型转换
                if key == 'headless':
                    value = value.lower() in ('true', '1', 'yes')
                elif key == 'timeout':
                    value = int(value)

                config[section][key] = value

    def get(self, key: str, default: Any = None) -> Any:
        """获取配置值"""
        sections = key.split('.')
        value = self._config

        for section in sections:
            if isinstance(value, dict) and section in value:
                value = value[section]
            else:
                return default

        return value

    def set(self, key: str, value: Any) -> None:
        """设置配置值"""
        sections = key.split('.')
        config = self._config

        for section in sections[:-1]:
            if section not in config:
                config[section] = {}
            config = config[section]

        config[sections[-1]] = value
        self._save_config()

    def _save_config(self) -> None:
        """保存配置到文件"""
        self.config_path.parent.mkdir(parents=True, exist_ok=True)
        with open(self.config_path, 'w') as f:
            yaml.dump(self._config, f, default_flow_style=False)
```

### 4. API 客户端 (`api_client.py`)
```python
import aiohttp
import asyncio
from typing import Any, Dict, Optional, Union
import json
from datetime import datetime

class APIClient:
    """API 客户端"""

    def __init__(
        self,
        base_url: str,
        api_key: Optional[str] = None,
        timeout: int = 30,
        verify_ssl: bool = True
    ):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.timeout = aiohttp.ClientTimeout(total=timeout)
        self.verify_ssl = verify_ssl
        self._session = None

    async def _get_session(self) -> aiohttp.ClientSession:
        """获取或创建会话"""
        if self._session is None or self._session.closed:
            headers = {}
            if self.api_key:
                headers['Authorization'] = f'Bearer {self.api_key}'

            self._session = aiohttp.ClientSession(
                base_url=self.base_url,
                headers=headers,
                timeout=self.timeout,
                connector=aiohttp.TCPConnector(verify_ssl=self.verify_ssl)
            )
        return self._session

    async def request(
        self,
        method: str,
        endpoint: str,
        data: Optional[Dict[str, Any]] = None,
        params: Optional[Dict[str, Any]] = None,
        files: Optional[Dict[str, Any]] = None
    ) -> Dict[str, Any]:
        """发送请求"""
        session = await self._get_session()

        url = f"{self.base_url}{endpoint}"

        kwargs = {
            'params': params,
        }

        if files:
            # 文件上传
            data = aiohttp.FormData()
            for key, value in files.items():
                if isinstance(value, tuple):
                    data.add_field(key, value[1], filename=value[0])
                else:
                    data.add_field(key, value)
            kwargs['data'] = data
        elif data:
            kwargs['json'] = data

        async with session.request(method, url, **kwargs) as response:
            response_data = await response.json()

            if response.status >= 400:
                raise APIError(
                    status_code=response.status,
                    message=response_data.get('error', {}).get('message', 'Unknown error'),
                    details=response_data.get('error', {}).get('details')
                )

            return response_data

    async def get(self, endpoint: str, params: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        """GET 请求"""
        return await self.request('GET', endpoint, params=params)

    async def post(self, endpoint: str, data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        """POST 请求"""
        return await self.request('POST', endpoint, data=data)

    async def put(self, endpoint: str, data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        """PUT 请求"""
        return await self.request('PUT', endpoint, data=data)

    async def delete(self, endpoint: str) -> Dict[str, Any]:
        """DELETE 请求"""
        return await self.request('DELETE', endpoint)

    async def close(self):
        """关闭会话"""
        if self._session and not self._session.closed:
            await self._session.close()

class APIError(Exception):
    """API 错误"""

    def __init__(self, status_code: int, message: str, details: Optional[Dict] = None):
        self.status_code = status_code
        self.message = message
        self.details = details
        super().__init__(f"API Error {status_code}: {message}")
```

## 命令实现详解

### 1. 服务器命令 (`commands/server.py`)
```python
import click
import asyncio
import subprocess
from rich.console import Console
from rich.progress import Progress, SpinnerColumn, TextColumn

console = Console()

@server.command()
@click.option('--mode', type=click.Choice(['all', 'server', 'ui']), default='all', help='启动模式')
@click.option('--host', default='0.0.0.0', help='服务器主机')
@click.option('--port', default=8000, help='服务器端口')
@click.option('--reload', is_flag=True, help='开发模式热重载')
@click.pass_context
def start(ctx, mode, host, port, reload):
    """启动 Skyvern 服务"""
    if mode == 'all':
        # 启动所有服务
        asyncio.run(_start_all_services(ctx, host, port, reload))
    elif mode == 'server':
        # 只启动后端服务
        asyncio.run(_start_server(ctx, host, port, reload))
    elif mode == 'ui':
        # 只启动前端服务
        asyncio.run(_start_ui(ctx))

@server.command()
@click.pass_context
def stop(ctx):
    """停止 Skyvern 服务"""
    asyncio.run(_stop_services(ctx))

@server.command()
@click.pass_context
def restart(ctx):
    """重启 Skyvern 服务"""
    asyncio.run(_restart_services(ctx))

async def _start_all_services(ctx, host, port, reload):
    """启动所有服务"""
    with Progress(
        SpinnerColumn(),
        TextColumn("[progress.description]{task.description}"),
        console=console
    ) as progress:

        # 启动后端
        task1 = progress.add_task("启动后端服务...", total=None)
        server_process = await _start_backend_server(host, port, reload)
        progress.update(task1, description="✓ 后端服务已启动")

        # 等待服务就绪
        task2 = progress.add_task("等待服务就绪...", total=None)
        await _wait_for_server_ready(f"http://{host}:{port}")
        progress.update(task2, description="✓ 服务已就绪")

        # 启动前端
        task3 = progress.add_task("启动前端服务...", total=None)
        ui_process = await _start_frontend_ui()
        progress.update(task3, description="✓ 前端服务已启动")

        console.print("\n[green]✓ Skyvern 服务已成功启动！[/green]")
        console.print(f"后端 API: http://{host}:{port}")
        console.print("前端界面: http://localhost:3000")

        # 保持服务运行
        try:
            while True:
                await asyncio.sleep(1)
        except KeyboardInterrupt:
            console.print("\n[yellow]正在停止服务...[/yellow]")
            await _stop_processes([server_process, ui_process])
```

### 2. 任务命令 (`commands/task.py`)
```python
import click
import json
from rich.console import Console
from rich.table import Table
from rich.prompt import Prompt, Confirm
from rich.syntax import Syntax

console = Console()

@task.command()
@click.option('--title', required=True, help='任务标题')
@click.option('--url', required=True, help='目标URL')
@click.option('--goal', required=True, help='任务目标')
@click.option('--webhook', help='Webhook回调URL')
@click.option('--proxy', help='代理设置')
@click.pass_context
def create(ctx, title, url, goal, webhook, proxy):
    """创建新任务"""
    api_client = ctx.obj['api_client']

    task_data = {
        'title': title,
        'url': url,
        'goal': goal,
        'webhook_callback_url': webhook,
        'proxy': json.loads(proxy) if proxy else None
    }

    try:
        # 创建任务
        task = asyncio.run(api_client.post('/api/v1/tasks', data=task_data))
        console.print(f"[green]✓ 任务创建成功[/green]")
        console.print(f"任务ID: {task['task_id']}")

        # 询问是否立即执行
        if Confirm.ask("是否立即执行该任务？"):
            execute_task(ctx, task['task_id'])

    except APIError as e:
        console.print(f"[red]✗ 创建任务失败: {e.message}[/red]")

@task.command()
@click.argument('task_id')
@click.pass_context
def execute(ctx, task_id):
    """执行任务"""
    api_client = ctx.obj['api_client']

    try:
        # 执行任务
        execution = asyncio.run(api_client.post(f'/api/v1/tasks/{task_id}/execute'))
        execution_id = execution['execution_id']

        console.print(f"[green]✓ 任务已开始执行[/green]")
        console.print(f"执行ID: {execution_id}")

        # 实时显示执行状态
        stream_task_status(ctx, task_id, execution_id)

    except APIError as e:
        console.print(f"[red]✗ 执行任务失败: {e.message}[/red]")

@task.command()
@click.option('--status', help='按状态过滤')
@click.option('--limit', default=10, help='显示数量')
@click.pass_context
def list(ctx, status, limit):
    """列出任务"""
    api_client = ctx.obj['api_client']

    params = {'limit': limit}
    if status:
        params['status'] = status

    try:
        response = asyncio.run(api_client.get('/api/v1/tasks', params=params))
        tasks = response.get('tasks', [])

        if not tasks:
            console.print("[yellow]没有找到任务[/yellow]")
            return

        # 创建表格
        table = Table(title="任务列表")
        table.add_column("任务ID", style="cyan")
        table.add_column("标题", style="white")
        table.add_column("状态", style="green")
        table.add_column("创建时间", style="blue")
        table.add_column("目标URL", style="yellow")

        for task in tasks:
            status_color = {
                'pending': 'yellow',
                'running': 'blue',
                'completed': 'green',
                'failed': 'red'
            }.get(task['status'], 'white')

            table.add_row(
                task['task_id'][:8],
                task['title'][:30],
                f"[{status_color}]{task['status']}[/{status_color}]",
                task['created_at'][:19],
                task['url'][:40]
            )

        console.print(table)

    except APIError as e:
        console.print(f"[red]✗ 获取任务列表失败: {e.message}[/red]")

@task.command()
@click.argument('task_id')
@click.pass_context
def show(ctx, task_id):
    """显示任务详情"""
    api_client = ctx.obj['api_client']

    try:
        task = asyncio.run(api_client.get(f'/api/v1/tasks/{task_id}'))

        # 显示任务信息
        console.print(f"[bold]任务详情[/bold]")
        console.print(f"ID: {task['task_id']}")
        console.print(f"标题: {task['title']}")
        console.print(f"URL: {task['url']}")
        console.print(f"目标: {task['goal']}")
        console.print(f"状态: {task['status']}")
        console.print(f"创建时间: {task['created_at']}")

        # 显示执行历史
        if task.get('runs'):
            console.print("\n[bold]执行历史[/bold]")
            for run in task['runs'][-5:]:  # 显示最近5次执行
                console.print(f"- {run['execution_id']}: {run['status']} ({run['created_at']})")

    except APIError as e:
        console.print(f"[red]✗ 获取任务详情失败: {e.message}[/red]")

async def stream_task_status(ctx, task_id, execution_id):
    """流式显示任务状态"""
    api_client = ctx.obj['api_client']

    try:
        last_status = None

        while True:
            execution = await api_client.get(f'/api/v1/tasks/{task_id}/executions/{execution_id}')
            current_status = execution['status']

            if current_status != last_status:
                if current_status == 'running':
                    console.print(f"[blue]⚡ 任务执行中...[/blue]")
                elif current_status == 'completed':
                    console.print(f"[green]✓ 任务执行完成[/green]")
                    if execution.get('result'):
                        console.print(f"结果: {execution['result']}")
                    break
                elif current_status == 'failed':
                    console.print(f"[red]✗ 任务执行失败[/red]")
                    if execution.get('error_message'):
                        console.print(f"错误: {execution['error_message']}")
                    break

                last_status = current_status

            await asyncio.sleep(2)

    except APIError:
        console.print("[red]✗ 无法获取任务状态[/red]")
```

### 3. 工作流命令 (`commands/workflow.py`)
```python
import click
import json
import yaml
from pathlib import Path
from rich.console import Console
from rich.tree import Tree
from rich.panel import Panel

console = Console()

@workflow.command()
@click.option('--file', '-f', type=click.Path(exists=True), required=True, help='工作流定义文件')
@click.pass_context
def create(ctx, file):
    """创建工作流"""
    api_client = ctx.obj['api_client']

    # 读取工作流定义
    file_path = Path(file)
    with open(file_path, 'r') as f:
        if file_path.suffix.lower() == '.yaml':
            workflow_def = yaml.safe_load(f)
        else:
            workflow_def = json.load(f)

    try:
        # 创建工作流
        workflow = asyncio.run(api_client.post('/api/v1/workflows', data=workflow_def))
        console.print(f"[green]✓ 工作流创建成功[/green]")
        console.print(f"工作流ID: {workflow['workflow_id']}")

        # 显示工作流结构
        display_workflow_structure(workflow)

    except APIError as e:
        console.print(f"[red]✗ 创建工作流失败: {e.message}[/red]")

@workflow.command()
@click.argument('workflow_id')
@click.option('--data', help='输入数据 (JSON格式)')
@click.option('--proxy', help='代理位置')
@click.pass_context
def run(ctx, workflow_id, data, proxy):
    """运行工作流"""
    api_client = ctx.obj['api_client']

    # 解析输入数据
    input_data = json.loads(data) if data else {}

    run_data = {
        'data': input_data,
        'proxy_location': proxy
    }

    try:
        # 运行工作流
        run = asyncio.run(api_client.post(f'/api/v1/workflows/{workflow_id}/run', data=run_data))
        console.print(f"[green]✓ 工作流已开始运行[/green]")
        console.print(f"运行ID: {run['run_id']}")

        # 实时显示运行状态
        stream_workflow_run_status(ctx, workflow_id, run['run_id'])

    except APIError as e:
        console.print(f"[red]✗ 运行工作流失败: {e.message}[/red]")

@workflow.command()
@click.argument('workflow_id')
@click.pass_context
def show(ctx, workflow_id):
    """显示工作流详情"""
    api_client = ctx.obj['api_client']

    try:
        workflow = asyncio.run(api_client.get(f'/api/v1/workflows/{workflow_id}'))

        # 显示工作流信息
        console.print(Panel.fit(
            f"工作流: {workflow['title']}\n"
            f"ID: {workflow['workflow_id']}\n"
            f"状态: {workflow['status']}\n"
            f"描述: {workflow.get('description', 'N/A')}",
            title="工作流信息"
        ))

        # 显示工作流结构
        display_workflow_structure(workflow)

    except APIError as e:
        console.print(f"[red]✗ 获取工作流详情失败: {e.message}[/red]")

def display_workflow_structure(workflow):
    """显示工作流结构树"""
    tree = Tree("[bold]工作流结构[/bold]")

    for block in workflow.get('blocks', []):
        block_node = tree.add(f"[cyan]{block['block_type']}[/cyan] - {block.get('label', 'Unnamed')}")

        # 添加块参数
        if block.get('parameters'):
            params_node = block_node.add("[dim]参数[/dim]")
            for key, value in block['parameters'].items():
                params_node.add(f"  {key}: {value}")

        # 添加输出schema
        if block.get('output_schema'):
            schema_node = block_node.add("[dim]输出Schema[/dim]")
            schema_node.add(f"  {json.dumps(block['output_schema'], ensure_ascii=False)}")

    console.print(tree)
```

### 4. 认证命令 (`commands/auth.py`)
```python
import click
import keyring
from rich.console import Console
from rich.prompt import Prompt

console = Console()

@auth.command()
@click.option('--email', prompt=True, help='邮箱')
@click.option('--password', prompt=True, hide_input=True, help='密码')
@click.pass_context
def login(ctx, email, password):
    """登录系统"""
    api_client = ctx.obj['api_client']

    try:
        # 执行登录
        login_data = {
            'email': email,
            'password': password
        }

        response = asyncio.run(api_client.post('/api/v1/auth/login', data=login_data))

        # 保存令牌
        api_client.api_key = response['access_token']
        ctx.obj['config_manager'].set('api.key', response['access_token'])

        # 可选：保存到系统密钥环
        if Confirm.ask("是否保存到系统密钥环？"):
            keyring.set_password('skyvern', 'api_key', response['access_token'])

        console.print("[green]✓ 登录成功[/green]")
        console.print(f"欢迎, {response['user']['email']}")

    except APIError as e:
        console.print(f"[red]✗ 登录失败: {e.message}[/red]")

@auth.command()
@click.pass_context
def logout(ctx):
    """登出系统"""
    # 清除本地令牌
    ctx.obj['config_manager'].set('api.key', None)

    # 清除系统密钥环
    try:
        keyring.delete_password('skyvern', 'api_key')
    except keyring.errors.PasswordDeleteError:
        pass

    console.print("[green]✓ 已登出[/green]")

@auth.command()
@click.pass_context
def whoami(ctx):
    """显示当前用户信息"""
    api_client = ctx.obj['api_client']

    if not api_client.api_key:
        console.print("[yellow]未登录，请先使用 'skyvern auth login' 登录[/yellow]")
        return

    try:
        user = asyncio.run(api_client.get('/api/v1/users/me'))

        console.print(Panel.fit(
            f"邮箱: {user['email']}\n"
            f"姓名: {user.get('first_name', '')} {user.get('last_name', '')}\n"
            f"角色: {user.get('role', 'N/A')}\n"
            f"组织: {user.get('organization_name', 'N/A')}",
            title="用户信息"
        ))

    except APIError as e:
        console.print(f"[red]✗ 获取用户信息失败: {e.message}[/red]")
```

## 输出格式化 (`output.py`)

```python
from typing import Any, Dict, List, Optional
from rich.console import Console
from rich.table import Table
from rich.tree import Tree
from rich.panel import Panel
import json
import yaml

class OutputFormatter:
    """输出格式化器"""

    def __init__(self, format_type: str = 'table'):
        self.format_type = format_type
        self.console = Console()

    def format_output(self, data: Any, title: Optional[str] = None) -> str:
        """格式化输出"""
        if self.format_type == 'json':
            return self._format_json(data)
        elif self.format_type == 'yaml':
            return self._format_yaml(data)
        elif self.format_type == 'table':
            return self._format_table(data, title)
        else:
            return str(data)

    def _format_json(self, data: Any) -> str:
        """JSON格式"""
        return json.dumps(data, indent=2, ensure_ascii=False)

    def _format_yaml(self, data: Any) -> str:
        """YAML格式"""
        return yaml.dump(data, allow_unicode=True, default_flow_style=False)

    def _format_table(self, data: Any, title: Optional[str] = None) -> str:
        """表格格式"""
        if isinstance(data, list):
            return self._format_list_as_table(data, title)
        elif isinstance(data, dict):
            return self._format_dict_as_table(data, title)
        else:
            return str(data)

    def _format_list_as_table(self, items: List[Dict], title: Optional[str] = None) -> str:
        """将列表格式化为表格"""
        if not items:
            return "No data to display"

        table = Table(title=title)

        # 添加列
        if items:
            for key in items[0].keys():
                table.add_column(key.replace('_', ' ').title())

        # 添加行
        for item in items:
            values = [str(item.get(key, '')) for key in item.keys()]
            table.add_row(*values)

        # 使用字符串IO捕获输出
        with self.console.capture() as capture:
            self.console.print(table)
        return capture.get()

    def _format_dict_as_table(self, data: Dict, title: Optional[str] = None) -> str:
        """将字典格式化为表格"""
        table = Table(title=title)
        table.add_column("Key")
        table.add_column("Value")

        for key, value in data.items():
            table.add_row(str(key), str(value))

        with self.console.capture() as capture:
            self.console.print(table)
        return capture.get()
```

## 使用示例

### 1. 基本使用
```bash
# 启动所有服务
skyvern server start --mode all

# 创建任务
skyvern task create --title "登录GitHub" --url "https://github.com/login" --goal "使用凭据登录GitHub"

# 执行任务
skyvern task execute <task_id>

# 查看任务列表
skyvern task list --status completed

# 登录
skyvern auth login

# 创建工作流
skyvern workflow create --file workflow.yaml

# 运行工作流
skyvern workflow run <workflow_id> --data '{"username": "test", "password": "test"}'
```

### 2. 配置管理
```bash
# 查看配置
skyvern config show

# 设置配置
skyvern config set api.base_url https://api.skyvern.ai
skyvern config set browser.headless false

# 使用配置文件
skyvern --config /path/to/config.yaml task list
```

### 3. 输出格式
```bash
# JSON输出
skyvern --format json task list

# YAML输出
skyvern --format yaml workflow show <workflow_id>

# 详细输出
skyvern --verbose task execute <task_id>
```

## 开发指南

### 1. 添加新命令
1. 在 `commands/` 目录下创建新的命令文件
2. 使用 Click 装饰器定义命令
3. 在 `main.py` 中注册命令组
4. 实现命令逻辑

### 2. 扩展API客户端
1. 在 `api_client.py` 中添加新的请求方法
2. 处理特定的响应格式
3. 实现错误处理逻辑

### 3. 自定义输出格式
1. 扩展 `OutputFormatter` 类
2. 添加新的格式化方法
3. 支持不同的数据结构

---

> 📖 **返回**: [Skyvern 根文档](../CLAUDE.md) | [Schemas 模块](../schemas/CLAUDE.md) | [Frontend 模块](../../skyvern-frontend/CLAUDE.md)