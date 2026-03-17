---
name: astrbot-plugin-dev
description: |
  AstrBot 插件开发、配置管理和服务器运维的完整指南。
  
  使用场景：
  1. 开发 AstrBot 插件时需要了解插件架构、事件系统、API 用法
  2. 修改 AstrBot 配置时需要使用 Dashboard API 或管理插件/技能/MCP
  3. 远程管理 AstrBot 服务器时需要查看插件、技能、MCP 配置状态
  4. 调试插件、配置 MCP 服务器、管理技能启用状态
  
  适用于所有 AstrBot 相关的开发和运维任务。
---

# AstrBot 插件开发技能

## 环境变量配置

**AI Agent 执行前必读：**

1. **获取 ASTRBOT_DIR 路径**：
   ```
   步骤 1：调用 Read 工具读取 TOOLS.md
   - path: ~/.openclaw/workspace/TOOLS.md
   
   步骤 2：在内容中查找 "ASTRBOT_DIR" 配置
   - 格式示例：ASTRBOT_DIR=/opt/astrbot
   - 提取路径值
   
   步骤 3：如果未找到，询问用户
   - 常见路径：/opt/astrbot（Docker）、~/AstrBot（本地）
   ```

2. **路径替换**：
   - 文档中所有 `${ASTRBOT_DIR}` 需要替换为从 TOOLS.md 读取的实际路径
   - 在执行命令前完成替换

**示例**：
```bash
# 从 TOOLS.md 读取到：ASTRBOT_DIR="/opt/astrbot"

# 替换后的命令
cat /opt/astrbot/data/cmd_config.json
```

---

## 使用场景

1. **插件开发** - 开发 AstrBot 插件时需要：
   - 了解插件架构和事件系统
   - 查询 AstrBot API 用法（消息发送、事件监听、会话控制等）
   - 开发新插件或调试现有插件
   - 遵循开发规范和最佳实践

2. **修改 AstrBot 配置** - 需要程序化修改配置时需要：
   - 通过 Dashboard API 获取/修改配置
   - 管理插件（安装、卸载、重载）
   - 配置 MCP 服务器和技能管理
   - 远程管理 AstrBot 实例

3. **服务器管理** - 查看和管理服务器上的插件、技能、MCP 配置

**执行原则（AI agent 必读）：**

1. **开发插件前，必须先读取相关文档：**
   ```
   步骤 1：使用 Read 工具读取文档
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/plugin-development-guide.md</parameter>
   </invoke>
   </function_calls>
   
   步骤 2：将文档内容传递给编码子代理
   <function_calls>
   <invoke name="sessions_spawn">
   <parameter name="runtime">acp</parameter>
   <parameter name="agentId">claude</parameter>
   <parameter name="mode">run</parameter>
   <parameter name="cwd">${ASTRBOT_DIR}/data/plugins/astrbot_plugin_<name></parameter>
   <parameter name="task">开发 AstrBot 插件：<需求描述>

参考文档：
<文档完整内容></parameter>
   </invoke>
   </function_calls>
   
   注意：
   - mode="run" 用于一次性任务
   - mode="session" 用于需要多轮交互的复杂开发
   - cwd 设置为插件目录，确保代码生成在正确位置
   ```

2. **查询 API 用法时，读取对应的官方文档：**
   ```
   示例：需要发送消息功能
   
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/dev/star/guides/send-message.md</parameter>
   </invoke>
   </function_calls>
   
   然后将文档内容传递给子代理（参考步骤 1）
   ```
   
   常用文档路径：
   - 发送消息：`references/dev/star/guides/send-message.md`
   - 监听事件：`references/dev/star/guides/listen-message-event.md`
   - 会话控制：`references/dev/star/guides/session-control.md`
   - 数据存储：`references/dev/star/guides/storage.md`

3. **修改配置前，读取配置格式文档：**
   ```
   步骤 1：读取配置格式文档
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/dashboard_api/dashboard_interfaces_and_config_formats.md</parameter>
   </invoke>
   </function_calls>
   
   步骤 2：读取当前配置（从 TOOLS.md 获取 ASTRBOT_DIR）
   <function_calls>
   <invoke name="exec">
   <parameter name="command">cat ${ASTRBOT_DIR}/data/cmd_config.json</parameter>
   </invoke>
   </function_calls>
   
   步骤 3：修改配置（使用 Edit 工具或 Python 脚本）
   
   步骤 4：验证 JSON 格式
   <function_calls>
   <invoke name="exec">
   <parameter name="command">python3 -m json.tool < ${ASTRBOT_DIR}/data/cmd_config.json</parameter>
   </invoke>
   </function_calls>
   ```

