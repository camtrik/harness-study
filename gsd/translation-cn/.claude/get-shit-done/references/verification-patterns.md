# 验证模式

如何验证不同类型的产物是真实实现而非桩或占位符。

<core_principle>
**存在 ≠ 实现**

文件存在不意味着功能工作。验证必须检查：
1. **存在**——文件在预期路径存在
2. **实质性**——内容是真实实现，而非占位符
3. **已接入**——连接到系统其余部分
4. **功能正常**——调用时实际工作

级别 1-3 可以程序化检查。级别 4 通常需要人工验证。
</core_principle>

<stub_detection>

## 通用桩模式

这些模式不分文件类型均表示占位符代码：

**基于注释的桩：**
```bash
# 用于桩注释的 grep 模式
grep -E "(TODO|FIXME|XXX|HACK|PLACEHOLDER)" "$file"
grep -E "implement|add later|coming soon|will be" "$file" -i
grep -E "// \.\.\.|/\* \.\.\. \*/|# \.\.\." "$file"
```

**输出中的占位符文本：**
```bash
# UI 占位符模式
grep -E "placeholder|lorem ipsum|coming soon|under construction" "$file" -i
grep -E "sample|example|test data|dummy" "$file" -i
grep -E "\[.*\]|<.*>|\{.*\}" "$file"  # 遗留的模板括号
```

**空或简单实现：**
```bash
# 什么都不做的函数
grep -E "return null|return undefined|return \{\}|return \[\]" "$file"
grep -E "pass$|\.\.\.|\bnothing\b" "$file"
grep -E "console\.(log|warn|error).*only" "$file"  # 仅日志函数
```

**预期动态但硬编码的值：**
```bash
# 硬编码 ID、计数或内容
grep -E "id.*=.*['\"].*['\"]" "$file"  # 硬编码字符串 ID
grep -E "count.*=.*\d+|length.*=.*\d+" "$file"  # 硬编码计数
grep -E "\\\$\d+\.\d{2}|\d+ items" "$file"  # 硬编码显示值
```

</stub_detection>

<react_components>

## React/Next.js 组件

**存在性检查：**
```bash
# 文件存在且导出组件
[ -f "$component_path" ] && grep -E "export (default |)function|export const.*=.*\(" "$component_path"
```

**实质性检查：**
```bash
# 返回实际 JSX，而非占位符
grep -E "return.*<" "$component_path" | grep -v "return.*null" | grep -v "placeholder" -i

# 具有有意义的内容（不仅是包裹 div）
grep -E "<[A-Z][a-zA-Z]+|className=|onClick=|onChange=" "$component_path"

# 使用 props 或 state（非静态）
grep -E "props\.|useState|useEffect|useContext|\{.*\}" "$component_path"
```

**React 特定的桩模式：**
```javascript
// 红色警告——这些是桩：
return <div>Component</div>
return <div>Placeholder</div>
return <div>{/* TODO */}</div>
return <p>Coming soon</p>
return null
return <></>

// 也是桩——空处理器：
onClick={() => {}}
onChange={() => console.log('clicked')}
onSubmit={(e) => e.preventDefault()}  // 仅阻止默认，什么都不做
```

**接入检查：**
```bash
# 组件导入它需要的内容
grep -E "^import.*from" "$component_path"

# Props 实际被使用（不仅是接收）
# 查找解构或 props.X 使用
grep -E "\{ .* \}.*props|\bprops\.[a-zA-Z]+" "$component_path"

# API 调用存在（用于获取数据的组件）
grep -E "fetch\(|axios\.|useSWR|useQuery|getServerSideProps|getStaticProps" "$component_path"
```

**功能验证（需要人工）：**
- 组件是否渲染可见内容？
- 交互元素是否响应点击？
- 数据是否加载并显示？
- 错误状态是否适当显示？

</react_components>

<api_routes>

## API 路由（Next.js App Router / Express / 等）

**存在性检查：**
```bash
# 路由文件存在
[ -f "$route_path" ]

# 导出 HTTP 方法处理器（Next.js App Router）
grep -E "export (async )?(function|const) (GET|POST|PUT|PATCH|DELETE)" "$route_path"

# 或 Express 风格处理器
grep -E "\.(get|post|put|patch|delete)\(" "$route_path"
```

**实质性检查：**
```bash
# 具有实际逻辑，不仅是 return 语句
wc -l "$route_path"  # 超过 10-15 行暗示真实实现

# 与数据源交互
grep -E "prisma\.|db\.|mongoose\.|sql|query|find|create|update|delete" "$route_path" -i

# 具有错误处理
grep -E "try|catch|throw|error|Error" "$route_path"

# 返回有意义的响应
grep -E "Response\.json|res\.json|res\.send|return.*\{" "$route_path" | grep -v "message.*not implemented" -i
```

**API 路由特定的桩模式：**
```typescript
// 红色警告——这些是桩：
export async function POST() {
  return Response.json({ message: "Not implemented" })
}

export async function GET() {
  return Response.json([])  // 空数组，无数据库查询
}

export async function PUT() {
  return new Response()  // 空响应
}

// 仅 console log：
export async function POST(req) {
  console.log(await req.json())
  return Response.json({ ok: true })
}
```

