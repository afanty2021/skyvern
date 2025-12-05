# Frontend 模块文档

> 📍 **位置**: [skyvern](../skyvern/CLAUDE.md) → skyvern-frontend

## 模块概述

Frontend 模块是 Skyvern 的 Web 用户界面，基于 React 18 和 TypeScript 构建。它提供了直观的用户界面用于创建、管理和监控浏览器自动化任务和工作流。

## 技术栈

### 核心技术
- **React 18** - 前端框架
- **TypeScript** - 类型安全的JavaScript
- **Vite** - 构建工具和开发服务器
- **React Router v6** - 路由管理
- **TanStack Query** - 服务器状态管理
- **Zustand** - 客户端状态管理

### UI组件库
- **Radix UI** - 无样式组件基础
- **Tailwind CSS** - 原子化CSS框架
- **Shadcn/ui** - 基于Radix的组件库
- **Lucide React** - 图标库
- **Framer Motion** - 动画库

### 开发工具
- **ESLint** - 代码检查
- **Prettier** - 代码格式化
- **Husky** - Git钩子
- **Lint-staged** - 暂存文件检查

## 项目结构

```
skyvern-frontend/
├── public/                  # 静态资源
│   ├── favicon.ico
│   ├── logo192.png
│   └── index.html
├── src/                     # 源代码
│   ├── components/          # 通用组件
│   │   ├── ui/             # UI基础组件
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── table.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── toast.tsx
│   │   │   └── ...
│   │   ├── layout/         # 布局组件
│   │   │   ├── header.tsx
│   │   │   ├── sidebar.tsx
│   │   │   ├── footer.tsx
│   │   │   └── layout.tsx
│   │   ├── forms/          # 表单组件
│   │   │   ├── task-form.tsx
│   │   │   ├── workflow-form.tsx
│   │   │   └── credential-form.tsx
│   │   └── charts/         # 图表组件
│   │       ├── task-chart.tsx
│   │       ├── workflow-chart.tsx
│   │       └── metrics-chart.tsx
│   ├── pages/              # 页面组件
│   │   ├── dashboard/      # 仪表板
│   │   │   ├── index.tsx
│   │   │   ├── overview.tsx
│   │   │   └── metrics.tsx
│   │   ├── tasks/          # 任务页面
│   │   │   ├── index.tsx
│   │   │   ├── create.tsx
│   │   │   ├── detail.tsx
│   │   │   └── execute.tsx
│   │   ├── workflows/      # 工作流页面
│   │   │   ├── index.tsx
│   │   │   ├── create.tsx
│   │   │   ├── detail.tsx
│   │   │   ├── editor.tsx
│   │   │   └── run.tsx
│   │   ├── browser-sessions/  # 浏览器会话
│   │   │   ├── index.tsx
│   │   │   ├── create.tsx
│   │   │   └── control.tsx
│   │   ├── credentials/    # 凭据管理
│   │   │   ├── index.tsx
│   │   │   └── create.tsx
│   │   ├── webhooks/       # Webhook管理
│   │   │   ├── index.tsx
│   │   │   └── create.tsx
│   │   ├── settings/       # 设置页面
│   │   │   ├── profile.tsx
│   │   │   ├── organization.tsx
│   │   │   └── integrations.tsx
│   │   └── auth/           # 认证页面
│   │       ├── login.tsx
│   │       ├── register.tsx
│   │       └── forgot-password.tsx
│   ├── hooks/              # 自定义Hooks
│   │   ├── use-api.ts      # API调用Hook
│   │   ├── use-auth.ts     # 认证Hook
│   │   ├── use-socket.ts   # WebSocket Hook
│   │   ├── use-local-storage.ts  # 本地存储Hook
│   │   └── use-debounce.ts # 防抖Hook
│   ├── services/           # 服务层
│   │   ├── api.ts          # API客户端
│   │   ├── auth.ts         # 认证服务
│   │   ├── websocket.ts    # WebSocket服务
│   │   └── storage.ts      # 存储服务
│   ├── stores/             # 状态管理
│   │   ├── auth-store.ts   # 认证状态
│   │   ├── app-store.ts    # 应用状态
│   │   └── settings-store.ts # 设置状态
│   ├── types/              # 类型定义
│   │   ├── api.ts          # API类型
│   │   ├── auth.ts         # 认证类型
│   │   ├── task.ts         # 任务类型
│   │   ├── workflow.ts     # 工作流类型
│   │   └── common.ts       # 通用类型
│   ├── utils/              # 工具函数
│   │   ├── format.ts       # 格式化工具
│   │   ├── validation.ts   # 验证工具
│   │   ├── constants.ts    # 常量定义
│   │   └── helpers.ts      # 辅助函数
│   ├── styles/             # 样式文件
│   │   ├── globals.css     # 全局样式
│   │   ├── components.css  # 组件样式
│   │   └── themes.css      # 主题样式
│   ├── App.tsx             # 应用根组件
│   ├── main.tsx            # 应用入口
│   └── vite-env.d.ts       # Vite类型声明
├── package.json            # 项目配置
├── tsconfig.json           # TypeScript配置
├── vite.config.ts          # Vite配置
├── tailwind.config.js      # Tailwind配置
├── .eslintrc.js            # ESLint配置
├── .prettierrc             # Prettier配置
└── README.md               # 项目说明
```

