<purpose>
生成、更新和验证所有项目文档 — 包括规范文档类型和现有手写文档。编排器检测项目的文档结构，组装一个追踪每个项目的工作清单，在波次间分派并行的文档编写和文档验证 agent，审查现有文档的准确性，识别文档差距，并通过有界的修复循环修复不准确之处。所有状态都持久化在工作清单中，以便在步骤之间不会丢失任何工作项。输出：经过实时代码库验证的完整、结构感知的文档。
</purpose>

<available_agent_types>
- gsd-doc-writer — 编写和更新项目文档文件
- gsd-doc-verifier — 根据实时代码库验证文档中的事实性声明
</available_agent_types>

<process>

<step name="init_context">
加载 docs-update 上下文，提取 doc_writer_model、commit_docs、existing_docs 数组、project_type 布尔信号、doc_tooling、monorepo_workspaces 和 project_root。
</step>

<step name="classify_project">
将 project_type 布尔信号映射到主要类型标签（monorepo/cli-tool/saas/open-source-library/generic），并收集条件文档信号（has_api_routes → API.md，is_open_source → CONTRIBUTING.md，has_deploy_config → DEPLOYMENT.md）。
</step>

<step name="build_doc_queue">
组装完整的文档队列：6 个始终包含的文档（README、ARCHITECTURE、GETTING-STARTED、DEVELOPMENT、TESTING、CONFIGURATION）加上最多 3 个条件文档。CHANGELOG.md 永远不排队。检查手写文档以进行准确性审查。检测代码库中的文档差距。
</step>

<step name="resolve_modes">
对队列中的每个文档确定 create（新文件）还是 update（现有文件）模式。按层级解析路径（如果项目使用分组子目录结构）。将工作清单持久化到 `.planning/tmp/docs-work-manifest.json`。
</step>

<step name="preservation_check">
检查队列中的手写文档（无 GSD 标记），收集 preserve/supplement/regenerate 的用户决策。如果 AskUserQuestion 不可用，默认 preserve。
</step>

<step name="dispatch_wave_1">
启动 3 个并行 gsd-doc-writer agent（README、ARCHITECTURE、CONFIGURATION）。这些是基础文档，没有交叉引用需求。使用 `run_in_background=true`。

> **编排器规则 — CODEX 运行时**：调用所有波次 1 Task() 后，等待所有 agent 完成后再继续。
</step>

<step name="collect_wave_1">
等待所有波次 1 agent 完成。验证文件存在。更新清单状态。
</step>

<step name="dispatch_wave_2">
启动波次 2 的 agent（GETTING-STARTED、DEVELOPMENT、TESTING 以及条件文档 API、DEPLOYMENT、CONTRIBUTING）。可以引用波次 1 的输出。
</step>

<step name="collect_wave_2">
等待所有波次 2 agent 完成。验证文件存在。
</step>

<step name="dispatch_monorepo_packages">
如果 monorepo_workspaces 非空，为每个包含 package.json 的工作区包生成 README。
</step>

<step name="sequential_generation">
如果 Task 工具不可用（Antigravity、Gemini CLI、Codex、Copilot），在当前上下文中顺序生成文档。
</step>

<step name="verify_docs">
根据实时代码库验证所有文档（规范的和手写的）中的事实性声明。启动 gsd-doc-verifier agent。呈现合并的验证摘要。
</step>

<step name="fix_loop">
用发现的修复模式纠正标记的不准确之处（最多 2 次迭代）。如果检测到回归则立即停止。
</step>

<step name="scan_for_secrets">
在提交之前扫描生成/更新的文档中是否意外泄露密钥。
</step>

<step name="commit_docs">
如果 commit_docs 为 true，提交生成的文档文件。
</step>

<step name="report">
呈现完成摘要，包括生成的文档表、模式、行数以及任何跳过/失败/验证结果。
</step>

</process>

<success_criteria>
- [ ] docs-init JSON 已加载并提取所有字段
- [ ] 项目类型已从 project_type 信号正确分类
- [ ] 文档队列包含所有始终包含的文档加上匹配项目信号的条件文档
- [ ] 未生成或排队 CHANGELOG.md
- [ ] 每个文档以正确模式生成
- [ ] 波次 1 文档在波次 2 开始之前完成
- [ ] 生成的文档不包含 GSD 方法论内容
- [ ] 所有生成的文件已提交（如果 commit_docs 为 true）
- [ ] 手写文档在分派前提示 preserve/supplement/regenerate（除非 --force）
- [ ] verify_docs 步骤已检查所有生成的文档
- [ ] fix_loop 最多运行 2 次迭代并在回归时停止
- [ ] scan_for_secrets 在提交前运行
</success_criteria>
