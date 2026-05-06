# 导入文档工作流

扫描仓库中的混合规划文档（ADR、PRD、SPEC、DOC），将其综合为合并的上下文，并引导或合并到 `.planning/`。

参数：`[path]`（可选目标目录）、`--mode new|merge`、`--manifest <file>`、`--resolve auto|interactive`（v1 仅支持 auto）。

流程：解析参数→初始化并检测模式→发现文档（清单/目录规范/内容启发式，上限50个）→并行分类→综合→冲突门→路由（new模式/merge模式）→最终确定。