## 核心功能模块

### 1. 认证系统 (`pages/auth/`, `services/auth.ts`, `stores/auth-store.ts`)

```typescript
// stores/auth-store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { User } from '@/types/auth';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: async (email: string, password: string) => {
        try {
          const response = await authService.login({ email, password });
          set({
            user: response.user,
            token: response.access_token,
            isAuthenticated: true,
          });
        } catch (error) {
          throw error;
        }
      },

      logout: () => {
        set({
          user: null,
          token: null,
          isAuthenticated: false,
        });
      },

      refreshToken: async () => {
        const { token } = get();
        if (!token) return;

        try {
          const response = await authService.refreshToken(token);
          set({
            user: response.user,
            token: response.access_token,
          });
        } catch (error) {
          get().logout();
        }
      },
    }),
    {
      name: 'auth-storage',
    }
  )
);
```

### 2. 任务管理 (`pages/tasks/`, `hooks/use-api.ts`)

```typescript
// pages/tasks/index.tsx
import { useQuery } from '@tanstack/react-query';
import { TaskTable } from '@/components/tasks/task-table';
import { TaskFilters } from '@/components/tasks/task-filters';
import { CreateTaskButton } from '@/components/tasks/create-task-button';
import { taskService } from '@/services/api';

export function TasksPage() {
  const [filters, setFilters] = useState<TaskFilters>({});

  const {
    data: tasksData,
    isLoading,
    error,
    refetch,
  } = useQuery({
    queryKey: ['tasks', filters],
    queryFn: () => taskService.getTasks(filters),
    staleTime: 5 * 60 * 1000, // 5分钟
  });

  return (
    <div className="container mx-auto py-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">任务管理</h1>
        <CreateTaskButton onSuccess={() => refetch()} />
      </div>

      <TaskFilters
        filters={filters}
        onFiltersChange={setFilters}
        className="mb-6"
      />

      {isLoading ? (
        <TaskTableSkeleton />
      ) : error ? (
        <ErrorMessage error={error} onRetry={refetch} />
      ) : (
        <TaskTable
          tasks={tasksData?.tasks || []}
          total={tasksData?.total || 0}
          onRefresh={refetch}
        />
      )}
    </div>
  );
}
```

### 3. 工作流编辑器 (`pages/workflows/editor.tsx`)

