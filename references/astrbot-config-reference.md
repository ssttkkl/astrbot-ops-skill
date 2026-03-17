# AstrBot 配置文件参考

## 配置文件位置

### 主要配置文件（容器内路径）
- **主配置**: `/AstrBot/data/cmd_config.json` - AstrBot 核心配置
- **Docker Compose**: `/opt/astrbot/docker-compose.yml` - 容器编排配置（宿主机）
- **Bay 配置**: `/app/config.yaml` - Shipyard Neo Bay 沙箱配置
- **NapCat 配置**: `/app/napcat/config/napcat.json` - QQ 协议配置

### 数据目录（容器内路径）
- **数据根目录**: `/AstrBot/data/`
- **插件配置**: `/AstrBot/data/config/`
- **插件数据**: `/AstrBot/data/plugin_data/`
- **技能目录**: `/AstrBot/data/skills/`
- **知识库**: `/AstrBot/data/knowledge_base/`
- **数据库**: `/AstrBot/data/data_v4.db`

### 宿主机路径映射
- **宿主机数据目录**: `/opt/astrbot/data/` → 容器内 `/AstrBot/data/`
- **宿主机 Bay 配置**: `/opt/astrbot/config.yaml` → 容器内 `/app/config.yaml`
- **宿主机 NapCat 配置**: `/opt/astrbot/napcat/config/` → 容器内 `/app/napcat/config/`

---

## Docker Compose 配置 (`docker-compose.yml`)

### 服务架构

```yaml
services:
  napcat:      # QQ 协议服务
  astrbot:     # AstrBot 主服务
  bay:         # Shipyard Neo Bay 沙箱
```

### 端口映射

| 服务 | 端口 | 说明 |
|------|------|------|
| napcat | 6099 | QQ 协议 WebSocket |
| astrbot | 6185 | Web 控制台 |
| bay | 8114 | 沙箱运行时 API |

### NapCat 配置

```yaml
napcat:
  environment:
    - NAPCAT_UID=1000
    - NAPCAT_GID=1000
    - MODE=astrbot
  ports:
    - "6099:6099"
  volumes:
    - ./data:/AstrBot/data
    - ./napcat/config:/app/napcat/config
    - ./ntqq:/app/.config/QQ
```

### AstrBot 配置

```yaml
astrbot:
  environment:
    - TZ=Asia/Shanghai
    - BAY_DATA_DIR=/bay-data
  image: ghcr.io/ssttkkl/astrbot-with-nsjail:latest
  privileged: true
  ports:
    - "6185:6185"
  volumes:
    - ${PWD}/data:/AstrBot/data
    - bay-data:/bay-data:ro
```

### Bay 配置

```yaml
bay:
  image: ghcr.io/astrbotdevs/shipyard-neo-bay:latest
  ports:
    - "8114:8114"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./config.yaml:/app/config.yaml:ro
    - bay-data:/app/data
    - bay-cargos:/var/lib/bay/cargos
  environment:
    - BAY_CONFIG_FILE=/app/config.yaml
    - BAY_DATA_DIR=/app/data
    - BAY_API_KEY=access-token
```

---

## 主配置文件 (`cmd_config.json`)

### 1. 平台设置 (platform_settings)

```json
{
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
    "reply_with_mention": false,
    "reply_with_quote": true
  }
}
```

**字段说明**:
- `unique_session`: 是否为每个用户创建独立会话
- `rate_limit`: 速率限制配置
  - `time`: 时间窗口（秒）
  - `count`: 最大请求数
  - `strategy`: 限流策略（`stall` 或 `reject`）
- `forward_threshold`: 消息转发阈值（字符数）
- `enable_id_white_list`: 是否启用白名单
- `reply_with_quote`: 是否引用回复

### 2. 模型提供商配置 (provider_sources)

```json
{
  "provider_sources": [
    {
      "provider": "openai",
      "type": "openai_chat_completion",
      "provider_type": "chat_completion",
      "key": ["sk-xxx"],
      "api_base": "http://172.19.0.1:8317/v1",
      "timeout": 120,
      "proxy": "",
      "custom_headers": {},
      "id": "openai",
      "enable": true
    }
  ]
}
```

**字段说明**:
- `provider`: 提供商名称
- `type`: API 类型
- `key`: API 密钥列表（支持多个密钥轮换）
- `api_base`: API 基础 URL
- `timeout`: 请求超时时间（秒）
- `proxy`: 代理地址
- `enable`: 是否启用

