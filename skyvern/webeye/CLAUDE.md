# WebEye 模块文档

> 📍 **位置**: [skyvern](../CLAUDE.md) → webeye

## 模块概述

WebEye 是 Skyvern 的浏览器自动化引擎，基于 Playwright 实现，负责所有浏览器相关的操作，包括页面导航、元素交互、截图获取、DOM 解析等。它是连接 AI 决策和实际浏览器操作的桥梁。

## 文件结构

```
skyvern/webeye/
├── __init__.py
├── browser_factory.py       # 浏览器实例管理
├── browser_manager.py       # 浏览器会话管理
├── page_manager.py          # 页面状态管理
├── dom_scraper.py           # DOM 内容抓取
├── element_finder.py        # 元素定位器
├── action_executor.py       # 动作执行引擎
├── screenshot_manager.py    # 截图管理
├── cookie_manager.py        # Cookie 管理
├── proxy_manager.py         # 代理管理
├── download_manager.py      # 下载管理
├── recorder/                # 录制功能
│   ├── __init__.py
│   ├── session_recorder.py  # 会话录制器
│   └── action_recorder.py   # 动作录制器
├── actions/                 # 动作定义
│   ├── __init__.py
│   ├── base.py             # 基础动作类
│   ├── navigation.py       # 导航动作
│   ├── input.py            # 输入动作
│   ├── click.py            # 点击动作
│   ├── select.py           # 选择动作
│   ├── upload.py           # 上传动作
│   ├── download.py         # 下载动作
│   ├── wait.py             # 等待动作
│   ├── scroll.py           # 滚动动作
│   ├── switch.py           # 切换动作
│   ├── keyboard.py         # 键盘动作
│   ├── mouse.py            # 鼠标动作
│   ├── captcha.py          # 验证码处理
│   └── custom.py           # 自定义动作
├── utils/                  # 工具函数
│   ├── __init__.py
│   ├── dom_utils.py        # DOM 操作工具
│   ├── element_utils.py    # 元素操作工具
│   ├── selector_utils.py   # 选择器工具
│   └── wait_utils.py       # 等待工具
└── exceptions.py           # 自定义异常
```

## 核心组件

### 1. 浏览器工厂 (`browser_factory.py`)
```python
class BrowserFactory:
    """创建和管理浏览器实例的工厂类"""

    @staticmethod
    def create_browser(
        browser_type: str = "chromium",
        headless: bool = True,
        proxy: Optional[ProxySettings] = None,
        user_data_dir: Optional[str] = None,
        viewport: Optional[Viewport] = None
    ) -> Browser:
        """创建新的浏览器实例"""

    @staticmethod
    def get_browser_options(
        browser_type: str,
        headless: bool,
        proxy: Optional[ProxySettings]
    ) -> BrowserTypeLauncherOptions:
        """获取浏览器启动选项"""
```

### 2. 浏览器管理器 (`browser_manager.py`)
```python
class BrowserManager:
    """管理浏览器生命周期和资源"""

    def __init__(self):
        self.active_browsers: Dict[str, Browser] = {}
        self.browser_pools: Dict[str, List[Browser]] = {}

    async def get_browser(
        self,
        browser_id: str,
        create_if_missing: bool = True
    ) -> Browser:
        """获取或创建浏览器实例"""

    async def close_browser(self, browser_id: str) -> None:
        """关闭指定浏览器"""

    async def cleanup_all(self) -> None:
        """清理所有浏览器资源"""
```

### 3. 页面管理器 (`page_manager.py`)
```python
class PageManager:
    """管理页面状态和导航"""

    def __init__(self, browser: Browser):
        self.browser = browser
        self.pages: Dict[str, Page] = {}

    async def get_page(self, page_id: str) -> Page:
        """获取页面实例"""

    async def navigate_to(
        self,
        page: Page,
        url: str,
        wait_until: str = "networkidle"
    ) -> NavigationResult:
        """导航到指定URL"""

    async def wait_for_page_ready(
        self,
        page: Page,
        timeout: int = 30000
    ) -> bool:
        """等待页面加载完成"""
```

### 4. DOM 抓取器 (`dom_scraper.py`)
```python
class DOMScraper:
    """提取和分析页面DOM内容"""

    async def scrape_page(
        self,
        page: Page,
        include_hidden: bool = False,
        max_depth: int = 10
    ) -> DOMTree:
        """抓取完整的页面DOM"""

    async def extract_forms(self, page: Page) -> List[Form]:
        """提取页面中的表单"""

    async def extract_links(self, page: Page) -> List[Link]:
        """提取页面中的链接"""

    async def extract_images(self, page: Page) -> List[Image]:
        """提取页面中的图片"""
```

