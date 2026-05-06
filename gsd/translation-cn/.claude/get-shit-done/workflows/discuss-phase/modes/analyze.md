# --analyze 模式 — 每个问题前的权衡表格

> **延迟加载叠加。** 当 `$ARGUMENTS` 中存在 `--analyze` 时，从 `workflows/discuss-phase.md` 读取此文件。可与 default、`--all`、`--chain`、`--text`、`--batch` 组合。

## 效果

在呈现每个问题（或在批处理模式下，问题组）之前，提供该决策的简要**权衡分析**：
- 2-3 个选项，附基于代码库上下文和常见模式的优缺点
- 推荐的方案及推理
- 来自先前阶段的已知陷阱或约束

## 示例

```markdown
**Trade-off analysis: Authentication strategy**

| Approach | Pros | Cons |
|----------|------|------|
| Session cookies | Simple, httpOnly prevents XSS | Requires CSRF protection, sticky sessions |
| JWT (stateless) | Scalable, no server state | Token size, revocation complexity |
| OAuth 2.0 + PKCE | Industry standard for SPAs | More setup, redirect flow UX |

💡 Recommended: OAuth 2.0 + PKCE — your app has social login in requirements (REQ-04) and this aligns with the existing NextAuth setup in `src/lib/auth.ts`.

How should users authenticate?
```

这为用户提供了无需额外提示就能做出明智决策的上下文。

没有 `--analyze` 时，直接呈现问题（无权衡表格）。

## 分析来源

- 优缺点应反映 `scout_codebase` 中加载的代码库上下文和 `load_prior_context` 中呈现的任何先前决策。
- 推荐必须明确关联到项目上下文（例如现有库、先前阶段决策、已记录的需求）。
- 如果相关的 ADR 或规范在 CONTEXT.md 的 `<canonical_refs>` 中被引用，在推荐中引用它。