```typescript
// pages/workflows/editor.tsx
import { useState } from 'react';
import { DndProvider } from 'react-dnd';
import { HTML5Backend } from 'react-dnd-html5-backend';
import { WorkflowCanvas } from '@/components/workflows/workflow-canvas';
import { BlockPalette } from '@/components/workflows/block-palette';
import { BlockProperties } from '@/components/workflows/block-properties';
import { Workflow, WorkflowBlock } from '@/types/workflow';

export function WorkflowEditor() {
  const [workflow, setWorkflow] = useState<Workflow | null>(null);
  const [selectedBlock, setSelectedBlock] = useState<WorkflowBlock | null>(null);

  const handleBlockUpdate = (blockId: string, updates: Partial<WorkflowBlock>) => {
    if (!workflow) return;

    setWorkflow({
      ...workflow,
      blocks: workflow.blocks.map(block =>
        block.block_id === blockId ? { ...block, ...updates } : block
      ),
    });
  };

  const handleAddBlock = (blockType: BlockType, position: { x: number; y: number }) => {
    if (!workflow) return;

    const newBlock: WorkflowBlock = {
      block_id: generateId(),
      block_type: blockType,
      label: `${blockType} Block`,
      data: {},
      parameters: {},
      position,
      connections: [],
    };

    setWorkflow({
      ...workflow,
      blocks: [...workflow.blocks, newBlock],
    });
  };

  return (
    <DndProvider backend={HTML5Backend}>
      <div className="flex h-screen">
        {/* 左侧：块调色板 */}
        <div className="w-64 bg-gray-50 border-r">
          <BlockPalette onAddBlock={handleAddBlock} />
        </div>

        {/* 中间：画布 */}
        <div className="flex-1">
          <WorkflowCanvas
            workflow={workflow}
            selectedBlock={selectedBlock}
            onBlockSelect={setSelectedBlock}
            onBlockUpdate={handleBlockUpdate}
            onAddBlock={handleAddBlock}
          />
        </div>

        {/* 右侧：属性面板 */}
        <div className="w-80 bg-gray-50 border-l">
          <BlockProperties
            block={selectedBlock}
            onUpdate={(updates) => {
              if (selectedBlock) {
                handleBlockUpdate(selectedBlock.block_id, updates);
              }
            }}
          />
        </div>
      </div>
    </DndProvider>
  );
}
```

### 4. 实时监控 (`hooks/use-socket.ts`)

```typescript
// hooks/use-socket.ts
import { useEffect, useRef } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { websocketService } from '@/services/websocket';

export function useSocket() {
  const queryClient = useQueryClient();
  const socketRef = useRef<WebSocket | null>(null);

  useEffect(() => {
    // 建立WebSocket连接
    const socket = websocketService.connect();
    socketRef.current = socket;

    // 监听任务更新
    socket.on('task:update', (data) => {
      queryClient.setQueryData(['tasks', data.task_id], data);
      queryClient.invalidateQueries(['tasks']);
    });

    // 监听工作流运行更新
    socket.on('workflow_run:update', (data) => {
      queryClient.setQueryData(['workflow_runs', data.run_id], data);
      queryClient.invalidateQueries(['workflow_runs']);
    });

    // 监听浏览器会话更新
    socket.on('browser_session:update', (data) => {
      queryClient.setQueryData(['browser_sessions', data.session_id], data);
      queryClient.invalidateQueries(['browser_sessions']);
    });

    return () => {
      socket.disconnect();
    };
  }, [queryClient]);

  const emit = (event: string, data: any) => {
    if (socketRef.current) {
      socketRef.current.emit(event, data);
    }
  };

  return { emit };
}
```

### 5. API 客户端 (`services/api.ts`)

```typescript
// services/api.ts
import axios, { AxiosInstance, AxiosResponse } from 'axios';

class APIClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000',
      timeout: 30000,
    });

    // 请求拦截器
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('auth-storage');
        if (token) {
          const { state } = JSON.parse(token);
          if (state.token) {
            config.headers.Authorization = `Bearer ${state.token}`;
          }
        }
        return config;
      },
      (error) => {
        return Promise.reject(error);
      }
    );

    // 响应拦截器
    this.client.interceptors.response.use(
      (response) => response,
      async (error) => {
        if (error.response?.status === 401) {
          // 处理token过期
          const authStore = useAuthStore.getState();
          try {
            await authStore.refreshToken();
            // 重试原请求
            return this.client.request(error.config);
          } catch {
            authStore.logout();
            window.location.href = '/login';
          }
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string, params?: any): Promise<T> {
    const response: AxiosResponse<T> = await this.client.get(url, { params });
    return response.data;
  }

  async post<T>(url: string, data?: any): Promise<T> {
    const response: AxiosResponse<T> = await this.client.post(url, data);
    return response.data;
  }

  async put<T>(url: string, data?: any): Promise<T> {
    const response: AxiosResponse<T> = await this.client.put(url, data);
    return response.data;
  }

  async delete<T>(url: string): Promise<T> {
    const response: AxiosResponse<T> = await this.client.delete(url);
    return response.data;
  }
}

export const apiClient = new APIClient();
```