</api_routes>

<database_schema>

## 数据库模式（Prisma / Drizzle / SQL）

**存在性检查：**
```bash
[ -f "prisma/schema.prisma" ] || [ -f "drizzle/schema.ts" ] || [ -f "src/db/schema.sql" ]

grep -E "^model $model_name|CREATE TABLE $table_name|export const $table_name" "$schema_path"
```

**实质性检查：**
```bash
# 具有预期字段（不仅是 id）
grep -A 20 "model $model_name" "$schema_path" | grep -E "^\s+\w+\s+\w+"

# 具有关系（如预期）
grep -E "@relation|REFERENCES|FOREIGN KEY" "$schema_path"

# 具有适当的字段类型（不仅是 String）
grep -A 20 "model $model_name" "$schema_path" | grep -E "Int|DateTime|Boolean|Float|Decimal|Json"
```

</database_schema>

<hooks_utilities>

## 自定义 Hook 和工具

**接入检查：**
```bash
# Hook 实际在某处被导入
grep -r "import.*$hook_name" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"

# Hook 实际被调用
grep -r "$hook_name()" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"
```

</hooks_utilities>

<wiring_verification>

## 接入验证模式

接入验证检查组件实际通信。这是大多数桩隐藏的地方。

### 模式：组件 → API

**检查：** 组件是否实际调用 API？

```bash
grep -E "fetch\(['\"].*$api_path|axios\.(get|post).*$api_path" "$component_path"
grep -E "fetch\(|axios\." "$component_path" | grep -v "^.*//.*fetch"
grep -E "await.*fetch|\.then\(|setData|setState" "$component_path"
```

### 模式：API → 数据库

**检查：** API 路由是否实际查询数据库？

### 模式：表单 → 处理器

**检查：** 表单提交是否实际做了什么？

### 模式：状态 → 渲染

**检查：** 组件是否渲染状态，而非硬编码内容？

</wiring_verification>

<verification_checklist>

## 快速验证清单

### 组件清单
- [ ] 文件在预期路径存在
- [ ] 导出函数/const 组件
- [ ] 返回 JSX（非 null/空）
- [ ] 渲染中无占位符文本
- [ ] 使用 props 或 state（非静态）
- [ ] 事件处理器有真实实现
- [ ] 导入正确解析
- [ ] 在应用中的某处被使用

### API 路由清单
- [ ] 文件在预期路径存在
- [ ] 导出 HTTP 方法处理器
- [ ] 处理器超过 5 行
- [ ] 查询数据库或服务
- [ ] 返回有意义的响应（非空/占位符）
- [ ] 具有错误处理
- [ ] 验证输入
- [ ] 从前端调用

### 接入清单
- [ ] 组件 → API：fetch/axios 调用存在且使用响应
- [ ] API → 数据库：查询存在且结果被返回
- [ ] 表单 → 处理器：onSubmit 调用 API/mutation
- [ ] 状态 → 渲染：状态变量出现在 JSX 中

</verification_checklist>

<automated_verification_script>

## 自动化验证方法

对于验证子 agent，使用此模式：

```bash
# 1. 检查存在性
check_exists() {
  [ -f "$1" ] && echo "EXISTS: $1" || echo "MISSING: $1"
}

# 2. 检查桩模式
check_stubs() {
  local file="$1"
  local stubs=$(grep -c -E "TODO|FIXME|placeholder|not implemented" "$file" 2>/dev/null || echo 0)
  [ "$stubs" -gt 0 ] && echo "STUB_PATTERNS: $stubs in $file"
}

# 3. 检查接入（组件调用 API）
check_wiring() {
  local component="$1"
  local api_path="$2"
  grep -q "$api_path" "$component" && echo "WIRED: $component → $api_path" || echo "NOT_WIRED: $component → $api_path"
}

# 4. 检查实质性（超过 N 行，具有预期模式）
check_substantive() {
  local file="$1"
  local min_lines="$2"
  local pattern="$3"
  local lines=$(wc -l < "$file" 2>/dev/null || echo 0)
  local has_pattern=$(grep -c -E "$pattern" "$file" 2>/dev/null || echo 0)
  [ "$lines" -ge "$min_lines" ] && [ "$has_pattern" -gt 0 ] && echo "SUBSTANTIVE: $file" || echo "THIN: $file ($lines lines, $has_pattern matches)"
}
```

对每个 must-have 产物运行这些检查。将结果聚合到 VERIFICATION.md。

</automated_verification_script>

<human_verification_triggers>

## 何时需要人工验证

有些东西无法程序化验证。将这些标记为人工测试：

**始终人工：**
- 视觉外观（看起来对吗？）
- 用户流完成度（你能实际做到那件事吗？）
- 实时行为（WebSocket、SSE）
- 外部服务集成（Stripe、邮件发送）
- 错误消息清晰度（消息有帮助吗？）
- 性能感觉（感觉快吗？）

**不确定时人工：**
- grep 无法追踪的复杂接入
- 依赖状态的动态行为
- 边界情况和错误状态
- 移动端响应式
- 无障碍

</human_verification_triggers>