### 5. 元素定位器 (`element_finder.py`)
```python
class ElementFinder:
    """高级元素定位和选择器生成"""

    async def find_element(
        self,
        page: Page,
        selector: str,
        timeout: int = 10000,
        state: str = "visible"
    ) -> Optional[ElementHandle]:
        """查找单个元素"""

    async def find_elements(
        self,
        page: Page,
        selector: str,
        timeout: int = 10000
    ) -> List[ElementHandle]:
        """查找多个元素"""

    def generate_smart_selector(
        self,
        element_description: str,
        page_context: Dict
    ) -> str:
        """生成智能选择器"""
```

### 6. 动作执行引擎 (`action_executor.py`)
```python
class ActionExecutor:
    """执行所有类型的浏览器动作"""

    async def execute_action(
        self,
        page: Page,
        action: Action,
        context: ExecutionContext
    ) -> ActionResult:
        """执行单个动作"""

    async def execute_actions(
        self,
        page: Page,
        actions: List[Action],
        context: ExecutionContext
    ) -> List[ActionResult]:
        """批量执行动作"""

    async def wait_for_action_result(
        self,
        action: Action,
        timeout: int = 30000
    ) -> ActionResult:
        """等待动作执行结果"""
```

## 动作系统详解

### 基础动作类 (`actions/base.py`)
```python
@dataclass
class Action:
    """基础动作抽象类"""
    action_id: str
    action_type: str
    selector: Optional[str] = None
    parameters: Dict[str, Any] = field(default_factory=dict)
    timeout: int = 10000
    retry_count: int = 0
    max_retries: int = 3

    async def execute(
        self,
        page: Page,
        context: ExecutionContext
    ) -> ActionResult:
        """执行动作的抽象方法"""
        raise NotImplementedError
```

### 主要动作类型

#### 1. 点击动作 (`actions/click.py`)
```python
class ClickAction(Action):
    """点击元素动作"""
    action_type = "click"

    async def execute(self, page: Page, context: ExecutionContext) -> ActionResult:
        element = await self._find_element(page)
        await element.click(
            button=self.parameters.get("button", "left"),
            modifiers=self.parameters.get("modifiers", []),
            position=self.parameters.get("position")
        )
        return ActionResult(success=True, action_id=self.action_id)
```

#### 2. 输入动作 (`actions/input.py`)
```python
class InputAction(Action):
    """输入文本动作"""
    action_type = "input"

    async def execute(self, page: Page, context: ExecutionContext) -> ActionResult:
        element = await self._find_element(page)
        text = self.parameters.get("text", "")
        clear_first = self.parameters.get("clear_first", True)

        if clear_first:
            await element.clear()
        await element.fill(text)

        return ActionResult(success=True, action_id=self.action_id)
```

#### 3. 选择动作 (`actions/select.py`)
```python
class SelectAction(Action):
    """下拉选择动作"""
    action_type = "select"

    async def execute(self, page: Page, context: ExecutionContext) -> ActionResult:
        element = await self._find_element(page)
        value = self.parameters.get("value")
        index = self.parameters.get("index")
        label = self.parameters.get("label")

        select_element = SelectHandle(element)
        if value:
            await select_element.select_option(value=value)
        elif index is not None:
            await select_element.select_option(index=index)
        elif label:
            await select_element.select_option(label=label)

        return ActionResult(success=True, action_id=self.action_id)
```

#### 4. 上传动作 (`actions/upload.py`)
```python
class UploadAction(Action):
    """文件上传动作"""
    action_type = "upload"

    async def execute(self, page: Page, context: ExecutionContext) -> ActionResult:
        file_paths = self.parameters.get("file_paths", [])
        element = await self._find_element(page)

        with tempfile.TemporaryDirectory() as temp_dir:
            # 处理文件路径
            processed_files = await self._prepare_files(file_paths, temp_dir)
            await element.set_input_files(processed_files)

        return ActionResult(success=True, action_id=self.action_id)
```

#### 5. 验证码处理 (`actions/captcha.py`)
```python
class CaptchaAction(Action):
    """验证码识别和处理"""
    action_type = "captcha"

    async def execute(self, page: Page, context: ExecutionContext) -> ActionResult:
        captcha_type = self.parameters.get("type", "image")

        if captcha_type == "image":
            return await self._solve_image_captcha(page)
        elif captcha_type == "recaptcha":
            return await self._solve_recaptcha(page)
        elif captcha_type == "hcaptcha":
            return await self._solve_hcaptcha(page)

        return ActionResult(success=False, error="Unsupported captcha type")
```

## 截图管理 (`screenshot_manager.py`)

```python
class ScreenshotManager:
    """管理页面截图"""

    async def take_full_page_screenshot(
        self,
        page: Page,
        format: str = "png",
        quality: int = 90
    ) -> bytes:
        """截取整个页面"""

    async def take_element_screenshot(
        self,
        page: Page,
        selector: str,
        padding: int = 0
    ) -> bytes:
        """截取指定元素"""

    async def take_viewport_screenshot(
        self,
        page: Page
    ) -> bytes:
        """截取当前视口"""
```