### 3. 模型配置 (provider)

```json
{
  "provider": [
    {
      "id": "openai/claude-sonnet-4-6",
      "enable": true,
      "provider_source_id": "openai",
      "model": "claude-sonnet-4-6",
      "modalities": ["text", "image", "tool_use"],
      "custom_extra_body": {},
      "max_context_tokens": 200000
    }
  ]
}
```

**字段说明**:
- `id`: 模型唯一标识符（格式：`provider/model-name`）
- `provider_source_id`: 关联的提供商 ID
- `model`: 模型名称
- `modalities`: 支持的模态（text, image, tool_use 等）
- `max_context_tokens`: 最大上下文 token 数

### 4. 模型设置 (provider_settings)

```json
{
  "provider_settings": {
    "enable": true,
    "default_provider_id": "openai/claude-sonnet-4-6",
    "fallback_chat_models": ["openai/Kimi-K2.5"],
    "web_search": true,
    "websearch_provider": "tavily",
    "websearch_tavily_key": ["tvly-xxx"],
    "streaming_response": true,
    "max_context_length": -1,
    "context_limit_reached_strategy": "llm_compress"
  }
}
```

**字段说明**:
- `default_provider_id`: 默认使用的模型
- `fallback_chat_models`: 备用模型列表
- `web_search`: 是否启用网络搜索
- `websearch_provider`: 搜索提供商（`tavily`, `bocha`, `baidu`）
- `streaming_response`: 是否启用流式响应
- `context_limit_reached_strategy`: 上下文超限策略
  - `llm_compress`: LLM 压缩
  - `truncate`: 截断
  - `reject`: 拒绝

### 5. 沙箱配置 (sandbox)

```json
{
  "sandbox": {
    "booter": "shipyard_neo",
    "shipyard_neo_endpoint": "http://172.19.0.1:8114",
    "shipyard_neo_access_token": "access-token",
    "shipyard_neo_profile": "python-default",
    "shipyard_neo_ttl": 3600
  }
}
```

**字段说明**:
- `booter`: 沙箱启动器类型
- `shipyard_neo_endpoint`: Bay 服务地址
- `shipyard_neo_access_token`: 访问令牌
- `shipyard_neo_profile`: 使用的沙箱配置文件
- `shipyard_neo_ttl`: 会话超时时间（秒）

### 6. 管理员配置 (admins_id)

```json
{
  "admins_id": ["astrbot", "1098042114"]
}
```

管理员 QQ 号列表，拥有完整权限。

### 7. 控制台配置 (dashboard)

```json
{
  "dashboard": {
    "enable": true,
    "username": "astrbot",
    "password": "hashed-password",
    "jwt_secret": "random-secret",
    "host": "0.0.0.0",
    "port": 6185,
    "disable_access_log": true
  }
}
```

**字段说明**:
- `enable`: 是否启用 Web 控制台
- `username`: 登录用户名
- `password`: 密码（MD5 哈希）
- `jwt_secret`: JWT 密钥
- `host`: 监听地址
- `port`: 监听端口

### 8. 平台连接配置 (platform)

```json
{
  "platform": [
    {
      "id": "default",
      "type": "aiocqhttp",
      "enable": true,
      "ws_reverse_host": "0.0.0.0",
      "ws_reverse_port": 6199,
      "ws_reverse_token": ""
    }
  ]
}
```

**字段说明**:
- `type`: 平台类型（`aiocqhttp` 对应 NapCat）
- `ws_reverse_host`: WebSocket 反向连接监听地址
- `ws_reverse_port`: WebSocket 反向连接端口
- `ws_reverse_token`: 连接令牌

---

## Bay 配置文件 (`config.yaml`)

### 1. 沙箱配置文件 (profiles)

```yaml
profiles:
- id: python-default
  description: Python sandbox with uv and playwright support
  image: ghcr.io/ssttkkl/shipyard-neo-ship-custom:main
  runtime_type: ship
  runtime_port: 8123
  capabilities:
  - filesystem
  - shell
  - python
  resources:
    cpus: 1.0
    memory: 512m
  idle_timeout: 1800
  warm_pool_size: 1
  env:
    PLAYWRIGHT_BROWSERS_PATH: /root/.cache/ms-playwright
    UV_CACHE_DIR: /workspace/.uv-cache
```

