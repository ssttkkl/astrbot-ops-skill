# TDD 团队协作模式

当需要进行测试驱动开发时，可以使用多 agent 协作模式。

## 团队角色

### 0. Test Writer（测试编写者）

**职责**：在开发前编写测试用例，定义预期行为

**工作流程**：

1. **理解需求**
   - 阅读功能需求文档
   - 明确输入输出和边界条件

2. **编写测试用例**
   ```bash
   # 创建测试文件
   cd <project_path>/agent-test
   # 编辑测试用例 JSON
   vim test-cases-<feature>.json
   ```

3. **定义测试结构**
   ```json
   {
     "name": "测试名称",
     "description": "测试描述",
     "input": "输入内容",
     "expected_output": "预期输出",
     "expected_exit_code": 0
   }
   ```

4. **运行测试，确保失败（Red）**
   ```bash
   python3 test-script.py test-cases-<feature>.json <username> <password_md5>
   # 测试应该失败，证明测试有效
   ```

**输出**：测试用例文件、预期失败的测试报告

---

### 1. Test Executor（测试执行者）

**职责**：执行测试脚本，分析测试结果，报告失败用例

**工作流程**：

1. **建立 SSH 端口转发**（如果需要）
   ```bash
   ssh -fNL <local_port>:<remote_host>:<remote_port> <user>@<server>
   ```

2. **进入测试目录**
   ```bash
   cd <project_path>/agent-test
   # 或通过 SSH
   ssh <user>@<server> "cd <project_path>/agent-test"
   ```

3. **执行测试脚本**
   ```bash
   # 先读取配置文件获取凭证
   # 从 TOOLS.md 获取 ASTRBOT_DIR
   cat ${ASTRBOT_DIR}/data/cmd_config.json
   
   # 提取 username 和 password（MD5 哈希）
   # username: 通常是 "admin" 或配置文件中的 dashboard.username
   # password_md5: 配置文件中的 dashboard.password（已经是 MD5 哈希）
   
   # 执行测试
   python3 test-script.py <test-cases.json> <username> <password_md5>
   # 示例：
   # python3 test-script.py test-cases-python.json admin d57e17c796fe1a89cd2ccac987f1531b
   ```

4. **读取测试结果文件**
   ```bash
   cat test-results.json
   # 或使用 read 工具
   read: <project_path>/agent-test/test-results.json
   ```

5. **语义判断真实通过率**
   - 不只看 `passed` 字段
   - 检查输出内容是否符合预期
   - 识别错误信息（Error、Failed、Exception）
   - 判断退出码是否符合预期

6. **报告失败用例和原因**
   - 格式：测试名称、预期结果、实际输出、失败原因

**输出**：通过率统计、失败用例列表（名称、预期、实际、原因）

---

### 2. Developer（开发者）

**职责**：根据测试反馈修复代码，提交 git commit

**约束**：
- 只修改本地仓库代码
- 每次只修复一个类别的问题
- 遵循项目代码规范

**工作流程**：
1. 读取测试失败报告
2. 分析失败原因
3. 修改代码
4. 提交到 feature 分支

---

### 3. Code Reviewer（代码审查者）

**职责**：审查代码修改，检查安全性和正确性

**审查清单**：
- 代码符合规范
- 修复逻辑正确
- 没有引入新的安全问题
- 没有破坏现有功能
- 测试覆盖率是否足够

**输出**：通过/需要修改、具体问题列表

---

### 4. Refactorer（重构者）

**职责**：在测试通过后重构代码，提高代码质量

**工作流程**：

1. **识别重构机会**
   - 代码重复
   - 复杂的函数
   - 不清晰的命名
   - 过长的文件

2. **执行重构**
   ```bash
   # 重构代码（保持测试通过）
   # - 提取函数
   # - 重命名变量
   # - 简化逻辑
   # - 消除重复
   ```

3. **运行测试验证**
   ```bash
   # 每次重构后立即测试
   python3 test-script.py test-cases-all.json <username> <password_md5>
   ```

4. **提交重构**
   ```bash
   git commit -m "refactor: <description>"
   ```

**约束**：
- 重构不改变功能
- 每次重构后必须运行测试
- 小步重构，频繁提交

**输出**：重构后的代码、测试通过报告

---

### 5. Coverage Checker（覆盖率检查者）

**职责**：检查测试覆盖率，识别未测试的代码

**工作流程**：

