# AstrBot Dashboard 与核心通信机制汇总

## 1. 系统架构概览

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           AstrBot 系统架构                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────┐          ┌─────────────────────────────────┐  │
│  │    Dashboard (前端)      │          │    Dashboard Server (后端)      │  │
│  │    Vue 3 + TypeScript    │◄────────►│    Quart (Python)               │  │
│  │    Vite + Vuetify        │  HTTP    │    Port: 6185 (default)         │  │
│  └──────────────────────────┘  WebSocket└─────────────────────────────────┘  │
│                                         │                                   │
│                                         │ 调用                                │
│                                         ▼                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    AstrBot Core Lifecycle                           │    │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐  │    │
│  │  │ PlatformMgr  │ │ ProviderMgr  │ │ PluginMgr    │ │ ConfigMgr  │  │    │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                         │                                   │
│                                         │ 读写                               │
│                                         ▼                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         存储层                                      │    │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐  │    │
│  │  │ cmd_config.  │ │ data_v4.db   │ │ plugins/     │ │ data/      │  │    │
│  │  │ json         │ │ (SQLite)     │ │              │ │ attachments│  │    │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. 通信方式总览

Dashboard 与 AstrBot 核心之间采用以下三种通信方式：

| 通信方式 | 用途 | 协议/路径 |
|---------|------|----------|
| **REST API** | 配置管理、插件管理、统计信息等 | HTTP `/api/*` |
| **SSE (Server-Sent Events)** | 实时日志流、聊天流式响应 | HTTP `/api/live-log`, `/api/chat/send` |
| **WebSocket** | 实时聊天、双向通信 | WS `/api/unified_chat/ws` |

## 3. 认证机制

### 登录接口

**端点**: `POST /api/auth/login`

**请求体**:
```json
{
  "username": "astrbot",
  "password": "77b90590a8945a7d36c963981a307dc9"
}
```

> ⚠️ **注意**: 密码需要 MD5 加密后发送，不是明文。
>
> 默认密码 `astrbot` 的 MD5 值为: `77b90590a8945a7d36c963981a307dc9`

**响应**:
```json
{
  "status": "ok",
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "username": "astrbot",
    "change_pwd_hint": true
  }
}
```

- Token 有效期为 7 天
- `change_pwd_hint` 为 `true` 时表示正在使用默认密码，建议修改

### JWT Token 使用

```typescript
// Dashboard 前端 (main.ts)
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers['Authorization'] = `Bearer ${token}`;
  }
  return config;
});
```

- 用户登录后，后端返回 JWT Token
- Token 存储在浏览器 `localStorage` 中
- 每次请求自动携带 `Authorization: Bearer <token>` 头部

### WebSocket 认证

```typescript
// 通过 URL 查询参数传递 Token
const wsUrl = new URL('/api/unified_chat/ws', window.location.href);
wsUrl.searchParams.set('token', token);
```

## 4. 核心 API 端点

### 4.1 配置管理端点 (`/api/config/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/config/get` | GET | 获取 AstrBot 或插件配置 |
| `/config/astrbot/update` | POST | 更新 AstrBot 配置 |
| `/config/plugin/update` | POST | 更新插件配置 |
| `/config/default` | GET | 获取默认配置和元数据 |
| `/config/abconfs` | GET | 获取所有配置文件列表 |
| `/config/platform/*` | - | 平台适配器 CRUD |
| `/config/provider/*` | - | 提供商配置 CRUD |

### 4.2 认证端点 (`/api/auth/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/auth/login` | POST | 登录获取 JWT Token（密码需 MD5 加密） |
| `/auth/account/edit` | POST | 修改用户名和密码 |

### 4.3 插件管理端点 (`/api/plugin/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/plugin/get` | GET | 获取已安装插件列表 |
| `/plugin/market_list` | GET | 获取插件市场列表 |
| `/plugin/install` | POST | 从 Git 仓库安装插件 |
| `/plugin/install-upload` | POST | 从文件上传安装插件 |
| `/plugin/update` | POST | 更新指定插件 |
| `/plugin/uninstall` | POST | 卸载插件 |
| `/plugin/on`, `/plugin/off` | POST | 启用/停用插件 |
| `/plugin/reload` | POST | 重载插件 |

### 4.4 MCP 服务器端点 (`/api/tools/mcp/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/tools/mcp/servers` | GET | 获取 MCP 服务器列表 |
| `/tools/mcp/add` | POST | 添加 MCP 服务器 |
| `/tools/mcp/update` | POST | 更新 MCP 服务器 |
| `/tools/mcp/delete` | POST | 删除 MCP 服务器 |
| `/tools/mcp/test` | POST | 测试 MCP 服务器连接 |

### 4.5 技能（工具）管理端点 (`/api/tools/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/tools/list` | GET | 获取所有可用技能列表 |
| `/tools/toggle-tool` | POST | 启用/禁用指定技能 |

### 4.6 聊天端点 (`/api/chat/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/chat/send` | POST | 发送消息 (SSE 流式返回) |
| `/chat/sessions` | GET | 获取会话列表 |
| `/chat/get_session` | GET | 获取会话详情和历史消息 |
| `/chat/new_session` | GET | 创建新会话 |
| `/chat/delete_session` | GET | 删除会话 |
| `/chat/stop` | POST | 停止正在运行的对话 |
| `/chat/post_file` | POST | 上传文件/图片 |
| `/chat/get_attachment` | GET | 下载附件 |

