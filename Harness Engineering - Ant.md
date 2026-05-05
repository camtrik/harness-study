### D1 上下文管理
- AGENTS.md作为入口文件
- ARCHITECTURE.md和development.md分别提供了分层架构说明和开发环境说明
- 按需提供输入（skills，rules加载原理）
- ？依赖第三方契约进行字段解析时，可以通过hook注入的机制

### D2 工具系统
- 需求分析：brainstorming
- 方案设计：writing-plans
- 编码实现：harness-executor
- 代码审查：code-review
- 代码提交：code-submit
- 部署触发：deploy-trigger
- 自动化测试的mcp工具
- 一些标准化脚本，包括build，test，linter等

### D3 执行编排
Coordinator-Subagent分离架构。leader agent作为协调者负责规划。不参与代码的编写。
- 需求分析->方案设计->实现计划->编码实现->质量验证->代码审查->代码提交->任务复盘 

### D4 状态与记忆
定义JSON schema来管理任务状态和记录。task-meta, checkpoint, execution-trace 

### D5 评估与观测
建设dashboard来观测每次会话的过程，工具调用等。  
