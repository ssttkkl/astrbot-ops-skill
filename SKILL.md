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

## 服务器管理

### AstrBot 目录结构

```
${ASTRBOT_DIR}/data/
├── plugins/          # 插件目录
├── skills/           # 技能目录
├── mcp_server.json   # MCP 服务器配置
└── config.yaml       # 主配置文件
```

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


## 参考文档索引（人类查阅用）

本技能所有文档位于 `references/` 目录：

### 核心文档
- `debugging.md` - 插件调试指南

### Dashboard API
- `dashboard_api/dashboard_interfaces_and_config_formats.md` - API 接口和配置格式
- `dashboard_api/dashboard_communication_summary.md` - 通信机制汇总

### 官方文档地址：
- GitHub: https://github.com/AstrBotDevs/AstrBot/tree/master/docs
- DeepWiki: https://deepwiki.com/AstrBotDevs/AstrBot