## Cookie 管理 (`cookie_manager.py`)

```python
class CookieManager:
    """管理浏览器Cookie"""

    async def get_cookies(
        self,
        page: Page,
        urls: Optional[List[str]] = None
    ) -> List[Dict]:
        """获取Cookie"""

    async def set_cookies(
        self,
        page: Page,
        cookies: List[Dict]
    ) -> None:
        """设置Cookie"""

    async def clear_cookies(
        self,
        page: Page,
        domain: Optional[str] = None
    ) -> None:
        """清除Cookie"""
```

## 代理管理 (`proxy_manager.py`)

```python
class ProxyManager:
    """管理代理配置"""

    def __init__(self):
        self.proxy_pools: Dict[str, List[Proxy]] = {}
        self.current_proxy_index: Dict[str, int] = {}

    async def get_proxy(
        self,
        region: str = "us",
        rotate: bool = False
    ) -> Optional[Proxy]:
        """获取代理"""

    async def rotate_proxy(
        self,
        page: Page,
        new_proxy: Proxy
    ) -> None:
        """切换代理"""

    async def test_proxy(
        self,
        proxy: Proxy,
        test_url: str = "http://example.com"
    ) -> bool:
        """测试代理连通性"""
```

## 录制功能 (`recorder/`)

### 会话录制器 (`session_recorder.py`)
```python
class SessionRecorder:
    """录制完整的浏览器会话"""

    def __init__(self, output_dir: str):
        self.output_dir = output_dir
        self.actions: List[RecordedAction] = []
        self.screenshots: List[RecordedScreenshot] = []

    async def start_recording(self, page: Page) -> None:
        """开始录制"""

    async def record_action(
        self,
        action: Action,
        result: ActionResult
    ) -> None:
        """录制动作"""

    async def take_recording_screenshot(
        self,
        page: Page,
        action_id: str
    ) -> None:
        """录制截图"""

    async def save_recording(self) -> RecordingSession:
        """保存录制结果"""
```

## 工具函数 (`utils/`)

### DOM 工具 (`dom_utils.py`)
- `get_element_xpath()`: 获取元素的XPath
- `get_element_css_path()`: 获取CSS路径
- `compare_elements()`: 比较两个元素
- `find_similar_elements()`: 查找相似元素

### 选择器工具 (`selector_utils.py`)
- `optimize_selector()`: 优化选择器
- `selector_to_xpath()`: CSS选择器转XPath
- `generate_fallback_selectors()`: 生成备用选择器

## 异常处理 (`exceptions.py`)

```python
class WebEyeException(Exception):
    """WebEye基础异常"""
    pass

class ElementNotFoundError(WebEyeException):
    """元素未找到异常"""
    pass

class ActionTimeoutError(WebEyeException):
    """动作执行超时异常"""
    pass

class PageLoadError(WebEyeException):
    """页面加载失败异常"""
    pass

class ProxyConnectionError(WebEyeException):
    """代理连接失败异常"""
    pass
```

## 性能优化

### 1. 元素缓存
- 缓存常用元素的选择器
- 重用已定位的元素句柄
- 智能更新缓存策略

### 2. 并发控制
- 限制同时进行的浏览器实例数
- 队列管理系统
- 资源池化

### 3. 内存管理
- 及时释放不用的页面
- 垃圾回收优化
- 内存使用监控

## 安全考虑

### 1. 隔离性
- 每个任务使用独立的浏览器上下文
- Cookie和存储隔离
- 防止跨站脚本攻击

### 2. 隐私保护
- 清理敏感信息
- 禁用不必要的浏览器功能
- 安全的文件处理

### 3. 资源限制
- CPU和内存使用限制
- 网络带宽控制
- 磁盘空间管理

## 测试策略

### 1. 单元测试
- 测试每个动作的执行
- 验证元素定位逻辑
- 测试异常处理

### 2. 集成测试
- 端到端工作流测试
- 多页面交互测试
- 错误恢复测试

### 3. 性能测试
- 并发执行测试
- 资源使用测试
- 响应时间测试

## 开发指南

### 添加新动作
1. 继承 `Action` 基类
2. 实现 `execute()` 方法
3. 定义参数验证
4. 添加测试用例

### 优化性能
1. 分析性能瓶颈
2. 优化选择器策略
3. 改进并发处理
4. 监控资源使用

### 调试技巧
1. 启用详细日志
2. 使用调试模式
3. 截图辅助调试
4. 分析网络请求

---

> 📖 **返回**: [Skyvern 根文档](../CLAUDE.md) | [Agent 模块](../agent/CLAUDE.md) | [Forge 模块](../forge/CLAUDE.md)