### 4.8 统计和系统端点 (`/api/stat/*`)

| 端点 | 方法 | 功能 |
|------|------|------|
| `/stat/get` | GET | 获取运行统计信息 |
| `/stat/version` | GET | 获取版本信息 |
| `/stat/restart-core` | POST | 重启 AstrBot 核心 |
| `/stat/changelog` | GET | 获取更新日志 |

## 5. 关键数据流

### 5.1 聊天消息流程

```
用户发送消息
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│ 选择传输模式: SSE 或 WebSocket                              │
└─────────────────────────────────────────────────────────────┘
        │
        ├──────────────────────────┬──────────────────────────┤
        ▼                          ▼                          ▼
┌──────────────┐        ┌──────────────────┐      ┌──────────────┐
│   SSE 模式   │        │   WebSocket 模式  │      │   后端处理   │
│              │        │                  │      │              │
│ POST /chat/  │        │ WS /unified_     │      │ PlatformMgr  │
│ send         │        │ chat/ws          │      │ 路由消息     │
│              │        │                  │      │      │       │
│ 流式返回:    │        │ 双向通信         │      │      ▼       │
│ data: {}     │        │                  │      │ Provider     │
└──────────────┘        └──────────────────┘      │ LLM 处理     │
                                                  │      │       │
                                                  │      ▼       │
                                                  │ 返回流式响应  │
                                                  └──────────────┘
```

### 5.2 插件安装流程

#### 方式一：从 Git 仓库安装

```
调用 POST /api/plugin/install
        │
        ▼
┌─────────────────────────────────────┐
│ 请求体:                              │
│ {                                    │
│   "url": "https://github.com/...",   │
│   "proxy": "https://ghproxy.com",    │
│   "ignore_version_check": false      │
│ }                                    │
└─────────────────────────────────────┘
        │
        ▼
返回安装结果（包含插件信息和版本警告）
```

#### 方式二：手动放置 + 接口加载

```
手动解压插件到 data/plugins/<插件目录>/
        │
        ▼
调用 POST /api/plugin/reload
        │
        ▼
┌─────────────────────────────────────┐
│ 请求体: { "name": "插件名称" }        │
│ （不传 name 则重载所有插件）          │
└─────────────────────────────────────┘
        │
        ▼
返回重载结果
```

#### 方式三：上传文件安装

```
调用 POST /api/plugin/install-upload
Content-Type: multipart/form-data
        │
        ▼
┌─────────────────────────────────────┐
│ form-data:                           │
│   file: <插件 zip 文件>               │
│   ignore_version_check: false        │
└─────────────────────────────────────┘
        │
        ▼
返回安装结果
```

### 5.3 MCP 服务器配置流程

```
调用 POST /api/tools/mcp/add
        │
        ▼
┌─────────────────────────────────────┐
│ 请求体:                              │
│ {                                    │
│   "name": "arxiv-server",            │
│   "active": true,                    │
│   "command": "uv",                   │
│   "args": ["tool", "run", ...],      │
│   "env": {"API_KEY": "xxx"}          │
│ }                                    │
└─────────────────────────────────────┘
        │
        ▼
自动测试连接 ──► 保存配置 ──► 启用服务器
        │
        ▼
返回结果（包含工具列表）
```

### 5.4 技能（工具）管理流程

```
调用 GET /api/tools/list
        │
        ▼
获取所有技能（MCP 工具 + 插件工具）
        │
        ▼
调用 POST /api/tools/toggle-tool
        │
        ▼
┌─────────────────────────────────────┐
│ 请求体: { "name": "工具名",           │
│          "activate": false }         │
└─────────────────────────────────────┘
        │
        ▼
启用/禁用指定技能
```

## 6. 配置文件路径与端口配置

### 配置文件路径

| 文件 | 路径 | 说明 |
|------|------|------|
| 主配置 | `data/cmd_config.json` | AstrBot 主要配置 |
| 数据库 | `data/data_v4.db` | SQLite 数据库 |
| 插件目录 | `data/plugins/` | 安装的插件 |
| 附件目录 | `data/attachments/` | 上传的文件/图片 |
| 日志文件 | `data/logs/astrbot.log` | 运行日志 |
| MCP 配置 | `data/mcp_config.json` | MCP 服务器配置 |

### Dashboard 端口配置

**默认端口**: `6185`

**配置位置**: `data/cmd_config.json` 中的 `dashboard` 字段

```json
{
  "dashboard": {
    "enable": true,
    "host": "0.0.0.0",
    "port": 6185,
    "username": "astrbot",
    "password": "77b90590a8945a7d36c963981a307dc9"
  }
}
```

| 字段 | 默认值 | 说明 |
|------|--------|------|
| `dashboard.enable` | true | 是否启用 Dashboard |
| `dashboard.host` | "0.0.0.0" | 监听地址 |
| `dashboard.port` | 6185 | 监听端口 |
| `dashboard.username` | "astrbot" | 登录用户名 |
| `dashboard.password` | MD5 | 登录密码（MD5 加密）|

**访问地址**: `http://<host>:<port>`，默认 `http://localhost:6185`
