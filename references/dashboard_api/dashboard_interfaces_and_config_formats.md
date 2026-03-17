# AstrBot Dashboard 接口与配置格式详解

## 目录

1. [Dashboard API 接口详解](#1-dashboard-api-接口详解)
   - [1.1 认证接口](#11-认证接口)
   - [1.2 配置管理接口](#12-配置管理接口)
   - [1.3 平台适配器接口](#13-平台适配器接口)
   - [1.4 提供商接口](#14-提供商接口)
   - [1.5 插件管理接口](#15-插件管理接口)
   - [1.6 聊天接口](#16-聊天接口)
   - [1.7 统计接口](#17-统计接口)
   - [1.8 实时日志接口 (SSE)](#18-实时日志接口-sse)
   - [1.9 文件服务](#19-文件服务)
   - [1.10 MCP 服务器接口](#110-mcp-服务器接口)
   - [1.11 技能（工具）管理接口](#111-技能工具管理接口)
2. [配置文件格式详解](#2-配置文件格式详解)
   - [2.1 主配置文件](#21-主配置文件)
   - [2.2 插件配置文件](#22-插件配置文件)
   - [2.3 MCP 服务器配置文件](#23-mcp-服务器配置文件)
3. [消息传输协议](#3-消息传输协议)
4. [数据模型定义](#4-数据模型定义)

---

## 1. Dashboard API 接口详解

### 1.1 认证接口

#### 登录

**端点**: `POST /api/auth/login`

**Content-Type**: `application/json`

**请求体**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `username` | string | 是 | 用户名（默认 `astrbot`） |
| `password` | string | 是 | 密码的 **MD5 哈希值** |

> ⚠️ **注意**: 密码需要 MD5 加密后发送，不是明文。
>
> 默认密码 `astrbot` 的 MD5 值为: `77b90590a8945a7d36c963981a307dc9`

**请求示例**:
```json
{
  "username": "astrbot",
  "password": "77b90590a8945a7d36c963981a307dc9"
}
```

**响应示例 - 成功**:
```json
{
  "status": "ok",
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGcOiJIUzI1NiJ9...",
    "username": "astrbot",
    "change_pwd_hint": true
  }
}
```

**响应字段**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `token` | string | JWT Token，用于后续请求认证 |
| `username` | string | 登录用户名 |
| `change_pwd_hint` | boolean | 是否使用默认密码，为 `true` 时建议修改 |

**响应示例 - 失败**:
```json
{
  "status": "error",
  "message": "用户名或密码错误"
}
```

**使用说明**:
- 获取 Token 后，后续请求需要在请求头中添加 `Authorization: Bearer <token>`
- Token 有效期为 7 天
- 默认密码为 `astrbot`，生产环境建议尽快修改

#### 修改账号信息

**端点**: `POST /api/auth/account/edit`

**Content-Type**: `application/json`

**请求头**: `Authorization: Bearer <token>`

**请求体**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `password` | string | 是 | 当前密码的 MD5 值 |
| `new_username` | string | 否 | 新用户名（与 new_password 至少填一个） |
| `new_password` | string | 否 | 新密码的 MD5 值 |
| `confirm_password` | string | 条件 | 确认新密码的 MD5 值（当 new_password 存在时必填） |

**请求示例**:
```json
{
  "password": "77b90590a8945a7d36c963981a307dc9",
  "new_username": "admin",
  "new_password": "e10adc3949ba59abbe56e057f20f883e",
  "confirm_password": "e10adc3949ba59abbe56e057f20f883e"
}
```

**响应示例 - 成功**:
```json
{
  "status": "ok",
  "message": "修改成功"
}
```

**响应示例 - 失败**:
```json
{
  "status": "error",
  "message": "原密码错误"
}
```

---

### 1.2 配置管理接口

#### 获取配置

**端点**: `GET /api/config/get`

**请求头**: `Authorization: Bearer <token>`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 配置名称，`astrbot` 或插件名 |
| `umop` | string | 否 | UMO (统一消息对象) 标识 |

**响应示例**:
```json
{
  "status": "ok",
  "message": "",
  "data": {
    "config": {
      "config_version": 2,
      "platform_settings": { ... },
      "provider_settings": { ... }
    }
  }
}
```

#### 更新 AstrBot 配置

**端点**: `POST /api/config/astrbot/update`

**请求体**:
```json
{
  "config": {
    "config_version": 2,
    "platform_settings": {
      "unique_session": false,
      "rate_limit": {
        "time": 60,
        "count": 30,
        "strategy": "stall"
      }
    }
  }
}
```

**响应示例**:
```json
{
  "status": "ok",
  "message": "配置保存成功"
}
```

#### 获取默认配置和元数据

**端点**: `GET /api/config/default`

**响应示例**:
```json
{
  "status": "ok",
  "data": {
    "default_config": { ... },
    "metadata": { ... },
    "metadata_v3": { ... }
  }
}
```

### 1.3 平台适配器接口

#### 获取平台列表

**端点**: `GET /api/config/platform/list`

**请求头**: `Authorization: Bearer <token>`

**响应示例**:
```json
{
  "status": "ok",
  "data": [
    {
      "id": "default",
      "type": "qq_official",
      "enable": true,
      "appid": "xxx",
      "secret": "***"
    }
  ]
}
```

#### 添加新平台

**端点**: `POST /api/config/platform/new`

**请求头**: `Authorization: Bearer <token>`

**请求体**:
```json
{
  "platform": {
    "id": "my_bot",
    "type": "aiocqhttp",
    "enable": true,
    "ws_reverse_host": "0.0.0.0",
    "ws_reverse_port": 6199
  }
}
```

#### 更新平台配置

**端点**: `POST /api/config/platform/update`

**请求头**: `Authorization: Bearer <token>`

**请求体**:
```json
{
  "index": 0,
  "platform": {
    "id": "my_bot",
    "enable": false
  }
}
```

#### 删除平台

**端点**: `POST /api/config/platform/delete`

**请求体**:
```json
{
  "index": 0
}
```

### 1.4 提供商接口

#### 获取提供商列表

**端点**: `GET /api/config/provider/list`

#### 添加新提供商

**端点**: `POST /api/config/provider/new`

**请求体**:
```json
{
  "provider_source_id": "openai_source",
  "provider": {
    "id": "gpt4",
    "type": "chat_completion",
    "enable": true,
    "model": "gpt-4"
  }
}
```

### 1.5 插件管理接口

#### 获取已安装插件

**端点**: `GET /api/plugin/get`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 否 | 插件名称，为空返回所有 |

**响应示例**:
```json
{
  "status": "ok",
  "data": [
    {
      "name": "astrbot_plugin_example",
      "display_name": "示例插件",
      "author": "Author",
      "desc": "插件描述",
      "version": "1.0.0",
      "activated": true,
      "reserved": false,
      "handlers": [
        {
          "event_type": "AdapterMessageEvent",
          "event_type_h": "平台消息下发时",
          "type": "指令",
          "cmd": "/hello",
          "desc": "发送问候"
        }
      ],
      "logo": "/api/file/xxx"
    }
  ]
}
```

#### 获取插件市场列表

**端点**: `GET /api/plugin/market_list`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `force_refresh` | boolean | 否 | 强制刷新缓存 |
| `custom_registry` | string | 否 | 自定义插件源 URL |

**响应示例**:
```json
{
  "status": "ok",
  "data": [
    {
      "name": "astrbot_plugin_xxx",
      "repo": "https://github.com/xxx/xxx",
      "author": "xxx",
      "desc": "xxx",
      "version": "1.0.0",
      "tags": ["tool"],
      "logo_url": "https://..."
    }
  ]
}
```

#### 安装插件

**端点**: `POST /api/plugin/install`

**请求体**:
```json
{
  "url": "https://github.com/user/repo",
  "proxy": "https://ghproxy.com",
  "ignore_version_check": false
}
```

#### 上传安装插件

**端点**: `POST /api/plugin/install-upload`

**Content-Type**: `multipart/form-data`

**表单字段**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `file` | File | 插件 zip 文件 |
| `ignore_version_check` | string | "true" 或 "false" |

#### 更新插件

**端点**: `POST /api/plugin/update`

**请求体**:
```json
{
  "name": "astrbot_plugin_xxx",
  "proxy": "https://ghproxy.com"
}
```

#### 批量更新插件

**端点**: `POST /api/plugin/update-all`

**请求体**:
```json
{
  "names": ["plugin1", "plugin2"],
  "proxy": "https://ghproxy.com"
}
```

**响应示例**:
```json
{
  "status": "ok",
  "message": "批量更新完成",
  "data": {
    "results": [
      {"name": "plugin1", "status": "ok", "message": "更新成功"},
      {"name": "plugin2", "status": "error", "message": "..."}
    ]
  }
}
```

#### 卸载插件

**端点**: `POST /api/plugin/uninstall`

**请求体**:
```json
{
  "name": "astrbot_plugin_xxx",
  "delete_config": true,
  "delete_data": false
}
```

#### 启用/停用插件

**端点**: `POST /api/plugin/on`, `POST /api/plugin/off`

**请求体**:
```json
{
  "name": "astrbot_plugin_xxx"
}
```

#### 重载插件

**端点**: `POST /api/plugin/reload`

**请求体**:
```json
{
  "name": "astrbot_plugin_xxx"
}
```

### 1.6 聊天接口

#### 发送消息 (SSE)

**端点**: `POST /api/chat/send`

**Content-Type**: `application/json`

**请求体**:
```json
{
  "message": "你好",
  "session_id": "webchat-xxx",
  "selected_provider": "provider_id",
  "selected_model": "gpt-4",
  "enable_streaming": true
}
```

**响应**: SSE 流

```
data: {"type": "session_id", "data": "xxx"}

data: {"type": "plain", "data": "你好", "streaming": true}

data: {"type": "plain", "data": "！", "streaming": true}

data: {"type": "end", "streaming": false}
```

#### WebSocket 聊天

**端点**: `WS /api/unified_chat/ws`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `token` | string | 是 | JWT Token |

**消息格式 - 发送**:
```json
{
  "ct": "chat",
  "t": "send",
  "message_id": "msg_uuid",
  "session_id": "webchat-xxx",
  "message": [
    {"type": "plain", "text": "你好"}
  ],
  "selected_provider": "provider_id",
  "selected_model": "gpt-4",
  "enable_streaming": true
}
```

**消息格式 - 接收**:
```json
{
  "type": "plain",
  "data": "你好",
  "streaming": true,
  "session_id": "webchat-xxx",
  "message_id": "msg_uuid"
}
```

**消息格式 - 中断**:
```json
{
  "ct": "chat",
  "t": "interrupt",
  "session_id": "webchat-xxx",
  "message_id": "msg_uuid"
}
```

**消息格式 - 绑定会话**:
```json
{
  "ct": "chat",
  "t": "bind",
  "session_id": "webchat-xxx"
}
```

#### 获取会话列表

**端点**: `GET /api/chat/sessions`

**响应示例**:
```json
{
  "status": "ok",
  "data": [
    {
      "session_id": "webchat-xxx",
      "title": "新会话",
      "updated_at": "2024-01-01T00:00:00Z",
      "is_running": false
    }
  ]
}
```

#### 获取会话详情

**端点**: `GET /api/chat/get_session`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `session_id` | string | 是 | 会话 ID |

**响应示例**:
```json
{
  "status": "ok",
  "data": {
    "history": [
      {
        "id": 1,
        "content": {
          "type": "user",
          "message": [{"type": "plain", "text": "你好"}]
        },
        "created_at": "2024-01-01T00:00:00Z"
      },
      {
        "id": 2,
        "content": {
          "type": "bot",
          "message": [{"type": "plain", "text": "你好！"}],
          "reasoning": "思考过程...",
          "agent_stats": {
            "token_usage": {"input_other": 10, "input_cached": 0, "output": 5},
            "start_time": 1234567890,
            "end_time": 1234567895
          }
        }
      }
    ],
    "is_running": false
  }
}
```

#### 创建新会话

**端点**: `GET /api/chat/new_session`

**响应示例**:
```json
{
  "status": "ok",
  "data": {
    "session_id": "webchat-xxx"
  }
}
```

#### 删除会话

**端点**: `GET /api/chat/delete_session`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `session_id` | string | 是 | 会话 ID |

#### 停止会话

**端点**: `POST /api/chat/stop`

**请求体**:
```json
{
  "session_id": "webchat-xxx"
}
```

#### 上传文件

**端点**: `POST /api/chat/post_file`

**Content-Type**: `multipart/form-data`

**表单字段**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `file` | File | 上传的文件 |

**响应示例**:
```json
{
  "status": "ok",
  "data": {
    "attachment_id": "att_uuid",
    "filename": "image.png",
    "type": "image"
  }
}
```

#### 获取附件

**端点**: `GET /api/chat/get_attachment`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `attachment_id` | string | 是 | 附件 ID |

**响应**: 文件流

### 1.7 统计接口

#### 获取统计信息

**端点**: `GET /api/stat/get`

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `offset_sec` | int | 否 | 时间偏移（秒），默认 86400 |

**响应示例**:
```json
{
  "status": "ok",
  "data": {
    "message_count": 100,
    "platform_count": 2,
    "plugin_count": 5,
    "cpu_percent": 12.5,
    "memory": {
      "process": 256,
      "system": 16384
    },
    "running": {
      "hours": 1,
      "minutes": 30,
      "seconds": 0
    },
    "plugins": [
      {"name": "plugin1", "version": "1.0.0", "is_enabled": true}
    ]
  }
}
```

#### 获取版本信息

**端点**: `GET /api/stat/version`

**响应示例**:
```json
{
  "status": "ok",
  "data": {
    "version": "4.19.5",
    "dashboard_version": "1.0.0",
    "change_pwd_hint": false,
    "need_migration": false
  }
}
```

#### 重启核心

**端点**: `POST /api/stat/restart-core`

**响应示例**:
```json
{
  "status": "ok",
  "message": "正在重启..."
}
```

### 1.8 实时日志接口 (SSE)

**端点**: `GET /api/live-log`

**请求头**:
```
Authorization: Bearer <token>
```

**响应**: SSE 流

```
data: {"level": "INFO", "message": "AstrBot started", "timestamp": 1234567890}

data: {"level": "DEBUG", "message": "Debug info", "timestamp": 1234567891}
```

### 1.9 文件服务

#### 获取文件

**端点**: `GET /api/file/<token>`

通过文件令牌获取文件内容，用于插件 Logo 等。

### 1.10 MCP 服务器接口

MCP (Model Context Protocol) 服务器用于扩展 AstrBot 的工具调用能力。

#### 获取 MCP 服务器列表

**端点**: `GET /api/tools/mcp/servers`

**请求头**: `Authorization: Bearer <token>`

**响应示例**:
```json
{
  "status": "ok",
  "data": [
    {
      "name": "arxiv-server",
      "active": true,
      "command": "uv",
      "args": ["tool", "run", "arxiv-mcp-server"],
      "tools": ["search_papers", "get_paper"],
      "errlogs": []
    }
  ]
}
```

#### 添加 MCP 服务器

**端点**: `POST /api/tools/mcp/add`

**请求头**: `Authorization: Bearer <token>`

**Content-Type**: `application/json`

**请求体**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 服务器名称 |
| `active` | boolean | 否 | 是否启用（默认 true）|
| `command` | string | 条件 | 启动命令（如 `uv`, `npx`）|
| `args` | array | 否 | 命令参数 |
| `url` | string | 条件 | 远程服务器 URL（与 command 二选一）|
| `headers` | object | 否 | HTTP 请求头 |
| `transport` | string | 否 | 传输类型：`sse` 或 `streamable_http` |
| `env` | object | 否 | 环境变量 |

**Stdio 模式示例**（本地命令）:
```json
{
  "name": "arxiv-server",
  "active": true,
  "command": "uv",
  "args": ["tool", "run", "arxiv-mcp-server", "--storage-path", "data/arxiv"],
  "env": {
    "API_KEY": "xxx"
  }
}
```

**SSE 模式示例**（远程服务器）:
```json
{
  "name": "remote-mcp",
  "active": true,
  "url": "http://localhost:3000/sse",
  "transport": "sse",
  "headers": {
    "Authorization": "Bearer token"
  }
}
```

**响应示例**:
```json
{
  "status": "ok",
  "message": "Successfully added MCP server arxiv-server"
}
```

#### 更新 MCP 服务器

**端点**: `POST /api/tools/mcp/update`

**请求头**: `Authorization: Bearer <token>`

**请求体**: 同添加接口，可额外包含 `oldName` 字段用于重命名

```json
{
  "name": "arxiv-server-v2",
  "oldName": "arxiv-server",
  "active": true,
  "command": "uv",
  "args": ["tool", "run", "arxiv-mcp-server"]
}
```

#### 删除 MCP 服务器

**端点**: `POST /api/tools/mcp/delete`

**请求头**: `Authorization: Bearer <token>`

**请求体**:
```json
{
  "name": "arxiv-server"
}
```

#### 测试 MCP 连接

**端点**: `POST /api/tools/mcp/test`

**请求头**: `Authorization: Bearer <token>`

**请求体**:
```json
{
  "mcp_server_config": {
    "url": "http://localhost:3000/sse",
    "transport": "sse"
  }
}
```

**响应示例**:
```json
{
  "status": "ok",
  "data": ["tool1", "tool2"],
  "message": "🎉 MCP server is available!"
}
```

### 1.11 技能（工具）管理接口

技能指 AstrBot 可调用的函数工具，包括 MCP 工具、插件注册的工具等。

#### 获取技能列表

**端点**: `GET /api/tools/list`

**请求头**: `Authorization: Bearer <token>`

**响应示例**:
```json
{
  "status": "ok",
  "data": [
    {
      "name": "search_papers",
      "description": "Search papers on arXiv",
      "parameters": {
        "type": "object",
        "properties": {
          "query": {"type": "string"}
        }
      },
      "active": true,
      "origin": "mcp",
      "origin_name": "arxiv-server"
    },
    {
      "name": "web_search",
      "description": "Search web",
      "parameters": {...},
      "active": true,
      "origin": "plugin",
      "origin_name": "astrbot_plugin_web_search"
    }
  ]
}
```

**字段说明**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 技能名称 |
| `description` | string | 技能描述 |
| `parameters` | object | 参数 JSON Schema |
| `active` | boolean | 是否启用 |
| `origin` | string | 来源：`mcp` / `plugin` / `unknown` |
| `origin_name` | string | 来源名称（MCP 服务器名或插件名）|

#### 启用/禁用技能

**端点**: `POST /api/tools/toggle-tool`

**请求头**: `Authorization: Bearer <token>`

**请求体**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 技能名称 |
| `activate` | boolean | 是 | `true` 启用，`false` 禁用 |

```json
{
  "name": "search_papers",
  "activate": false
}
```

**响应示例**:
```json
{
  "status": "ok",
  "message": "Operation successful."
}
```

---

## 2. 配置文件格式详解

### 2.1 主配置文件

**路径**: `data/cmd_config.json`

**编码**: UTF-8 with BOM

**完整结构**:

```json
{
  "config_version": 2,

  "platform_settings": {
    "unique_session": false,
    "rate_limit": {
      "time": 60,
      "count": 30,
      "strategy": "stall"
    },
    "reply_prefix": "",
    "forward_threshold": 1500,
    "enable_id_white_list": true,
    "id_whitelist": [],
    "id_whitelist_log": true,
    "wl_ignore_admin_on_group": true,
    "wl_ignore_admin_on_friend": true,
    "reply_with_mention": false,
    "reply_with_quote": false,
    "path_mapping": [],
    "segmented_reply": {
      "enable": false,
      "only_llm_result": true,
      "interval_method": "random",
      "interval": "1.5,3.5",
      "log_base": 2.6,
      "words_count_threshold": 150,
      "split_mode": "regex",
      "regex": ".*?[。？！~…]+|.+$",
      "split_words": ["。", "？", "！", "~", "…"],
      "content_cleanup_rule": ""
    },
    "no_permission_reply": true,
    "empty_mention_waiting": true,
    "empty_mention_waiting_need_reply": true,
    "friend_message_needs_wake_prefix": false,
    "ignore_bot_self_message": false,
    "ignore_at_all": false
  },

  "provider_sources": [
    {
      "id": "openai_source",
      "type": "openai",
      "enable": true,
      "api_key": "sk-xxx",
      "base_url": "https://api.openai.com/v1",
      "model": "gpt-4"
    }
  ],

  "provider": [
    {
      "id": "gpt4",
      "provider_source_id": "openai_source",
      "model": "gpt-4",
      "modalities": ["text", "image"],
      "custom_extra_body": {},
      "max_context_tokens": 0
    }
  ],

  "provider_settings": {
    "enable": true,
    "default_provider_id": "",
    "fallback_chat_models": [],
    "default_image_caption_provider_id": "",
    "image_caption_prompt": "Please describe the image using Chinese.",
    "provider_pool": ["*"],
    "wake_prefix": "",
    "web_search": false,
    "websearch_provider": "default",
    "websearch_tavily_key": [],
    "websearch_bocha_key": [],
    "websearch_baidu_app_builder_key": "",
    "web_search_link": false,
    "display_reasoning_text": false,
    "identifier": false,
    "group_name_display": false,
    "datetime_system_prompt": true,
    "default_personality": "default",
    "persona_pool": ["*"],
    "prompt_prefix": "{{prompt}}",
    "context_limit_reached_strategy": "truncate_by_turns",
    "llm_compress_instruction": "...",
    "llm_compress_keep_recent": 6,
    "llm_compress_provider_id": "",
    "max_context_length": -1,
    "dequeue_context_length": 1,
    "streaming_response": false,
    "show_tool_use_status": false,
    "show_tool_call_result": false,
    "sanitize_context_by_modalities": false,
    "max_quoted_fallback_images": 20,
    "quoted_message_parser": {
      "max_component_chain_depth": 4,
      "max_forward_node_depth": 6,
      "max_forward_fetch": 32,
      "warn_on_action_failure": false
    },
    "agent_runner_type": "local",
    "dify_agent_runner_provider_id": "",
    "coze_agent_runner_provider_id": "",
    "dashscope_agent_runner_provider_id": "",
    "deerflow_agent_runner_provider_id": "",
    "unsupported_streaming_strategy": "realtime_segmenting",
    "reachability_check": false,
    "max_agent_step": 30,
    "tool_call_timeout": 60,
    "tool_schema_mode": "full",
    "llm_safety_mode": true,
    "safety_mode_strategy": "system_prompt",
    "file_extract": {
      "enable": false,
      "provider": "moonshotai",
      "moonshotai_api_key": ""
    },
    "proactive_capability": {
      "add_cron_tools": true
    },
    "computer_use_runtime": "none",
    "computer_use_require_admin": true,
    "sandbox": {
      "booter": "shipyard_neo",
      "shipyard_endpoint": "",
      "shipyard_access_token": "",
      "shipyard_ttl": 3600,
      "shipyard_max_sessions": 10,
      "shipyard_neo_endpoint": "",
      "shipyard_neo_access_token": "",
      "shipyard_neo_profile": "python-default",
      "shipyard_neo_ttl": 3600
    }
  },

  "subagent_orchestrator": {
    "main_enable": false,
    "remove_main_duplicate_tools": false,
    "router_system_prompt": "...",
    "agents": []
  },

  "provider_stt_settings": {
    "enable": false,
    "provider_id": ""
  },

  "provider_tts_settings": {
    "enable": false,
    "provider_id": "",
    "dual_output": false,
    "use_file_service": false,
    "trigger_probability": 1.0
  },

  "provider_ltm_settings": {
    "group_icl_enable": false,
    "group_message_max_cnt": 300,
    "image_caption": false,
    "image_caption_provider_id": "",
    "active_reply": {
      "enable": false,
      "method": "possibility_reply",
      "possibility_reply": 0.1,
      "whitelist": []
    }
  },

  "content_safety": {
    "also_use_in_response": false,
    "internal_keywords": {
      "enable": true,
      "extra_keywords": []
    },
    "baidu_aip": {
      "enable": false,
      "app_id": "",
      "api_key": "",
      "secret_key": ""
    }
  },

  "admins_id": ["astrbot"],

  "t2i": false,
  "t2i_word_threshold": 150,
  "t2i_strategy": "remote",
  "t2i_endpoint": "",
  "t2i_use_file_service": false,
  "t2i_active_template": "base",

  "http_proxy": "",
  "no_proxy": ["localhost", "127.0.0.1", "::1", "10.*", "192.168.*"],

  "dashboard": {
    "enable": true,
    "username": "astrbot",
    "password": "77b90590a8945a7d36c963981a307dc9",
    "jwt_secret": "",
    "host": "0.0.0.0",
    "port": 6185,
    "disable_access_log": true,
    "ssl": {
      "enable": false,
      "cert_file": "",
      "key_file": "",
      "ca_certs": ""
    }
  },

  "platform": [
    {
      "id": "default",
      "type": "qq_official",
      "enable": false,
      "appid": "",
      "secret": "",
      "enable_group_c2c": true,
      "enable_guild_direct_message": true
    }
  ],

  "platform_specific": {
    "lark": {
      "pre_ack_emoji": {
        "enable": false,
        "emojis": ["Typing"]
      }
    },
    "telegram": {
      "pre_ack_emoji": {
        "enable": false,
        "emojis": ["✍️"]
      }
    }
  },

  "wake_prefix": ["/"],
  "log_level": "INFO",
  "log_file_enable": false,
  "log_file_path": "logs/astrbot.log",
  "log_file_max_mb": 20,
  "temp_dir_max_size": 1024,
  "trace_enable": false,
  "trace_log_enable": false,
  "trace_log_path": "logs/astrbot.trace.log",
  "trace_log_max_mb": 20,
  "pip_install_arg": "",
  "pypi_index_url": "https://mirrors.aliyun.com/pypi/simple/",
  "timezone": "Asia/Shanghai",
  "callback_api_base": "",
  "default_kb_collection": "",
  "plugin_set": ["*"],
  "kb_names": [],
  "kb_fusion_top_k": 20,
  "kb_final_top_k": 5,
  "kb_agentic_mode": false,
  "disable_builtin_commands": false
}
```

### 2.2 配置字段说明

#### 平台设置 (platform_settings)

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `unique_session` | boolean | false | 是否启用唯一会话 |
| `rate_limit.time` | int | 60 | 速率限制时间窗口（秒） |
| `rate_limit.count` | int | 30 | 速率限制次数 |
| `rate_limit.strategy` | string | "stall" | 速率限制策略：stall/discard |
| `reply_prefix` | string | "" | 回复前缀 |
| `forward_threshold` | int | 1500 | 转发阈值（字符数） |
| `enable_id_white_list` | boolean | true | 启用 ID 白名单 |
| `id_whitelist` | array | [] | ID 白名单列表 |
| `segmented_reply.enable` | boolean | false | 启用分段回复 |
| `segmented_reply.interval` | string | "1.5,3.5" | 分段间隔（秒） |
| `segmented_reply.split_mode` | string | "regex" | 分段模式：regex/words |
| `segmented_reply.regex` | string | ".*?[。？！~…]+|.+$" | 分段正则表达式 |
| `reply_with_mention` | boolean | false | 回复时 @ 用户 |
| `reply_with_quote` | boolean | false | 回复时引用消息 |
| `friend_message_needs_wake_prefix` | boolean | false | 私聊需要唤醒前缀 |
| `ignore_bot_self_message` | boolean | false | 忽略机器人自己的消息 |
| `ignore_at_all` | boolean | false | 忽略 @全体成员 |

#### 提供商源 (provider_sources)

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 提供商源唯一标识 |
| `type` | string | 是 | 提供商类型：openai/anthropic/gemini/ollama 等 |
| `enable` | boolean | 是 | 是否启用 |
| `api_key` | string | 条件 | API 密钥 |
| `base_url` | string | 条件 | 基础 URL |
| `model` | string | 否 | 默认模型 |

#### 模型实例 (provider)

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 模型实例唯一标识 |
| `provider_source_id` | string | 是 | 关联的提供商源 ID |
| `model` | string | 是 | 模型名称 |
| `modalities` | array | 否 | 支持的功能：["text", "image"] |
| `custom_extra_body` | object | 否 | 自定义请求体 |
| `max_context_tokens` | int | 否 | 最大上下文 Token 数 |

#### 提供商设置 (provider_settings)

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enable` | boolean | true | 启用 LLM 处理 |
| `default_provider_id` | string | "" | 默认提供商 ID |
| `fallback_chat_models` | array | [] | 失败时回退的模型列表 |
| `wake_prefix` | string | "" | 唤醒前缀 |
| `web_search` | boolean | false | 启用网络搜索 |
| `websearch_provider` | string | "default" | 搜索提供商 |
| `websearch_tavily_key` | array | [] | Tavily API 密钥列表 |
| `websearch_bocha_key` | array | [] | Bocha API 密钥列表 |
| `display_reasoning_text` | boolean | false | 显示推理文本 |
| `identifier` | boolean | false | 显示身份标识 |
| `group_name_display` | boolean | false | 显示群名称 |
| `datetime_system_prompt` | boolean | true | 使用日期时间系统提示词 |
| `default_personality` | string | "default" | 默认人设 |
| `context_limit_reached_strategy` | string | "truncate_by_turns" | 上下文限制策略 |
| `max_context_length` | int | -1 | 最大上下文长度，-1 表示不限制 |
| `streaming_response` | boolean | false | 流式响应 |
| `show_tool_use_status` | boolean | false | 显示工具使用状态 |
| `show_tool_call_result` | boolean | false | 显示工具调用结果 |
| `agent_runner_type` | string | "local" | Agent 运行器类型 |
| `max_agent_step` | int | 30 | 最大 Agent 步骤数 |
| `tool_call_timeout` | int | 60 | 工具调用超时时间 |
| `tool_schema_mode` | string | "full" | 工具 Schema 模式 |
| `llm_safety_mode` | boolean | true | LLM 安全模式 |

#### 语音设置

**STT (语音转文字)**:
```json
{
  "enable": false,
  "provider_id": ""
}
```

**TTS (文字转语音)**:
```json
{
  "enable": false,
  "provider_id": "",
  "dual_output": false,
  "use_file_service": false,
  "trigger_probability": 1.0
}
```

#### 长期记忆设置 (provider_ltm_settings)

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `group_icl_enable` | boolean | false | 启用群聊 ICL |
| `group_message_max_cnt` | int | 300 | 群消息最大数量 |
| `image_caption` | boolean | false | 图片自动描述 |
| `active_reply.enable` | boolean | false | 主动回复 |
| `active_reply.method` | string | "possibility_reply" | 主动回复方式 |
| `active_reply.possibility_reply` | float | 0.1 | 概率回复几率 |

#### 内容安全 (content_safety)

```json
{
  "also_use_in_response": false,
  "internal_keywords": {
    "enable": true,
    "extra_keywords": []
  },
  "baidu_aip": {
    "enable": false,
    "app_id": "",
    "api_key": "",
    "secret_key": ""
  }
}
```

#### Dashboard 设置

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enable` | boolean | true | 启用 WebUI |
| `username` | string | "astrbot" | 登录用户名 |
| `password` | string | MD5 值 | 登录密码（MD5 加密） |
| `jwt_secret` | string | "" | JWT 密钥（为空时自动生成） |
| `host` | string | "0.0.0.0" | 监听地址 |
| `port` | int | 6185 | 监听端口 |
| `disable_access_log` | boolean | true | 禁用访问日志 |
| `ssl.enable` | boolean | false | 启用 SSL |
| `ssl.cert_file` | string | "" | SSL 证书路径 |
| `ssl.key_file` | string | "" | SSL 私钥路径 |

#### 平台适配器 (platform)

平台适配器是一个数组，每个元素表示一个平台配置：

**QQ 官方 (WebSocket)**:
```json
{
  "id": "default",
  "type": "qq_official",
  "enable": false,
  "appid": "",
  "secret": "",
  "enable_group_c2c": true,
  "enable_guild_direct_message": true
}
```

**QQ 官方 (Webhook)**:
```json
{
  "id": "default",
  "type": "qq_official_webhook",
  "enable": false,
  "appid": "",
  "secret": "",
  "is_sandbox": false,
  "unified_webhook_mode": true,
  "webhook_uuid": "",
  "callback_server_host": "0.0.0.0",
  "port": 6196
}
```

**OneBot v11 (aiocqhttp)**:
```json
{
  "id": "default",
  "type": "aiocqhttp",
  "enable": false,
  "ws_reverse_host": "0.0.0.0",
  "ws_reverse_port": 6199,
  "ws_reverse_token": ""
}
```

**微信公众号**:
```json
{
  "id": "weixin_official_account",
  "type": "weixin_official_account",
  "enable": false,
  "appid": "",
  "secret": "",
  "token": "",
  "encoding_aes_key": "",
  "api_base_url": "https://api.weixin.qq.com/cgi-bin/",
  "unified_webhook_mode": true,
  "webhook_uuid": "",
  "callback_server_host": "0.0.0.0",
  "port": 6194,
  "active_send_mode": false
}
```

**Telegram**:
```json
{
  "id": "telegram",
  "type": "telegram",
  "enable": false,
  "telegram_token": "your_bot_token",
  "start_message": "Hello, I'm AstrBot!",
  "telegram_api_base_url": "https://api.telegram.org/bot",
  "telegram_file_base_url": "https://api.telegram.org/file/bot",
  "telegram_command_register": true,
  "telegram_command_auto_refresh": true,
  "telegram_command_register_interval": 300
}
```

**Discord**:
```json
{
  "id": "discord",
  "type": "discord",
  "enable": false,
  "discord_bot_token": "your_bot_token",
  "discord_gateway_intents": 32511
}
```

### 2.2 插件配置文件

插件可以定义自己的配置文件，路径为 `data/config/<plugin_name>.json`。

插件配置通过装饰器定义：

```python
from astrbot.api import AstrBotConfig

@register("astrbot_plugin_example", "Author", "Description", "1.0.0")
class MyPlugin(Star):
    def __init__(self, context: Context, config: AstrBotConfig):
        super().__init__(context)
        self.config = config
        # 访问配置
        api_key = config.get("api_key", "")
```

### 2.4 MCP 服务器配置文件

**路径**: `data/mcp_config.json`

**说明**: MCP (Model Context Protocol) 服务器配置用于扩展 AstrBot 的工具调用能力。

**完整结构**:

```json
{
  "mcpServers": {
    "arxiv-server": {
      "active": true,
      "command": "uv",
      "args": ["tool", "run", "arxiv-mcp-server", "--storage-path", "data/arxiv"]
    },
    "github-server": {
      "active": true,
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxx"
      }
    },
    "remote-sse-server": {
      "active": true,
      "url": "http://localhost:3000/sse",
      "transport": "sse",
      "headers": {
        "Authorization": "Bearer token"
      }
    },
    "remote-http-server": {
      "active": true,
      "url": "http://localhost:3000/mcp",
      "transport": "streamable_http",
      "headers": {},
      "timeout": 30
    }
  }
}
```

**配置字段说明**:

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `active` | boolean | 否 | 是否启用该服务器（默认 true）|
| `command` | string | 条件 | 启动命令（如 `uv`, `npx`, `node`），本地_stdio_模式必填 |
| `args` | array | 否 | 命令参数 |
| `env` | object | 否 | 环境变量 |
| `url` | string | 条件 | 远程服务器 URL，远程模式必填 |
| `transport` | string | 条件 | 传输类型：`sse` 或 `streamable_http`，远程模式必填 |
| `headers` | object | 否 | HTTP 请求头（远程模式）|
| `timeout` | number | 否 | 连接超时时间（秒）|
| `sse_read_timeout` | number | 否 | SSE 读取超时时间（秒）|

**模式说明**:

1. **Stdio 模式**（本地命令）: 配置 `command` 和 `args`
2. **SSE 模式**（远程）: 配置 `url` 和 `transport: "sse"`
3. **Streamable HTTP 模式**（远程）: 配置 `url` 和 `transport: "streamable_http"`

---

## 3. 消息传输协议

### 3.1 SSE (Server-Sent Events) 协议

SSE 用于流式返回聊天响应和日志。

**连接建立**:
```
GET /api/chat/send HTTP/1.1
Content-Type: application/json
Authorization: Bearer <token>

{ "message": "...", "session_id": "..." }
```

**数据格式**:
每行以 `data: ` 开头，后跟 JSON 对象，以两个换行符结束。

```
data: {"type": "session_id", "data": "xxx"}

data: {"type": "plain", "data": "你好", "streaming": true}

data: {"type": "end", "streaming": false}
```

**消息类型**:

| 类型 | 说明 | 示例 |
|------|------|------|
| `session_id` | 会话 ID | `{"type": "session_id", "data": "webchat-xxx"}` |
| `plain` | 普通文本 | `{"type": "plain", "data": "内容", "chain_type": "normal"}` |
| `image` | 图片 | `{"type": "image", "data": "[IMAGE]filename.png"}` |
| `record` | 语音 | `{"type": "record", "data": "[RECORD]filename.wav"}` |
| `file` | 文件 | `{"type": "file", "data": "[FILE]filename.pdf"}` |
| `tool_call` | 工具调用 | `{"type": "plain", "chain_type": "tool_call", "data": "{...}"}` |
| `tool_call_result` | 工具结果 | `{"type": "plain", "chain_type": "tool_call_result", "data": "{...}"}` |
| `reasoning` | 推理内容 | `{"type": "plain", "chain_type": "reasoning", "data": "..."}` |
| `update_title` | 更新标题 | `{"type": "update_title", "session_id": "...", "data": "新标题"}` |
| `message_saved` | 消息已保存 | `{"type": "message_saved", "data": {"id": 1, "created_at": "..."}}` |
| `agent_stats` | Agent 统计 | `{"type": "agent_stats", "data": {...}}` |
| `end` | 结束标记 | `{"type": "end", "streaming": false}` |
| `error` | 错误 | `{"type": "error", "data": "错误信息"}` |

### 3.2 WebSocket 协议

WebSocket 用于双向实时通信。

**连接建立**:
```
WS /api/unified_chat/ws?token=<jwt_token>
```

**客户端消息格式**:

所有消息包含 `ct` (channel type) 和 `t` (type) 字段：

**发送消息**:
```json
{
  "ct": "chat",
  "t": "send",
  "message_id": "msg_uuid",
  "session_id": "webchat-xxx",
  "message": [
    {"type": "plain", "text": "你好"},
    {"type": "image", "attachment_id": "att_uuid"}
  ],
  "selected_provider": "provider_id",
  "selected_model": "gpt-4",
  "enable_streaming": true
}
```

**绑定会话**:
```json
{
  "ct": "chat",
  "t": "bind",
  "session_id": "webchat-xxx"
}
```

**中断生成**:
```json
{
  "ct": "chat",
  "t": "interrupt",
  "session_id": "webchat-xxx",
  "message_id": "msg_uuid"
}
```

**服务端消息格式**:

```json
{
  "ct": "chat",
  "type": "plain",
  "data": "你好",
  "chain_type": "normal",
  "streaming": true,
  "session_id": "webchat-xxx",
  "message_id": "msg_uuid"
}
```

**会话绑定确认**:
```json
{
  "ct": "chat",
  "type": "session_bound",
  "session_id": "webchat-xxx"
}
```

**错误消息**:
```json
{
  "ct": "chat",
  "t": "error",
  "code": "ERROR_CODE",
  "data": "错误信息",
  "session_id": "webchat-xxx",
  "message_id": "msg_uuid"
}
```

### 3.3 消息 Part 结构

消息由多个 `part` 组成，每个 part 代表消息的一部分：

**纯文本**:
```json
{
  "type": "plain",
  "text": "文本内容"
}
```

**图片**:
```json
{
  "type": "image",
  "attachment_id": "att_uuid"
}
```

**语音**:
```json
{
  "type": "record",
  "attachment_id": "att_uuid"
}
```

**文件**:
```json
{
  "type": "file",
  "attachment_id": "att_uuid",
  "filename": "document.pdf"
}
```

**引用回复**:
```json
{
  "type": "reply",
  "message_id": 123,
  "selected_text": "引用的文本"
}
```

**视频**:
```json
{
  "type": "video",
  "attachment_id": "att_uuid"
}
```

**工具调用**:
```json
{
  "type": "tool_call",
  "tool_calls": [
    {
      "id": "call_xxx",
      "name": "tool_name",
      "args": {"arg1": "value1"},
      "ts": 1234567890,
      "result": "调用结果",
      "finished_ts": 1234567895
    }
  ]
}
```

---

## 4. 数据模型定义

### 4.1 TypeScript 类型定义 (前端)

**消息 Part**:
```typescript
interface MessagePart {
  type: 'plain' | 'image' | 'record' | 'file' | 'video' | 'reply' | 'tool_call';
  text?: string;           // for plain
  attachment_id?: string;  // for image, record, file, video
  filename?: string;       // for file
  message_id?: number;     // for reply
  tool_calls?: ToolCall[]; // for tool_call
  embedded_url?: string;   // 加载后填充
  embedded_file?: FileInfo;
  selected_text?: string;  // for reply
}
```

**工具调用**:
```typescript
interface ToolCall {
  id: string;
  name: string;
  args: Record<string, any>;
  ts: number;
  result?: string;
  finished_ts?: number;
}
```

**Token 使用统计**:
```typescript
interface TokenUsage {
  input_other: number;
  input_cached: number;
  output: number;
}

interface AgentStats {
  token_usage: TokenUsage;
  start_time: number;
  end_time: number;
  time_to_first_token: number;
}
```

**消息内容**:
```typescript
interface MessageContent {
  type: 'user' | 'bot';
  message: MessagePart[];
  reasoning?: string;
  isLoading?: boolean;
  agentStats?: AgentStats;
}

interface Message {
  id?: number;
  content: MessageContent;
  created_at?: string;
}
```

### 4.2 Python 模型定义 (后端)

**会话模型** (SQLAlchemy):
```python
class Conversation:
    __tablename__ = 'conversations'

    conversation_id = Column(String, primary_key=True)
    title = Column(String)
    updated_at = Column(DateTime)
    is_running = Column(Boolean, default=False)
    project_id = Column(String, nullable=True)
```

**消息历史模型**:
```python
class PlatformSessionHistory:
    __tablename__ = 'platform_session_history'

    id = Column(Integer, primary_key=True, autoincrement=True)
    platform_id = Column(String)
    user_id = Column(String)
    content = Column(JSON)  # MessageContent 字典
    sender_id = Column(String)
    sender_name = Column(String)
    created_at = Column(DateTime, default=datetime.now)
```

**附件模型**:
```python
class Attachment:
    __tablename__ = 'attachments'

    attachment_id = Column(String, primary_key=True)
    path = Column(String)
    type = Column(String)  # image, record, file, video
    mime_type = Column(String)
    created_at = Column(DateTime, default=datetime.now)
```

**MCP 服务器模型**:
```python
class MCPServer:
    name: str                    # 服务器名称
    active: bool                 # 是否启用
    command: str                 # 启动命令（本地模式）
    args: List[str]              # 命令参数
    env: Dict[str, str]          # 环境变量
    url: str                     # 远程 URL（远程模式）
    transport: str               # 传输类型: sse/streamable_http
    headers: Dict[str, str]      # HTTP 请求头
    timeout: int                 # 超时时间
    tools: List[str]             # 可用工具列表
    errlogs: List[str]           # 错误日志
```

**技能（工具）模型**:
```python
class FunctionTool:
    name: str                    # 工具名称
    description: str             # 工具描述
    parameters: dict             # 参数 JSON Schema
    active: bool                 # 是否启用
    origin: str                  # 来源: mcp/plugin/unknown
    origin_name: str             # 来源名称
```

### 4.3 配置元数据类型

配置系统支持以下类型 (`DEFAULT_VALUE_MAP`):

| 类型 | 默认值 | 说明 |
|------|--------|------|
| `int` | 0 | 整数 |
| `float` | 0.0 | 浮点数 |
| `bool` | false | 布尔值 |
| `string` | "" | 字符串 |
| `text` | "" | 长文本 |
| `list` | [] | 列表/数组 |
| `file` | [] | 文件列表 |
| `object` | {} | 对象/字典 |
| `template_list` | [] | 模板列表 |

---

## 附录

### A. 错误码列表

| 错误码 | 说明 |
|--------|------|
| `INTERRUPTED` | 用户中断 |
| `TIMEOUT` | 超时 |
| `PROVIDER_ERROR` | 提供商错误 |
| `CONFIG_ERROR` | 配置错误 |

### B. 平台类型列表

| 类型 | 名称 |
|------|------|
| `qq_official` | QQ 官方 Bot (WebSocket) |
| `qq_official_webhook` | QQ 官方 Bot (Webhook) |
| `aiocqhttp` | OneBot v11 |
| `weixin_official_account` | 微信公众号 |
| `wecom` | 企业微信 |
| `wecom_ai_bot` | 企业微信智能机器人 |
| `lark` | 飞书 |
| `dingtalk` | 钉钉 |
| `telegram` | Telegram |
| `discord` | Discord |
| `slack` | Slack |
| `line` | LINE |

### C. 提供商类型列表

| 类型 | 说明 |
|------|------|
| `openai` | OpenAI API |
| `anthropic` | Anthropic Claude |
| `gemini` | Google Gemini |
| `ollama` | Ollama 本地部署 |
| `dashscope` | 阿里云 DashScope |
| `dify` | Dify API |
| `coze` | Coze API |
| `xai` | xAI Grok |
