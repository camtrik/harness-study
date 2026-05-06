# iOS 应用脚手架参考

搭建 iOS 应用的规则和模式。任何涉及创建新 iOS app target 的计划都需应用。

---

## 关键规则：绝不使用 Package.swift 作为 iOS 应用的主构建系统

**绝不要使用 `Package.swift` 的 `.executableTarget`（或 `.target`）来搭建 iOS 应用。** Swift Package Manager 的 executable target 编译为 macOS 命令行工具——它们不会产生 `.app` 包，无法为 iOS 设备签名，也无法提交到 App Store。

**禁止的模式：**
```swift
// Package.swift — 不要用于 iOS 应用
.executableTarget(name: "MyApp", dependencies: [])
// 或
.target(name: "MyApp", dependencies: [])
```

使用此模式会生成 macOS CLI 二进制，而非 iOS 应用。应用将无法为任何 iOS 模拟器或设备构建。

---

## 必需模式：XcodeGen

所有 iOS 应用脚手架必须使用 XcodeGen 生成 `.xcodeproj`。

### 步骤 1 — 安装 XcodeGen（如果尚未安装）

```bash
brew install xcodegen
```

### 步骤 2 — 创建 `project.yml`

`project.yml` 是 XcodeGen 规范，描述项目结构。最小可行规范：

```yaml
name: MyApp
options:
  bundleIdPrefix: com.example
  deploymentTarget:
    iOS: "17.0"
settings:
  SWIFT_VERSION: "5.10"
  IPHONEOS_DEPLOYMENT_TARGET: "17.0"
targets:
  MyApp:
    type: application
    platform: iOS
    sources: [Sources/MyApp]
    settings:
      PRODUCT_BUNDLE_IDENTIFIER: com.example.MyApp
      INFOPLIST_FILE: Sources/MyApp/Info.plist
    scheme:
      testTargets:
        - MyAppTests
  MyAppTests:
    type: bundle.unit-test
    platform: iOS
    sources: [Tests/MyAppTests]
    dependencies:
      - target: MyApp
```

### 步骤 3 — 生成 .xcodeproj

```bash
xcodegen generate
```

这将在项目根目录创建 `MyApp.xcodeproj`。提交 `project.yml`，但将 `*.xcodeproj` 添加到 `.gitignore` 中（检出时重新生成）。

### 步骤 4 — 标准项目布局

```
MyApp/
├── project.yml              # XcodeGen 规范——提交它
├── .gitignore               # 包含 *.xcodeproj
├── Sources/
│   └── MyApp/
│       ├── MyAppApp.swift   # @main 入口点
│       ├── ContentView.swift
│       └── Info.plist
└── Tests/
    └── MyAppTests/
        └── MyAppTests.swift
```

---

## iOS 部署目标兼容性

在使用任何 SwiftUI 组件之前，始终检查 SwiftUI API 可用性与项目的 `IPHONEOS_DEPLOYMENT_TARGET`。

| API | 最低 iOS |
|-----|-------------|
| `NavigationView` | iOS 13 |
| `NavigationStack` | iOS 16 |
| `NavigationSplitView` | iOS 16 |
| 带多选的 `List(selection:)` | iOS 17 |
| `ScrollView` 滚动位置 API | iOS 17 |
| `Observable` 宏（`@Observable`） | iOS 17 |
| `SwiftData` | iOS 17 |
| `@Bindable` | iOS 17 |
| `TipKit` | iOS 17 |

**规则：** 如果计划需求使用的 SwiftUI API 超过项目的部署目标，要么：
1. 在 `project.yml` 中提高部署目标（并记录决策），或
2. 用 `if #available(iOS NN, *) { ... }` 包裹调用并附带回退实现。

不要静默使用需要高于声明的部署目标的 iOS 版本的 API——在旧设备上运行时应用将崩溃。

---

## 验证

运行 `xcodegen generate` 后，验证项目可构建：

```bash
xcodebuild -project MyApp.xcodeproj -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16' build
```

构建成功（exit code 0）确认脚手架对 iOS 有效。
