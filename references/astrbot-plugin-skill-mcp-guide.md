# AstrBot 插件/技能/MCP 服务器管理指南

## 一、插件管理

### 1.1 插件配置文件

**位置**: `/AstrBot/data/plugins.json`（容器内）

**结构**:
```json
{
  "timestamp": "2026-03-11T02:57:52.342301",
  "data": {
    "plugin-name": {
      "display_name": "插件显示名称",
      "desc": "插件描述",
      "author": "作者",
      "repo": "https://github.com/author/repo",
      "tags": ["tag1", "tag2"],
      "stars": 0,
      "version": "v1.0.0",
      "updated_at": "2026-03-10T00:00:00Z",
      "logo": "https://example.com/logo.png"
    }
  }
}
```

### 1.2 安装插件

**方法 1: 通过 Web 控制台**
1. 访问 http://123.57.192.94:6185
2. 登录后进入「插件管理」
3. 搜索插件并点击「安装」

**方法 2: 手动安装**
```bash
# 进入插件目录
cd /opt/astrbot/data/plugins

# 克隆插件仓库
git clone https://github.com/author/plugin-name

# 重启 AstrBot
docker-compose restart astrbot
```

### 1.3 启用/禁用插件

通过 Web 控制台的「插件管理」页面，点击插件卡片的开关按钮。

---

## 二、技能管理

### 2.1 技能配置文件

**位置**: `/AstrBot/data/skills.json`（容器内）

**结构**:
```json
{
  "skills": {
    "skill-name": {
      "active": true
    },
    "another-skill": {
      "active": false
    }
  }
}
```

### 2.2 技能目录

**位置**: `/AstrBot/data/skills/`（容器内）

**目录结构**:
```
/AstrBot/data/skills/
├── skill-name/
│   ├── __init__.py
│   ├── main.py
│   └── requirements.txt
└── another-skill/
    └── ...
```

### 2.3 添加技能

**方法 1: 通过 Web 控制台**
1. 访问「技能管理」页面
2. 点击「添加技能」
3. 输入技能代码或上传技能包

**方法 2: 手动添加**
```bash
# 进入技能目录
cd /opt/astrbot/data/skills

# 创建技能目录
mkdir my-skill
cd my-skill

# 创建技能文件
cat > main.py << 'EOF'
from astrbot.api import skill

@skill.register("my_skill")
async def my_skill_handler(ctx):
    await ctx.reply("Hello from my skill!")
EOF

# 编辑 skills.json 启用技能
cd /opt/astrbot/data
# 手动编辑 skills.json，添加:
# "my-skill": { "active": true }

# 重启 AstrBot
docker-compose restart astrbot
```

### 2.4 启用/禁用技能

**方法 1: 修改配置文件**
```bash
# 编辑 skills.json
ssh root@123.57.192.94
cd /opt/astrbot/data
vi skills.json

# 修改 active 字段
{
  "skills": {
    "skill-name": {
      "active": true  # true=启用, false=禁用
    }
  }
}

# 重启服务
docker-compose restart astrbot
```

**方法 2: 通过 Web 控制台**
在「技能管理」页面切换技能开关。

---

## 三、MCP 服务器管理

### 3.1 MCP 服务器配置文件

**位置**: `/AstrBot/data/mcp_server.json`（容器内）

**结构**:
```json
{
  "mcpServers": {
    "server-name": {
      "active": true,
      "transport": "streamable_http",
      "url": "https://mcp.example.com",
      "headers": {
        "Authorization": "Bearer your-token"
      },
      "timeout": 5,
      "sse_read_timeout": 300
    }
  }
}
```

### 3.2 字段说明

- `active`: 是否启用（true/false）
- `transport`: 传输协议
  - `streamable_http`: HTTP 流式传输
  - `stdio`: 标准输入输出