1. **生成覆盖率报告**
   ```bash
   # Python 项目
   pytest --cov=<module> --cov-report=html
   # 或使用 coverage.py
   coverage run -m pytest
   coverage report
   coverage html
   ```

2. **分析覆盖率**
   - 行覆盖率（Line Coverage）
   - 分支覆盖率（Branch Coverage）
   - 函数覆盖率（Function Coverage）

3. **识别未覆盖代码**
   ```bash
   # 查看未覆盖的行
   coverage report --show-missing
   ```

4. **报告缺失测试**
   - 列出未测试的函数
   - 列出未测试的分支
   - 建议补充的测试用例

**输出**：覆盖率报告、缺失测试列表

---

### 6. Deployer（部署者）

**职责**：部署代码到目标环境，验证服务正常

**工作流程**：

1. **推送代码到代码仓库**
   ```bash
   git push origin <branch>
   # 示例：
   # git push origin feature/fix_test_failure
   ```

2. **SSH 到服务器**
   ```bash
   # 从 TOOLS.md 读取服务器地址
   ssh <user>@<server>
   # 示例：
   # ssh root@123.57.192.94
   ```

3. **拉取最新代码**
   ```bash
   # 从 TOOLS.md 读取 ASTRBOT_DIR
   cd ${ASTRBOT_DIR}/data/plugins/<plugin_name>
   git pull origin <branch>
   # 验证 commit hash
   git log -1 --oneline
   # 示例：
   # cd /opt/astrbot/data/plugins/astrbot_plugin_nsjail
   ```

4. **重启服务**
   ```bash
   # 如果修改了 Dockerfile 或 docker-compose.yml，需要重新构建
   cd ${ASTRBOT_DIR}
   docker-compose down
   docker-compose up -d --build
   
   # 如果只修改了插件代码，直接重启即可
   docker restart <container_name>
   # 或 docker-compose 方式
   docker-compose restart <service_name>
   
   # 示例：
   # docker restart astrbot
   ```

5. **验证服务启动**
   ```bash
   # 检查容器状态
   docker ps | grep <container_name>
   # 检查日志
   docker logs <container_name> --tail 20
   # 或检查进程
   ps aux | grep <process_name>
   ```

---

## 交接协议

### Developer → Code Reviewer

```markdown
**Commit Hash**: <hash>
**修改内容**：
- 文件：<file>
- 行数：L<start>-L<end>
- 变更：<description>
**测试方法**：<command>
**已知问题**：<issues>
```

### Code Reviewer → Deployer

```markdown
**审查结果**：通过
**Commit Hash**: <hash>
**部署步骤**：<steps>
**验证方法**：<command>
```

### Deployer → Test Executor

```markdown
**部署完成**：
**Commit Hash**: <hash>
- 分支：<branch>
- 服务状态：运行中
**测试类别**：<category>
**预期改善**：<expected>
```

---

## 迭代流程

### 标准 TDD 流程（Red-Green-Refactor）

```
完整 TDD 循环：

1. Test Writer：编写新测试用例
   - 定义预期行为
   - 编写测试代码
   - 运行测试，确保失败（Red）
   
2. Developer：编写最小代码
   - 只写让测试通过的代码
   - 不过度设计
   - 运行测试，确保通过（Green）
   
3. Code Reviewer：代码审查
   - 检查代码质量
   - 检查测试覆盖率
   - 批准或要求修改
   
4. Refactorer：重构代码（Refactor）
   - 优化代码结构
   - 消除重复
   - 提高可读性
   - 运行测试，确保仍然通过
   
5. Test Executor：回归测试
   - 运行全部测试套件
   - 确保没有破坏现有功能
   - 报告测试覆盖率
   
6. Deployer：部署到环境
   
7. Test Executor：验收测试
   - 在真实环境测试
   - 验证功能正确性
```

### 测试修复流程（当前实现）

```
1. Orchestrator：选择下一个优先级任务
2. Test Executor：执行测试，报告失败
3. Developer：修复代码，提交 commit
4. Code Reviewer：审查代码
   - 如果不通过 → 返回 Developer
5. Deployer：部署到环境
6. Test Executor：重新测试
   - 如果失败 → 返回 Developer
   - 如果通过 → 标记 Done
7. Orchestrator：汇总结果，决定是否继续
```

### 两种流程的区别

- **标准 TDD**：先写测试 → 测试失败 → 写代码 → 测试通过 → 重构
- **测试修复**：测试失败 → 修复代码 → 测试通过（适用于已有代码的测试修复）
