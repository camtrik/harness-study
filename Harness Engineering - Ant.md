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

- 交互时间线， 
	- turn1: mcp, call subagents, ask_user_question...
	- turn2: ...
- Harness流水线
- 工具分析
- 复盘总结

### D6 错误复盘
- 错误记录（ERRORS_LAERNED）：现象分析，修复措施
- 模式沉淀（patterns.json）：提炼为结构化规则
- lint-deps.sh：将规则物理化为检查，在直接能执行


### 核心原则
- 仓库是唯一事实来源：不在仓库里，agent就不可见
- AGENTS.md是地图不是手册：80-120行，索引不存知识
- 物理约束>软提示：lnt-deps
- Coordinator不写代码：防止上下文损耗螺旋的首要措施
- 每个task完成后立即验证：
- 确定性步骤不交给模型：git, 构建，lint走脚本
- 报错信息面向agent优化：what+why+how，一条报错就是一次教学（对agent来说）
- 环境设计>prompt调优