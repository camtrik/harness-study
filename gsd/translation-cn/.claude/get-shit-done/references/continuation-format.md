# 继续格式

完成命令或工作流后呈现下一步的标准格式。

## 核心结构

```
---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**{标识符}：{名称}** — {一行描述}

`/clear` 然后：

`{可复制粘贴的命令}`

---

**也可选择：**
- `{替代选项 1}` — 描述
- `{替代选项 2}` — 描述

---
```

> 如果 init 上下文中未设置 `project_code`，则省略项目标识后缀：
> `## ▶ Next Up`（不含 ` — [CODE] Title`）。

## 格式规则

1. **始终显示是什么**——名称 + 描述，绝不仅仅是命令路径
2. **从来源获取上下文**——阶段从 ROADMAP.md，计划从 PLAN.md `<objective>`
3. **命令使用行内代码**——反引号，易于复制粘贴，渲染为可点击的链接
4. **先 `/clear`**——始终在命令前显示 `/clear`，以便用户按正确顺序运行
5. **"也可选择"而非"其他选项"**——听起来更像应用风格
6. **视觉分隔符**——上下用 `---` 以突出显示
7. **标题中的项目身份**——包含 init 上下文中的 `[PROJECT_CODE] PROJECT_TITLE`，使交接在不同会话间自我标识。如果 `project_code` 未设置，完全省略后缀（仅 `## ▶ Next Up`）

## 变体

### 执行下一个计划

```
---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**02-03：刷新 Token 轮换** — 添加带滑动过期的 /api/auth/refresh

`/clear` 然后：

`/gsd-execute-phase 2`

---

**也可选择：**
- 执行前审查计划
- `/gsd-list-phase-assumptions 2` — 检查假设

---
```

### 执行阶段中的最后一个计划

添加说明这是最后一个计划以及后续内容：

```
---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**02-03：刷新 Token 轮换** — 添加带滑动过期的 /api/auth/refresh
<sub>阶段 2 中的最终计划</sub>

`/clear` 然后：

`/gsd-execute-phase 2`

---

**完成后：**
- 阶段 2 → 阶段 3 过渡
- 下一步：**阶段 3：核心功能** — 用户仪表盘和设置

---
```

### 规划一个阶段

```
---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**阶段 2：认证** — 带刷新 Token 的 JWT 登录流程

`/clear` 然后：

`/gsd-plan-phase 2`

---

**也可选择：**
- `/gsd-discuss-phase 2` — 先收集上下文
- `/gsd-plan-phase --research-phase 2` — 调查未知
- 审查路线图

---
```

### 阶段完成，准备下一阶段

在下一操作之前显示完成状态：

```
---

## ✓ 阶段 2 完成

3/3 计划已执行

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**阶段 3：核心功能** — 用户仪表盘、设置和数据导出

`/clear` 然后：

`/gsd-plan-phase 3`

---

**也可选择：**
- `/gsd-discuss-phase 3` — 先收集上下文
- `/gsd-plan-phase --research-phase 3` — 调查未知
- 审查阶段 2 构建的内容

---
```

### 多个等价选项

当没有明确的主要操作时：

```
---

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**阶段 3：核心功能** — 用户仪表盘、设置和数据导出

`/clear` 然后选择其一：

**直接规划：** `/gsd-plan-phase 3`

**先讨论上下文：** `/gsd-discuss-phase 3`

**研究未知：** `/gsd-plan-phase --research-phase 3`

---
```

### 里程碑完成

```
---

## 🎉 里程碑 v1.0 完成

所有 4 个阶段已交付

## ▶ Next Up — [${PROJECT_CODE}] ${PROJECT_TITLE}

**开始 v1.1** — 询问 → 研究 → 需求 → 路线图

`/clear` 然后：

`/gsd-new-milestone`

---
```

## 拉取上下文

### 对于阶段（来自 ROADMAP.md）：

```markdown
### 阶段 2：认证
**目标**：带刷新 Token 的 JWT 登录流程
```

提取：`**阶段 2：认证** — 带刷新 Token 的 JWT 登录流程`

### 对于计划（来自 ROADMAP.md）：

```markdown
计划：
- [ ] 02-03：添加刷新 Token 轮换
```

或来自 PLAN.md `<objective>`：

```xml
<objective>
添加带滑动过期窗口的刷新 Token 轮换。

目的：在不损害安全性的情况下延长会话生命周期。
</objective>
```

提取：`**02-03：刷新 Token 轮换** — 添加带滑动过期的 /api/auth/refresh`

## 反模式

### 不要：仅有命令（无上下文）

```
## 继续

运行 `/clear`，然后粘贴：
/gsd-execute-phase 2
```

用户不知道 02-03 是关于什么的。

### 不要：缺少 /clear 说明

```
`/gsd-plan-phase 3`

先运行 /clear。
```

没有解释为什么。用户可能跳过。

### 不要："其他选项"措辞

```
其他选项：
- 审查路线图
```

听起来像事后想法。使用"也可选择："代替。

### 不要：用代码块包裹命令

```
```
/gsd-plan-phase 3
```
```

代码块在模板中创建嵌套歧义。使用行内反引号代替。