## 常用操作

### 执行测试

```bash
# 进入测试目录
cd <project_path>/agent-test

# 执行测试（需要 Dashboard 凭证）
python3 test-script.py <test-cases.json> <username> <password_md5>

# 示例
python3 test-script.py test-cases-python.json admin d57e17c796fe1a89cd2ccac987f1531b
```

### 查看日志

```bash
# Docker 容器日志
docker logs <container_name> --tail 20

# 实时查看
docker logs -f <container_name>

# 示例
docker logs astrbot --tail 50
```

### 更新部署

```bash
# SSH 到服务器
ssh <user>@<server>

# 进入插件目录
cd ${ASTRBOT_DIR}/data/plugins/<plugin_name>

# 拉取最新代码
git pull origin <branch>

# 验证 commit
git log -1 --oneline

# 重启服务
docker restart <container_name>
# 或
docker-compose restart <service_name>
```

### 验证服务状态

```bash
# 检查容器状态
docker ps | grep <container_name>

# 检查日志
docker logs <container_name> --tail 20
```

---

## 快速开始

### 1. 获取插件模板

```bash
# 使用 GitHub 模板创建新仓库
# https://github.com/Soulter/helloworld
# 点击 "Use this template" -> "Create new repository"
# 插件名格式：astrbot_plugin_<name>（小写，无空格）
```

### 2. 克隆到本地

```bash
git clone https://github.com/AstrBotDevs/AstrBot
mkdir -p AstrBot/data/plugins
cd AstrBot/data/plugins
git clone <你的插件仓库地址>
```

### 3. 最小插件示例

```python
from astrbot.api.event import filter, AstrMessageEvent
from astrbot.api.star import Context, Star
from astrbot.api import logger

class MyPlugin(Star):
    def __init__(self, context: Context):
        super().__init__(context)

    @filter.command("hello")
    async def hello(self, event: AstrMessageEvent):
        '''Hello World 指令'''
        user_name = event.get_sender_name()
        yield event.plain_result(f"Hello, {user_name}!")
```


### 4. 调试和测试

**插件调试：**
```
调用 read 工具读取：
~/.openclaw/workspace/skills/astrbot-plugin-dev/references/debugging.md
```

**TDD 测试：**
```
调用 read 工具读取：
~/.openclaw/workspace/skills/astrbot-plugin-dev/references/tdd-testing.md
```

## 开发规范

