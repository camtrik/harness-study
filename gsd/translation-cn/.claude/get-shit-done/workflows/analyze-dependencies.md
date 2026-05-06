<purpose>
在执行前分析 ROADMAP.md 中各阶段的依赖关系。检测阶段之间的文件重叠、语义 API/数据流依赖，并建议 `Depends on` 条目，以防止 `/gsd-manager` 并行执行时发生合并冲突。
</purpose>

<process>

## 1. 加载 ROADMAP.md

读取 `.planning/ROADMAP.md`。如果不存在，报错："No ROADMAP.md found — run `/gsd-new-project` first."

提取所有阶段。对于每个阶段捕获：
- 阶段编号和名称
- 范围/目标描述
- `Files` 或 `files_modified` 字段中列出的文件（如果有的话）
- 现有的 `Depends on` 字段值

## 2. 推断可能的文件修改

对于每个没有明确 `files_modified` 的阶段，分析范围/目标描述以推断哪些文件可能会被修改。使用以下启发式方法：

- **数据库/schema 阶段** → 迁移文件、schema 定义、model 文件
- **API/后端阶段** → 路由文件、controller 文件、service 文件、handler 文件
- **前端/UI 阶段** → 组件文件、页面文件、样式文件
- **认证阶段** → 中间件文件、认证路由文件、session/token 文件
- **配置/基础设施阶段** → 配置文件、环境文件、CI/CD 文件
- **测试阶段** → 测试文件、spec 文件、fixture 文件
- **共享工具阶段** → lib/utils 文件、共享类型定义

按推断的文件领域（数据库、API、前端、认证、配置、共享）对阶段进行分组。

## 3. 检测依赖关系

对于每对阶段（A、B），检查依赖信号：

### 文件重叠检测
如果阶段 A 和 B 都将修改同一领域或同一特定文件中的文件，则一个必须先于另一个运行。提供基础的阶段先运行。

### 语义依赖检测
读取每个阶段的范围/目标，查找以下模式：
- 阶段 B 提到使用、消费或调用阶段 A 创建/实现的东西
- 阶段 B 引用了阶段 A 构建的"API"、"schema"、"model"、"endpoint"或"interface"
- 阶段 B 说"after X is complete"、"once X is built"、"using the X from Phase N"
- 阶段 B 扩展或修改阶段 A 建立的代码

### 数据流检测
- 阶段 A 创建数据结构、schema 或类型 → 阶段 B 消费或转换它们
- 阶段 A 播种/迁移数据库 → 阶段 B 从该数据库读取
- 阶段 A 暴露 API 契约 → 阶段 B 实现该契约的客户端

## 4. 构建依赖表

输出依赖建议表（格式包含阶段详情和建议的依赖关系及原因）。

## 5. 总结建议的更改

显示 ROADMAP.md 建议 `Depends on` 更改的合并差异。

## 6. 确认并应用

询问用户："Apply these `Depends on` suggestions to ROADMAP.md? (yes / no / edit)"
- yes、no 或 edit 对应的处理逻辑。

</process>