## UI 组件设计

### 1. 基础组件 (`components/ui/`)

基于 Shadcn/ui 的组件系统：

```typescript
// components/ui/button.tsx
import { forwardRef } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/utils/helpers';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary',
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
```

### 2. 复合组件 (`components/tasks/task-table.tsx`)

```typescript
// components/tasks/task-table.tsx
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Task } from '@/types/task';
import { formatDate, formatDuration } from '@/utils/format';

interface TaskTableProps {
  tasks: Task[];
  onExecute?: (taskId: string) => void;
  onView?: (taskId: string) => void;
}

export function TaskTable({ tasks, onExecute, onView }: TaskTableProps) {
  const getStatusBadge = (status: Task['status']) => {
    const variants = {
      pending: 'secondary',
      running: 'default',
      completed: 'default',
      failed: 'destructive',
      cancelled: 'secondary',
    } as const;

    return (
      <Badge variant={variants[status] || 'secondary'}>
        {status}
      </Badge>
    );
  };

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>标题</TableHead>
          <TableHead>URL</TableHead>
          <TableHead>状态</TableHead>
          <TableHead>创建时间</TableHead>
          <TableHead>执行时长</TableHead>
          <TableHead>操作</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {tasks.map((task) => (
          <TableRow key={task.task_id}>
            <TableCell className="font-medium">{task.title}</TableCell>
            <TableCell>
              <div className="max-w-xs truncate" title={task.url}>
                {task.url}
              </div>
            </TableCell>
            <TableCell>{getStatusBadge(task.status)}</TableCell>
            <TableCell>{formatDate(task.created_at)}</TableCell>
            <TableCell>
              {task.execution_time && formatDuration(task.execution_time)}
            </TableCell>
            <TableCell>
              <div className="flex gap-2">
                {task.status === 'pending' && onExecute && (
                  <Button
                    size="sm"
                    variant="outline"
                    onClick={() => onExecute(task.task_id)}
                  >
                    执行
                  </Button>
                )}
                {onView && (
                  <Button
                    size="sm"
                    variant="ghost"
                    onClick={() => onView(task.task_id)}
                  >
                    查看
                  </Button>
                )}
              </div>
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

## 状态管理

### 1. 全局状态 (Zustand)

```typescript
// stores/app-store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AppState {
  theme: 'light' | 'dark';
  language: string;
  sidebarCollapsed: boolean;
  notifications: Notification[];
  setTheme: (theme: 'light' | 'dark') => void;
  setLanguage: (language: string) => void;
  toggleSidebar: () => void;
  addNotification: (notification: Notification) => void;
  removeNotification: (id: string) => void;
}

export const useAppStore = create<AppState>()(
  persist(
    (set, get) => ({
      theme: 'light',
      language: 'en',
      sidebarCollapsed: false,
      notifications: [],

      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleSidebar: () => set((state) => ({ sidebarCollapsed: !state.sidebarCollapsed })),

      addNotification: (notification) =>
        set((state) => ({
          notifications: [...state.notifications, notification],
        })),

      removeNotification: (id) =>
        set((state) => ({
          notifications: state.notifications.filter((n) => n.id !== id),
        })),
    }),
    {
      name: 'app-storage',
      partialize: (state) => ({
        theme: state.theme,
        language: state.language,
        sidebarCollapsed: state.sidebarCollapsed,
      }),
    }
  )
);
```

### 2. 服务器状态 (TanStack Query)

```typescript
// hooks/use-api.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { taskService, Task, CreateTaskRequest } from '@/services/api';

export function useTasks(filters?: TaskFilters) {
  return useQuery({
    queryKey: ['tasks', filters],
    queryFn: () => taskService.getTasks(filters),
    staleTime: 5 * 60 * 1000, // 5分钟
  });
}