- **功能测试**：所有功能必须经过测试
- **数据存储**：持久化数据存储在 `data/` 目录，不要存在插件目录
- **错误处理**：良好的异常处理，不要让插件崩溃
- **代码格式**：使用 [ruff](https://docs.astral.sh/ruff/) 格式化代码
- **异步网络**：使用 `aiohttp`/`httpx`，不要用 `requests`
- **日志记录**：使用 `from astrbot.api import logger`
- **插件命名**：`astrbot_plugin_<name>`（小写，无空格）

## 服务器管理

### 配置文件位置

**已安装插件目录：**
- 路径：`${ASTRBOT_DIR}/data/plugins/`
- 说明：每个插件一个子目录，目录名为 `astrbot_plugin_<name>`

**已安装技能目录：**
- 路径：`${ASTRBOT_DIR}/data/skills/`
- 说明：每个技能一个子目录

**MCP 服务器配置文件：**
- 路径：`${ASTRBOT_DIR}/data/mcp_server.json`
- 说明：包含所有 MCP 服务器的配置

**主配置文件：**
- 路径：`${ASTRBOT_DIR}/data/cmd_config.json`
- 说明：包含 Dashboard 登录凭证、LLM 配置等

### MCP 服务器配置

MCP 配置文件位于：`${ASTRBOT_DIR}/data/mcp_server.json`

**配置格式：**
```json
{
    "mcpServers": {
        "server_name": {
            "active": true,
            "transport": "streamable_http",
            "url": "https://example.com",
            "headers": {
                "Authorization": "Bearer YOUR_TOKEN"
            },
            "timeout": 5,
            "sse_read_timeout": 300
        }
    }
}
```

**字段说明：**
- `active`: 是否启用该 MCP 服务器
- `transport`: 传输方式（streamable_http / stdio）
- `url`: MCP 服务器地址
- `headers`: HTTP 请求头（可选）
- `timeout`: 连接超时时间（秒）
- `sse_read_timeout`: SSE 读取超时时间（秒）

### 目录结构

```
${ASTRBOT_DIR}/data/
├── plugins/          # 插件目录
├── skills/           # 技能目录
├── mcp_server.json   # MCP 服务器配置
└── config.yaml       # 主配置文件
```

## AI Agent 执行指南

**开发插件时的标准流程：**

1. **读取核心开发指南**
   ```
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/plugin-development-guide.md</parameter>
   </invoke>
   </function_calls>
   ```

2. **根据需求读取对应的 API 文档**
   ```
   示例：需要发送消息功能
   
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/dev/star/guides/send-message.md</parameter>
   </invoke>
   </function_calls>
   ```
   
   其他文档路径：
   - 监听事件：`references/dev/star/guides/listen-message-event.md`
   - 会话控制：`references/dev/star/guides/session-control.md`
   - 数据存储：`references/dev/star/guides/storage.md`

3. **将文档内容传递给编码子代理**
   ```
   <function_calls>
   <invoke name="sessions_spawn">
   <parameter name="runtime">acp</parameter>
   <parameter name="agentId">claude</parameter>
   <parameter name="mode">run</parameter>
   <parameter name="cwd">${ASTRBOT_DIR}/data/plugins/astrbot_plugin_<name></parameter>
   <parameter name="task">开发 AstrBot 插件...

参考文档：
[从步骤1和2读取的文档内容]</parameter>
   </invoke>
   </function_calls>
   ```

4. **调试时读取调试指南**
   ```
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/debugging.md</parameter>
   </invoke>
   </function_calls>
   ```

---

## TDD 团队协作模式

当需要进行测试驱动开发时，可以使用多 agent 协作模式。

**详细文档**：
```
read ~/.openclaw/workspace/skills/astrbot-plugin-dev/references/tdd-team-collaboration.md
```

该文档包含：
- 6 个团队角色（Test Writer、Test Executor、Developer、Code Reviewer、Refactorer、Coverage Checker、Deployer）
- 交接协议模板
- 标准 TDD 流程（Red-Green-Refactor）
- 测试修复流程

---

## 修改配置时的标准流程

1. **读取配置格式文档**
   ```
   <function_calls>
   <invoke name="Read">
   <parameter name="path">~/.openclaw/workspace/skills/astrbot-plugin-dev/references/dashboard_api/dashboard_interfaces_and_config_formats.md</parameter>
   </invoke>
   </function_calls>
   ```

2. **读取当前配置**
   ```
   <function_calls>
   <invoke name="exec">
   <parameter name="command">cat ${ASTRBOT_DIR}/data/mcp_server.json</parameter>
   </invoke>
   </function_calls>
   
   或读取主配置：
   <function_calls>
   <invoke name="exec">
   <parameter name="command">cat ${ASTRBOT_DIR}/data/cmd_config.json</parameter>
   </invoke>
   </function_calls>
   ```

3. **修改后验证格式**
   ```
   <function_calls>
   <invoke name="exec">
   <parameter name="command">python3 -m json.tool < ${ASTRBOT_DIR}/data/mcp_server.json</parameter>
   </invoke>
   </function_calls>
   ```

**官方文档地址（人类参考）：**
- GitHub: https://github.com/AstrBotDevs/AstrBot/tree/master/docs
- DeepWiki: https://deepwiki.com/AstrBotDevs/AstrBot


## 参考文档索引（人类查阅用）

本技能所有文档位于 `references/` 目录：

### 核心文档
- `plugin-development-guide.md` - 插件开发完整指南（推荐首读）
- `debugging.md` - 插件调试指南
- `tdd-testing.md` - TDD 测试指南

### Dashboard API
- `dashboard_api/dashboard_interfaces_and_config_formats.md` - API 接口和配置格式
- `dashboard_api/dashboard_communication_summary.md` - 通信机制汇总

### 官方开发指南（dev/ 目录）
- `dev/star/plugin-new.md` - 新版插件开发
- `dev/star/guides/simple.md` - 最小示例
- `dev/star/guides/send-message.md` - 发送消息
- `dev/star/guides/listen-message-event.md` - 监听事件
- `dev/star/guides/session-control.md` - 会话控制
- `dev/star/guides/storage.md` - 数据存储
- `dev/star/guides/ai.md` - AI 功能
- `dev/star/guides/html-to-pic.md` - 文转图
- `dev/star/guides/plugin-config.md` - 插件配置
- `dev/star/guides/env.md` - 环境变量