- `url`: MCP 服务器地址
- `headers`: 自定义请求头（如认证 token）
- `timeout`: 请求超时时间（秒）
- `sse_read_timeout`: SSE 读取超时时间（秒）

### 3.3 添加 MCP 服务器

**方法 1: 手动编辑配置**
```bash
# 编辑 mcp_server.json
ssh root@123.57.192.94
cd /opt/astrbot/data
vi mcp_server.json

# 添加新服务器
{
  "mcpServers": {
    "my-mcp-server": {
      "active": true,
      "transport": "streamable_http",
      "url": "https://my-mcp.example.com",
      "headers": {
        "Authorization": "Bearer my-token"
      },
      "timeout": 10,
      "sse_read_timeout": 300
    }
  }
}

# 重启服务
docker-compose restart astrbot
```

**方法 2: 通过 Web 控制台**
在「MCP 服务器管理」页面添加新服务器。

### 3.4 启用/禁用 MCP 服务器

修改配置文件中的 `active` 字段：
```json
{
  "mcpServers": {
    "server-name": {
      "active": false  # 禁用
    }
  }
}
```

重启服务生效：
```bash
docker-compose restart astrbot
```

---

## 四、配置文件修改流程

### 4.1 标准流程

1. **备份配置**
```bash
cd /opt/astrbot/data
cp skills.json skills.json.backup
cp mcp_server.json mcp_server.json.backup
```

2. **编辑配置**
```bash
vi skills.json
# 或
vi mcp_server.json
```

3. **验证 JSON 格式**
```bash
cat skills.json | jq
```

4. **重启服务**
```bash
cd /opt/astrbot
docker-compose restart astrbot
```

5. **查看日志**
```bash
docker-compose logs -f astrbot
```

### 4.2 常见问题

**问题 1: JSON 格式错误**
```bash
# 使用 jq 验证
cat skills.json | jq
# 如果有错误会提示具体位置
```

**问题 2: 配置未生效**
- 确认已重启服务
- 检查日志是否有错误
- 确认文件权限正确

**问题 3: 技能/插件加载失败**
- 检查依赖是否安装
- 查看日志中的错误信息
- 确认代码语法正确

---

## 五、快速命令

```bash
# 查看已安装插件
ls /opt/astrbot/data/plugins/

# 查看已安装技能
ls /opt/astrbot/data/skills/

# 查看技能配置
cat /opt/astrbot/data/skills.json | jq

# 查看 MCP 服务器配置
cat /opt/astrbot/data/mcp_server.json | jq

# 备份所有配置
cd /opt/astrbot/data
tar -czf config-backup-$(date +%Y%m%d).tar.gz \
  skills.json mcp_server.json cmd_config.json

# 重启服务
cd /opt/astrbot
docker-compose restart astrbot

# 查看实时日志
docker-compose logs -f astrbot
```

---

## 六、开发自定义技能

### 6.1 技能模板

```python
# /AstrBot/data/skills/my-skill/main.py
from astrbot.api import skill, Context

@skill.register(
    name="my_skill",
    description="我的自定义技能",
    usage="/myskill <参数>"
)
async def my_skill_handler(ctx: Context):
    """技能处理函数"""
    # 获取用户输入
    user_input = ctx.message.text
    
    # 处理逻辑
    result = process_input(user_input)
    
    # 回复用户
    await ctx.reply(result)

def process_input(text: str) -> str:
    """处理输入的业务逻辑"""
    return f"处理结果: {text}"
```

### 6.2 技能配置

```python
# /AstrBot/data/skills/my-skill/__init__.py
from .main import my_skill_handler

__all__ = ["my_skill_handler"]
```

### 6.3 依赖管理

```txt
# /AstrBot/data/skills/my-skill/requirements.txt
requests>=2.28.0
aiohttp>=3.8.0
```

安装依赖：
```bash
docker exec -it astrbot pip install -r /AstrBot/data/skills/my-skill/requirements.txt
```

---

*最后更新: 2026-03-11*