export function useTask(taskId: string) {
  return useQuery({
    queryKey: ['tasks', taskId],
    queryFn: () => taskService.getTask(taskId),
    enabled: !!taskId,
  });
}

export function useCreateTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateTaskRequest) => taskService.createTask(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
}

export function useExecuteTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (taskId: string) => taskService.executeTask(taskId),
    onSuccess: (_, taskId) => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
      queryClient.invalidateQueries({ queryKey: ['tasks', taskId] });
    },
  });
}
```

## 样式系统

### 1. Tailwind CSS 配置

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ['class'],
  content: [
    './index.html',
    './src/**/*.{ts,tsx,js,jsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      keyframes: {
        'accordion-down': {
          from: { height: 0 },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: 0 },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

### 2. 主题变量

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;

    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;

    --secondary: 210 40% 96%;
    --secondary-foreground: 222.2 84% 4.9%;

    --muted: 210 40% 96%;
    --muted-foreground: 215.4 16.3% 46.9%;

    --accent: 210 40% 96%;
    --accent-foreground: 222.2 84% 4.9%;

    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;

    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;

    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 84% 4.9%;

    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;

    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 94.1%;
  }
}
```

## 构建与部署

### 1. 构建配置 (`vite.config.ts`)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
      '/ws': {
        target: 'ws://localhost:8000',
        ws: true,
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          charts: ['recharts'],
        },
      },
    },
  },
});
```

### 2. Docker 部署

```dockerfile
# Dockerfile
FROM node:18-alpine as builder

WORKDIR /app

# 安装依赖
COPY package.json package-lock.json ./
RUN npm ci --only=production

# 构建应用
COPY . .
RUN npm run build

# 生产环境
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 3. GitHub Actions CI/CD

```yaml
# .github/workflows/frontend.yml
name: Frontend CI/CD

on:
  push:
    branches: [main]
    paths: ['skyvern-frontend/**']
  pull_request:
    branches: [main]
    paths: ['skyvern-frontend/**']

jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./skyvern-frontend

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: skyvern-frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run type checking
        run: npm run type-check

      - name: Run tests
        run: npm run test:ci

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          directory: ./skyvern-frontend/coverage

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    defaults:
      run:
        working-directory: ./skyvern-frontend

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: skyvern-frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
        env:
          VITE_API_BASE_URL: ${{ secrets.API_BASE_URL }}

      - name: Build Docker image
        run: |
          docker build -t skyvern/frontend:${{ github.sha }} .
          docker tag skyvern/frontend:${{ github.sha }} skyvern/frontend:latest

      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push skyvern/frontend:${{ github.sha }}
          docker push skyvern/frontend:latest
```

## 性能优化

### 1. 代码分割

```typescript
// 路由级别的代码分割
import { lazy } from 'react';

const Dashboard = lazy(() => import('@/pages/dashboard'));
const Tasks = lazy(() => import('@/pages/tasks'));
const Workflows = lazy(() => import('@/pages/workflows'));

// 组件级别的代码分割
const WorkflowEditor = lazy(() => import('@/components/workflows/workflow-editor'));
```

### 2. 虚拟滚动

```typescript
// 使用 react-window 处理大量数据
import { FixedSizeList as List } from 'react-window';

const TaskList = ({ tasks }: { tasks: Task[] }) => {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <TaskRow task={tasks[index]} />
    </div>
  );

  return (
    <List
      height={600}
      itemCount={tasks.length}
      itemSize={80}
    >
      {Row}
    </List>
  );
};
```

### 3. 缓存策略

```typescript
// API 缓存配置
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5分钟
      cacheTime: 10 * 60 * 1000, // 10分钟
      retry: 3,
      retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
});
```

## 测试策略

### 1. 单元测试 (Vitest + React Testing Library)

```typescript
// __tests__/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/ui/button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toBeInTheDocument();
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('handles click events', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('applies variant styles', () => {
    render(<Button variant="destructive">Delete</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-destructive');
  });
});
```

### 2. 集成测试

