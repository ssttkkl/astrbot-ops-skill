---
name: astrbot-ops
description: |
  AstrBot 服务器运维和配置管理指南。
  
  使用场景：
  1. 通过 Dashboard API 程序化修改 AstrBot 配置
  2. 远程管理服务器上的插件、技能、MCP 配置状态
  3. 配置 MCP 服务器、查看日志、更新部署、验证服务状态
  
  插件开发请使用 skill-astrbot-dev 技能。
---

# AstrBot 运维技能

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

1. **修改 AstrBot 配置** - 需要程序化修改配置时需要：
   - 通过 Dashboard API 获取/修改配置
   - 管理插件（安装、卸载、重载）
   - 配置 MCP 服务器和技能管理
   - 远程管理 AstrBot 实例

2. **服务器管理** - 查看和管理服务器上的插件、技能、MCP 配置

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
- `debugging.md` - 插件调试指南

### Dashboard API
- `dashboard_api/dashboard_interfaces_and_config_formats.md` - API 接口和配置格式
- `dashboard_api/dashboard_communication_summary.md` - 通信机制汇总
