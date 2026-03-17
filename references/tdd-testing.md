# AstrBot 插件 TDD 测试指南


### 方法 1：TDD 子代理编排（推荐）

使用多个子代理协作完成测试驱动开发，适合复杂插件开发和持续迭代。

**⚠️ 重要：编排子代理前必须先使用 agent-team-orchestration 技能**

在启动下面的子代理编排流程前，必须先使用 `agent-team-orchestration` 技能定义团队角色、任务生命周期和交接协议。这确保了子代理之间的协作流程清晰、任务分工明确。

**完整编排流程：**

```
测试用例准备 → 测试执行 → 开发修复 → 代码审查 → 服务器部署 → 再次测试 → 循环直到通过
```

#### 步骤 1：生成测试用例（Test Case Generator 子代理）

**重要：测试用例必须根据插件代码和文档动态生成，不同插件的测试场景不同。**

```python
sessions_spawn(
    runtime="subagent",
    agentId="general-task-execution",
    mode="run",
    task="""你是 Test Case Generator。生成测试用例。

**AstrBot 开发文档：**
请阅读 astrbot-plugin-dev 技能的 references/ 目录中的所有文档。

**服务器：** <用户名>@<服务器IP>
**插件路径：** ${ASTRBOT_DIR}/data/plugins/astrbot_plugin_<name>/

**任务：**
1. 读取插件代码和文档
2. 确定测试范围：
   - 如果是代码迭代：分析改动的影响范围
   - 如果是新插件：测试整个插件功能
3. 生成测试用例，覆盖多个场景（功能、边界、异常、安全等）
4. 输出 `test-cases.json` 文件

**测试用例格式：**
```json
[
  {
    "name": "测试名称",
    "command": "shell命令",
    "expected": "期望输出片段",
    "category": "测试类别"
  }
]
```

**重要：测试用例必须针对插件的实际功能设计。**

#### 步骤 2：准备测试脚本

使用现有的测试脚本：`scripts/test-script.py`

该脚本通过端口转发 + API 请求执行测试用例。详细实现请查看脚本文件。

#### 步骤 3：子代理编排

**重要：编排子代理前必须先使用 agent-team-orchestration 技能了解编排规范和最佳实践。**

**3.1 Test Executor 子代理**

```python
sessions_spawn(
    runtime="subagent",
    agentId="general-task-execution",
    mode="run",
    task="""你是 Test Executor。执行测试并报告结果。

**测试文件：**
- 测试用例：`test-cases.json`
- 测试脚本：`test-script.py`

**任务：**
1. 执行：`python3 test-script.py test-cases.json`
2. 等待测试完成
3. 读取 `test-results.json`
4. 分析每个测试，判断通过/失败
5. 报告测试摘要和失败的测试

**重要：不允许修改测试脚本和用例。**
""",
    timeoutSeconds=600
)
```

**3.2 Developer 子代理（必须使用 coding-agent 技能）**

**重要：开发任务必须通过 coding-agent 技能调用 Claude Code。**

```python
# 主 agent 调用 coding-agent 技能
# coding-agent 技能会自动使用 sessions_spawn(runtime="acp", agentId="claude")

task = f"""你是 Developer。根据测试反馈修复代码。

**AstrBot 开发文档：**
请阅读 astrbot-plugin-dev 技能的 references/ 目录中的所有文档以了解开发规范。

**服务器：** <用户名>@<服务器IP>
**插件路径：** ${ASTRBOT_DIR}/data/plugins/astrbot_plugin_nsjail/main.py

**测试反馈：**
{test_failures}

**任务：**
1. 读取 AstrBot 开发文档目录中的相关文档
2. 分析测试失败原因
3. 修改 main.py 修复问题
4. 不要在服务器构建容器
5. 报告修改内容

**重要：必须先阅读开发文档，再进行修改。**
"""

# 使用 coding-agent 技能执行开发任务
# 技能会处理文档传递和 ACP 调用
```

**3.3 Code Reviewer 子代理（必须使用 coding-agent 技能）**

**重要：代码审查任务必须通过 coding-agent 技能调用 Claude Code。**

```python
# 主 agent 调用 coding-agent 技能
# coding-agent 技能会自动使用 sessions_spawn(runtime="acp", agentId="claude")

task = f"""你是 Code Reviewer。审查代码修改。

**AstrBot 开发文档：**
请阅读 astrbot-plugin-dev 技能的 references/ 目录中的所有文档以了解开发规范。

**服务器：** <用户名>@<服务器IP>
**插件路径：** ${ASTRBOT_DIR}/data/plugins/astrbot_plugin_nsjail/main.py

**任务：**
1. 读取 AstrBot 开发文档目录中的相关文档
2. 读取修改后的代码
3. 检查：代码质量、安全性、功能正确性
4. 给出审查报告：通过/需要修改
5. 如需修改，列出具体问题

**重要：必须先阅读开发文档，再进行审查。**
"""

# 使用 coding-agent 技能执行审查任务
# 技能会处理文档传递和 ACP 调用
```

**3.4 部署并重新测试**

```bash
# 部署（不在服务器构建）
ssh <用户名>@<服务器IP> "docker restart <容器名>"

# 再次调用 Test Executor 子代理
# 重复流程直到所有测试通过
```

#### 步骤 4：完整编排示例

```python
# 主 agent 编排逻辑
iteration = 1
max_iterations = 5

while iteration <= max_iterations:
    print(f"=== 迭代 {iteration} ===")
    
    # 1. 执行测试
    test_result = spawn_test_executor()
    
    if test_result['all_passed']:
        print("✅ 所有测试通过！")
        break
    
    # 2. 开发修复
    spawn_developer(test_result['failures'])
    
    # 3. 代码审查
    review_result = spawn_code_reviewer()
    
    if review_result['needs_changes']:
        print("⚠️ 代码审查未通过，继续修复...")
        continue
    
    # 4. 部署
    deploy_to_server()
    
    iteration += 1

if iteration > max_iterations:
    print("❌ 达到最大迭代次数，测试未全部通过")
```

**优势：**
- TDD 驱动开发，测试先行
- 多子代理协作，职责分离
- 自动化迭代，持续改进
- 代码审查保证质量

**适用场景：**
- 复杂插件开发
- 需要多次迭代的功能
- 团队协作开发
- CI/CD 集成

---

### 方法 2：API 直接测试

通过 AstrBot API 执行沙箱命令测试插件功能。

**测试脚本：**

完整的测试脚本请参考：`scripts/test-script.py`

**适用场景：**
- 快速单个功能测试
- 交互式调试
- 简单验证