```typescript
// __tests__/pages/TasksPage.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { TasksPage } from '@/pages/tasks';

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

describe('TasksPage', () => {
  it('displays loading state', () => {
    const queryClient = createTestQueryClient();

    render(
      <QueryClientProvider client={queryClient}>
        <TasksPage />
      </QueryClientProvider>
    );

    expect(screen.getByTestId('task-table-skeleton')).toBeInTheDocument();
  });

  it('displays tasks after loading', async () => {
    const queryClient = createTestQueryClient();
    const mockTasks = [
      { task_id: '1', title: 'Task 1', status: 'completed' },
      { task_id: '2', title: 'Task 2', status: 'pending' },
    ];

    queryClient.setQueryData(['tasks', {}], { tasks: mockTasks, total: 2 });

    render(
      <QueryClientProvider client={queryClient}>
        <TasksPage />
      </QueryClientProvider>
    );

    await waitFor(() => {
      expect(screen.getByText('Task 1')).toBeInTheDocument();
      expect(screen.getByText('Task 2')).toBeInTheDocument();
    });
  });
});
```

### 3. E2E 测试 (Playwright)

```typescript
// e2e/tasks.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Tasks', () => {
  test.beforeEach(async ({ page }) => {
    // 登录
    await page.goto('/login');
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    await page.click('[data-testid="login-button"]');
    await expect(page).toHaveURL('/dashboard');
  });

  test('should create and execute a task', async ({ page }) => {
    // 导航到任务页面
    await page.click('[data-testid="tasks-nav-link"]');
    await expect(page).toHaveURL('/tasks');

    // 创建任务
    await page.click('[data-testid="create-task-button"]');
    await expect(page.locator('[data-testid="create-task-modal"]')).toBeVisible();

    await page.fill('[data-testid="task-title-input"]', 'Test Task');
    await page.fill('[data-testid="task-url-input"]', 'https://example.com');
    await page.fill('[data-testid="task-goal-input"]', 'Extract information');
    await page.click('[data-testid="submit-task-button"]');

    // 验证任务已创建
    await expect(page.locator('[data-testid="task-table"]')).toContainText('Test Task');

    // 执行任务
    await page.click('[data-testid="execute-task-button"]');
    await expect(page.locator('[data-testid="task-status"]')).toContainText('running');
  });
});
```

## 开发指南

### 1. 添加新功能

1. **创建类型定义**
   ```typescript
   // types/new-feature.ts
   export interface NewFeature {
     id: string;
     name: string;
     // ...
   }
   ```

2. **添加 API 服务**
   ```typescript
   // services/new-feature.ts
   export const newFeatureService = {
     getAll: () => apiClient.get<NewFeature[]>('/new-features'),
     create: (data: CreateNewFeatureRequest) => apiClient.post<NewFeature>('/new-features', data),
     // ...
   };
   ```

3. **创建 Hooks**
   ```typescript
   // hooks/use-new-feature.ts
   export function useNewFeatures() {
     return useQuery({
       queryKey: ['new-features'],
       queryFn: newFeatureService.getAll,
     });
   }
   ```

4. **构建组件**
   ```typescript
   // components/new-feature/new-feature-list.tsx
   export function NewFeatureList() {
     const { data: features, isLoading } = useNewFeatures();
     // ...
   }
   ```

5. **添加页面**
   ```typescript
   // pages/new-features/index.tsx
   export default function NewFeaturesPage() {
     return (
       <Layout>
         <NewFeatureList />
       </Layout>
     );
   }
   ```

6. **更新路由**
   ```typescript
   // App.tsx
   <Route path="/new-features" element={<NewFeaturesPage />} />
   ```

### 2. 代码规范

- 使用 TypeScript 进行类型安全开发
- 遵循 ESLint 和 Prettier 配置
- 组件使用函数式组件和 Hooks
- 使用语义化的 HTML 标签
- 添加适当的注释和文档

### 3. 性能最佳实践

- 使用 `React.memo` 防止不必要的重渲染
- 使用 `useMemo` 和 `useCallback` 优化计算
- 实现虚拟滚动处理大量数据
- 使用代码分割减少初始加载时间
- 优化图片和静态资源加载

---

> 📖 **返回**: [Skyvern 根文档](../skyvern/CLAUDE.md) | [CLI 模块](../skyvern/cli/CLAUDE.md)