**字段说明**:
- `id`: 配置文件 ID
- `image`: Docker 镜像
- `runtime_type`: 运行时类型（`ship`）
- `capabilities`: 能力列表（filesystem, shell, python 等）
- `resources`: 资源限制
- `idle_timeout`: 空闲超时（秒）
- `warm_pool_size`: 预热池大小

### 2. 驱动配置 (driver)

```yaml
driver:
  type: docker
  docker:
    connect_mode: container_network
    network: astrbot_astrbot_network
    socket: unix:///var/run/docker.sock
  image_pull_policy: always
```

### 3. 安全配置 (security)

```yaml
security:
  allow_anonymous: false
  api_key: your-api-key
```

### 4. 垃圾回收配置 (gc)

```yaml
gc:
  enabled: true
  interval_seconds: 300
  run_on_startup: true
  expired_sandbox:
    enabled: true
  idle_session:
    enabled: true
  orphan_cargo:
    enabled: true
  orphan_container:
    enabled: true
```

---

## NapCat 配置 (`napcat.json`)

```json
{
  "fileLog": false,
  "consoleLog": true,
  "fileLogLevel": "debug",
  "consoleLogLevel": "info",
  "packetBackend": "auto",
  "o3HookMode": 1
}
```

**字段说明**:
- `fileLog`: 是否启用文件日志
- `consoleLog`: 是否启用控制台日志
- `consoleLogLevel`: 控制台日志级别
- `packetBackend`: 数据包后端（`auto`, `kritor`, `chronocat`）
- `o3HookMode`: Hook 模式

---

## 常用配置修改场景

### 1. 添加新的 LLM 模型

编辑 `/opt/astrbot/cmd_config.json`：

```json
{
  "provider": [
    {
      "id": "openai/new-model",
      "enable": true,
      "provider_source_id": "openai",
      "model": "new-model",
      "modalities": ["text", "image"],
      "max_context_tokens": 128000
    }
  ]
}
```

### 2. 修改默认模型

```json
{
  "provider_settings": {
    "default_provider_id": "openai/claude-sonnet-4-6"
  }
}
```

### 3. 配置网络搜索

```json
{
  "provider_settings": {
    "web_search": true,
    "websearch_provider": "tavily",
    "websearch_tavily_key": ["your-api-key"]
  }
}
```

### 4. 修改沙箱配置

编辑 `/opt/astrbot/config.yaml`：

```yaml
profiles:
- id: python-default
  resources:
    cpus: 2.0
    memory: 1024m
  idle_timeout: 3600
```

### 5. 添加管理员

```json
{
  "admins_id": ["astrbot", "1098042114", "new-admin-qq"]
}
```

### 6. 修改 Web 控制台端口

```json
{
  "dashboard": {
    "port": 8080
  }
}
```

同时修改 `docker-compose.yml`：

```yaml
astrbot:
  ports:
    - "8080:6185"
```

---

## 配置文件修改注意事项

1. **备份**: 修改前先备份配置文件
2. **JSON 格式**: 确保 JSON 格式正确
3. **重启服务**: 修改后需要重启容器
4. **权限**: 某些配置需要管理员权限
5. **网络**: 容器间通信使用 `172.19.0.1`（Docker 网桥地址）

---

## 快速命令

```bash
# 进入 AstrBot 目录
cd /opt/astrbot

# 查看配置
cat cmd_config.json | jq

# 备份配置
cp data/cmd_config.json data/cmd_config.json.backup

# 重启服务
docker-compose restart astrbot

# 查看日志
docker-compose logs -f astrbot

# 重启所有服务
docker-compose restart

# 停止服务
docker-compose down

# 启动服务
docker-compose up -d
```

---

## 服务访问地址

| 服务 | 地址 | 说明 |
|------|------|------|
| Web 控制台 | http://123.57.192.94:6185 | AstrBot 管理界面 |
| Bay API | http://123.57.192.94:8114 | 沙箱运行时 API |
| NapCat | ws://123.57.192.94:6099 | QQ 协议 WebSocket |

---

## SSH 端口转发

本地访问远程服务：

```bash
# 转发 Web 控制台
ssh -fNL 6185:127.0.0.1:6185 root@123.57.192.94

# 转发 Bay API
ssh -fNL 8114:127.0.0.1:8114 root@123.57.192.94

# 访问
# Web 控制台: http://localhost:6185
# Bay API: http://localhost:8114
```

---

*最后更新: 2026-03-11*
