# AstrBot 插件调试指南


完整的插件调试流程，通过 Dashboard API 进行远程调试：

#### 步骤 1：安装插件

根据 AstrBot 部署方式不同，插件安装方式有所区别：

**场景 A：本地部署（直接运行）**

```bash
# AstrBot 本地数据目录
ASTRBOT_DATA_DIR="~/AstrBot/data"

# 复制插件到插件目录
cp -r ~/my-plugin ${ASTRBOT_DATA_DIR}/plugins/astrbot_plugin_myplugin

# 或者使用符号链接（开发时方便）
ln -s ~/workspace/my-plugin ${ASTRBOT_DATA_DIR}/plugins/astrbot_plugin_myplugin
```

**自动化动作**：将插件目录复制或链接到 `AstrBot/data/plugins/<插件名>/`

**场景 B：远程服务器部署（SSH）**

```bash
# 本地插件目录
LOCAL_PLUGIN_DIR="~/workspace/my-plugin"

# 服务器信息
SERVER="<用户名>@<服务器IP>"
REMOTE_PLUGIN_DIR="${ASTRBOT_DIR}/data/plugins/astrbot_plugin_myplugin"

# 同步插件代码到服务器
rsync -avz ${LOCAL_PLUGIN_DIR}/ ${SERVER}:${REMOTE_PLUGIN_DIR}/ \
  --exclude='.git' --exclude='__pycache__' --exclude='*.pyc'
```

**自动化动作**：使用 rsync/scp 同步本地插件代码到远程服务器的 `${ASTRBOT_DIR}/data/plugins/<插件名>/`

**场景 C：Docker 部署（本地或远程）**

```bash
# 方法 1：直接复制到容器（适合临时测试）
docker cp ~/my-plugin/. astrbot:${ASTRBOT_DIR}/data/plugins/astrbot_plugin_myplugin/

# 方法 2：通过卷挂载（推荐开发时使用）
# docker-compose.yml 配置示例：
# volumes:
#   - ./my-plugin:${ASTRBOT_DIR}/data/plugins/astrbot_plugin_myplugin:ro

# 方法 3：构建自定义镜像（适合生产部署）
# Dockerfile:
# COPY my-plugin ${ASTRBOT_DIR}/data/plugins/astrbot_plugin_myplugin
```

**自动化动作**：
- 临时测试：`docker cp` 复制文件到容器
- 开发环境：配置 Docker Compose 卷挂载
- 生产部署：构建包含插件的自定义镜像

**三种场景对比**：

| 部署方式 | 适用场景 | 同步命令 | 重启需求 |
|---------|---------|---------|---------|
| 本地部署 | 本地开发测试 | `cp` / `ln -s` | 重载插件即可 |
| 远程服务器 | 云服务器部署 | `rsync` / `scp` | 重载插件即可 |
| Docker | 容器化部署 | `docker cp` / 卷挂载 | 重载插件或重启容器 |

#### 步骤 2：获取 Dashboard 登录 Token

**前置条件：设置端口转发（远程服务器/Docker 部署）**

如果 AstrBot 部署在远程服务器或 Docker 容器中，需要先建立端口转发：

```bash
# SSH 端口转发（远程服务器）
ssh -fNL 6185:127.0.0.1:6185 <用户名>@<服务器IP>

# Docker 端口映射（已在 docker-compose.yml 中配置）
# ports:
#   - "6185:6185"
```

**步骤 2.1：从配置文件获取登录凭证**

```bash
# 查看配置文件
cat ${ASTRBOT_DIR}/data/cmd_config.json | grep -A5 dashboard

# 输出示例：
# "dashboard": {
#     "enable": true,
#     "username": "astrbot",
#     "password": "<MD5_HASHED_PASSWORD>",  # MD5 哈希值
#     ...
# }
```

**注意：** 配置文件中的 `password` 已经是 MD5 哈希值，直接使用即可。

**步骤 2.2：调用登录接口获取 JWT Token**

使用上一步获取的用户名和密码调用登录接口：

```bash
# 登录获取 token
TOKEN_RESPONSE=$(curl -s http://127.0.0.1:6185/api/auth/login   -H "Content-Type: application/json"   -d '{"username": "astrbot", "password": "<MD5_HASHED_PASSWORD>"}')

# 提取 token
TOKEN=$(echo $TOKEN_RESPONSE | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['token'])")
```

**自动化动作总结**：
1. 确保端口转发已设置（远程/Docker 部署）
2. 从配置文件读取用户名和密码（MD5 哈希）
3. 调用 `/api/auth/login` 接口获取 JWT Token
4. 从响应中提取 `data.token` 字段

#### 步骤 3：重载插件

使用 Dashboard API 重载指定插件：

```bash
curl -s http://127.0.0.1:6185/api/plugin/reload \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${TOKEN}" \
  -d '{"name": "astrbot_plugin_nsjail"}'
```

**自动化动作**：
1. 调用 `/api/plugin/reload` 接口
2. 请求体包含 `"name": "<插件名>"`
3. 不传 name 则重载所有插件

#### 步骤 4：发送测试对话

通过 API 发送消息触发插件功能：

```bash
curl -s http://127.0.0.1:6185/api/chat/send \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${TOKEN}" \
  -d '{
    "message": "使用 execute_shell 工具执行命令：echo 1+1 | bc",
    "session_id": "test-session-001"
  }'
```

**自动化动作**：
1. 调用 `/api/chat/send` 接口
2. 指定 `message` 内容（包含触发插件的关键词）
3. 指定 `session_id` 用于会话追踪
4. 接口返回 SSE 流式响应

#### 步骤 5：检查日志

查看 AstrBot 容器日志，确认插件行为：

```bash
# 查看实时日志
ssh <用户名>@<服务器IP> "docker logs <容器名> --tail 100 -f 2>&1"

# 查看特定时间段的日志
ssh <用户名>@<服务器IP> "docker logs <容器名> 2>&1 | grep -E '(nsjail|execute_shell|插件名)' -i"

# 查看工具调用日志
ssh <用户名>@<服务器IP> "docker logs <容器名> 2>&1 | grep '添加函数调用工具'"
```

**检查要点**：
- 插件是否正确加载（`正在载入插件`）
- 工具是否正确注册（`added LLM tool`）
- 工具是否被调用（`execute_shell` 或插件特定日志）
- 是否有错误信息（`[ERRO]`、`[WARN]`）

#### 调试流程示例

完整的测试脚本请参考：`scripts/test-script.py`

#### 常见问题排查

| 问题 | 排查方法 |
|------|----------|
| 插件未加载 | 检查日志中 `正在载入插件` 是否成功 |
| 工具未注册 | 检查日志中 `added LLM tool` 是否出现 |
| LLM 不调用工具 | 检查 `tool_schema_mode` 配置，尝试改为 `function_call` |
| 工具调用失败 | 检查错误日志，确认 nsjail 命令参数正确 |
| 配置不生效 | 检查 `provider_settings` 中的工具相关配置 |

