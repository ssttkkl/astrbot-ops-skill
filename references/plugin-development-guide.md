# AstrBot 插件开发指南

本文档包含插件开发的完整指南。

## 目录

1. [环境准备](#环境准备)
2. [核心概念](#核心概念)
3. [事件监听器](#事件监听器)
4. [消息发送](#消息发送)
5. [插件配置](#插件配置)
6. [常用 API](#常用-api)
7. [调试与安装](#调试与安装)

---

## 环境准备

### 获取插件模板

1. 打开 AstrBot 插件模板: [helloworld](https://github.com/Soulter/helloworld)
2. 点击右上角的 `Use this template`
3. 然后点击 `Create new repository`。
4. 在 `Repository name` 处填写您的插件名。插件名格式:
   - 推荐以 `astrbot_plugin_` 开头；
   - 不能包含空格；
   - 保持全部字母小写；
   - 尽量简短。
5. 点击右下角的 `Create repository`。

### 克隆项目到本地

克隆 AstrBot 项目本体和刚刚创建的插件仓库到本地。

```bash
git clone https://github.com/AstrBotDevs/AstrBot
mkdir -p AstrBot/data/plugins
cd AstrBot/data/plugins
git clone 插件仓库地址
```

然后，使用 `VSCode` 打开 `AstrBot` 项目。找到 `data/plugins/<你的插件名字>` 目录。

更新 `metadata.yaml` 文件，填写插件的元数据信息。

> [!WARNING]
> 请务必修改此文件，AstrBot 识别插件元数据依赖于 `metadata.yaml` 文件。

### 设置插件 Logo（可选）

可以在插件目录下添加 `logo.png` 文件作为插件的 Logo。请保持长宽比为 1:1，推荐尺寸为 256x256。

### 插件展示名（可选）

可以修改(或添加) `metadata.yaml` 文件中的 `display_name` 字段，作为插件在插件市场等场景中的展示名，以方便用户阅读。

### 声明支持平台（可选）

你可以在 `metadata.yaml` 中新增 `support_platforms` 字段（`list[str]`），声明插件支持的平台适配器。

```yaml
support_platforms:
  - telegram
  - discord
```

支持的平台列表：
- `aiocqhttp` - OneBot v11
- `qq_official` - QQ 官方 Bot
- `telegram` - Telegram
- `wecom` - 企业微信
- `lark` - 飞书
- `dingtalk` - 钉钉
- `discord` - Discord
- `slack` - Slack
- `kook` - Kook
- `vocechat` - VoceChat
- `weixin_official_account` - 微信公众号
- `satori` - Satori
- `misskey` - Misskey
- `line` - LINE

### 声明 AstrBot 版本范围（可选）

你可以在 `metadata.yaml` 中新增 `astrbot_version` 字段，声明插件要求的 AstrBot 版本范围。

```yaml
astrbot_version: ">=4.16,<5"
```

可选示例：
- `>=4.17.0`
- `>=4.16,<5`
- `~=4.17`

### 插件依赖管理

目前 AstrBot 对插件的依赖管理使用 `pip` 自带的 `requirements.txt` 文件。如果你的插件需要依赖第三方库，请务必在插件目录下创建 `requirements.txt` 文件并写入所使用的依赖库。

---

## 核心概念

### 插件结构

```
astrbot_plugin_example/
├── main.py              # 插件主文件（必需）
├── metadata.yaml        # 插件元数据（必需）
├── requirements.txt     # Python 依赖
├── logo.png            # 插件 Logo（可选）
├── _conf_schema.json   # 配置 Schema（可选）
└── tools/              # 函数工具目录（可选）
```

### metadata.yaml 示例

```yaml
name: "example"
display_name: "示例插件"
author: "Your Name"
desc: "插件描述"
version: "1.0.0"
repo: "https://github.com/user/repo"
support_platforms:
  - aiocqhttp
  - telegram
astrbot_version: ">=4.16,<5"
```

---

## 事件监听器

### 指令

```python
@filter.command("test")
async def test(self, event: AstrMessageEvent):
    yield event.plain_result("测试")

# 带参数
@filter.command("echo")
async def echo(self, event: AstrMessageEvent, message: str):
    yield event.plain_result(f"你说: {message}")

# 指令别名
@filter.command("help", alias={'帮助', 'h'})
async def help(self, event: AstrMessageEvent):
    yield event.plain_result("帮助信息")
```

### 指令组

```python
@filter.command_group("math")
def math(self):
    pass

@math.command("add")
async def add(self, event: AstrMessageEvent, a: int, b: int):
    yield event.plain_result(f"结果: {a + b}")
```

### 事件过滤

```python
# 监听所有消息
@filter.event_message_type(filter.EventMessageType.ALL)
async def on_all(self, event: AstrMessageEvent):
    pass

# 仅私聊
@filter.event_message_type(filter.EventMessageType.PRIVATE_MESSAGE)
async def on_private(self, event: AstrMessageEvent):
    pass

# 仅群聊
@filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)
async def on_group(self, event: AstrMessageEvent):
    pass

# 平台过滤
@filter.platform_adapter_type(filter.PlatformAdapterType.AIOCQHTTP)
async def on_qq(self, event: AstrMessageEvent):
    pass

# 管理员权限
@filter.permission_type(filter.PermissionType.ADMIN)
@filter.command("admin")
async def admin_cmd(self, event: AstrMessageEvent):
    pass
```

---

## 消息发送

### 被动消息

```python
# 纯文本
yield event.plain_result("Hello")

# 图片
yield event.image_result("path/to/image.jpg")
yield event.image_result("https://example.com/image.jpg")

# 消息链
import astrbot.api.message_components as Comp

chain = [
    Comp.At(qq=event.get_sender_id()),
    Comp.Plain("你好"),
    Comp.Image.fromURL("https://example.com/image.jpg")
]
yield event.chain_result(chain)
```

### 主动消息

```python
from astrbot.api.event import MessageChain

umo = event.unified_msg_origin
chain = MessageChain().message("Hello").file_image("path/to/image.jpg")
await self.context.send_message(umo, chain)
```

---

## 插件配置

### 创建配置 Schema

创建 `_conf_schema.json`：

```json
{
  "api_key": {
    "description": "API 密钥",
    "type": "string",
    "hint": "从官网获取",
    "obvious_hint": true
  },
  "timeout": {
    "description": "超时时间（秒）",
    "type": "int",
    "default": 30
  }
}
```

### 使用配置

```python
from astrbot.api import AstrBotConfig

class MyPlugin(Star):
    def __init__(self, context: Context, config: AstrBotConfig):
        super().__init__(context)
        self.api_key = config.get("api_key", "")
        self.timeout = config.get("timeout", 30)
```

---

## 常用 API

### Context 对象

```python
# LLM 提供商
provider = self.context.get_using_provider(umo=event.unified_msg_origin)
llm_resp = await provider.text_chat(prompt="Hi", context=[], system_prompt="")

# 对话管理
conv_mgr = self.context.conversation_manager
curr_cid = await conv_mgr.get_curr_conversation_id(uid)
conversation = await conv_mgr.get_conversation(uid, curr_cid)

# 人格管理
persona_mgr = self.context.persona_manager
persona = persona_mgr.get_persona("persona_id")

# 发送消息
await self.context.send_message(unified_msg_origin, message_chain)

# 配置
config = self.context.get_config(umo=event.unified_msg_origin)
```

### 文转图

```python
# 基本文转图
url = await self.text_to_image("文本内容")
yield event.image_result(url)

# HTML 模板渲染
TMPL = '''
<div style="font-size: 32px;">
<h1>{{ title }}</h1>
<ul>
{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}
</ul>
</div>
'''

url = await self.html_render(TMPL, {"title": "标题", "items": ["项1", "项2"]})
yield event.image_result(url)
```

### 会话控制

```python
from astrbot.core.utils.session_waiter import session_waiter, SessionController

@filter.command("game")
async def game(self, event: AstrMessageEvent):
    yield event.plain_result("游戏开始！")

    @session_waiter(timeout=60)
    async def game_waiter(controller: SessionController, event: AstrMessageEvent):
        user_input = event.message_str

        if user_input == "退出":
            await event.send(event.plain_result("游戏结束"))
            controller.stop()
            return

        # 处理逻辑
        await event.send(event.plain_result(f"你输入了: {user_input}"))
        controller.keep(timeout=60, reset_timeout=True)

    try:
        await game_waiter(event)
    except TimeoutError:
        yield event.plain_result("超时了")
```

---

## 调试与安装

### 方式一：手动安装（开发调试）

适用于本地开发调试场景：

1. **放置插件**
   ```bash
   # 将插件目录复制到 AstrBot 的插件目录
   cp -r astrbot_plugin_example /path/to/AstrBot/data/plugins/
   ```

2. **启动 AstrBot**
   ```bash
   python main.py
   ```

3. **重载插件**
   - 打开 Dashboard WebUI (`http://localhost:6185`)
   - 进入「扩展」页面
   - 找到你的插件，点击右上角 `...` -> `重载插件`
   - 查看日志输出确认加载成功

4. **热更新**
   - 修改代码后，再次点击「重载插件」即可生效
   - 无需重启 AstrBot

### 方式二：通过 API 安装（自动化部署）

适用于自动化部署或远程管理场景：

#### 1. 登录获取 Token

```bash
curl -X POST http://localhost:6185/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "astrbot",
    "password": "77b90590a8945a7d36c963981a307dc9"
  }'
```

响应：
```json
{
  "status": "ok",
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

#### 2. 安装插件

**从 Git 仓库安装：**

```bash
curl -X POST http://localhost:6185/api/plugin/install \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "url": "https://github.com/user/astrbot_plugin_example",
    "proxy": "https://ghproxy.com"
  }'
```

**上传本地文件安装：**

```bash
curl -X POST http://localhost:6185/api/plugin/install-upload \
  -H "Authorization: Bearer <token>" \
  -F "file=@astrbot_plugin_example.zip"
```

#### 3. 重载插件

如果插件已存在，更新代码后需要重载：

```bash
curl -X POST http://localhost:6185/api/plugin/reload \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "name": "astrbot_plugin_example"
  }'
```

> 不传 `name` 字段则重载所有插件

#### 4. 查看插件列表

```bash
curl -X GET "http://localhost:6185/api/plugin/get" \
  -H "Authorization: Bearer <token>"
```

#### 5. 卸载插件

```bash
curl -X POST http://localhost:6185/api/plugin/uninstall \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "name": "astrbot_plugin_example",
    "delete_config": false,
    "delete_data": false
  }'
```

---

## 目录结构参考

### references/ 目录说明

```
references/
├── plugin-development-guide.md         # 本文件（快速参考手册）
├── plugin-new.md                       # 新版插件开发指南（官方原始版）
├── dashboard_interfaces_and_config_formats.md  # Dashboard API 详细接口定义
└── dashboard_communication_summary.md  # Dashboard 通信机制和接口汇总
```
