# Go Fractals CLI - 实现计划

使用 `superpowers:subagent-driven-development` skill 执行此计划。

## 上下文

构建一个生成 ASCII 分形图案的 CLI 工具。完整规范见 `design.md`。

## 任务

### 任务 1：项目搭建

创建 Go module 和目录结构。

**执行：**
- 使用 module 名称 `github.com/superpowers-test/fractals` 初始化 `go.mod`
- 创建目录结构：`cmd/fractals/`、`internal/sierpinski/`、`internal/mandelbrot/`、`internal/cli/`
- 创建最小的 `cmd/fractals/main.go`，打印 "fractals cli"
- 添加 `github.com/spf13/cobra` 依赖

**验证：**
- `go build ./cmd/fractals` 成功
- `./fractals` 打印 "fractals cli"

---

### 任务 2：CLI 框架与帮助

搭建 Cobra 根命令并输出帮助信息。

**执行：**
- 创建 `internal/cli/root.go`，包含根命令
- 配置帮助文本，显示可用子命令
- 将根命令接入 `main.go`

**验证：**
- `./fractals --help` 显示用法说明，列出 "sierpinski" 和 "mandelbrot" 作为可用命令
- `./fractals`（无参数）显示帮助信息

---

### 任务 3：Sierpinski 算法

实现 Sierpinski 三角形生成算法。

**执行：**
- 创建 `internal/sierpinski/sierpinski.go`
- 实现 `Generate(size, depth int, char rune) []string`，返回三角形的各行
- 使用递归中点细分算法
- 创建 `internal/sierpinski/sierpinski_test.go`，包含以下测试：
  - 小三角形（size=4, depth=2）匹配预期输出
  - size=1 返回单个字符
  - depth=0 返回填充的三角形

**验证：**
- `go test ./internal/sierpinski/...` 通过

---

### 任务 4：Sierpinski CLI 集成

将 Sierpinski 算法接入 CLI 子命令。

**执行：**
- 创建 `internal/cli/sierpinski.go`，包含 `sierpinski` 子命令
- 添加标志：`--size`（默认 32）、`--depth`（默认 5）、`--char`（默认 '*'）
- 调用 `sierpinski.Generate()` 并将结果打印到 stdout

**验证：**
- `./fractals sierpinski` 输出三角形
- `./fractals sierpinski --size 16 --depth 3` 输出更小的三角形
- `./fractals sierpinski --help` 显示标志文档

---

### 任务 5：Mandelbrot 算法

实现 Mandelbrot 集合 ASCII 渲染器。

**执行：**
- 创建 `internal/mandelbrot/mandelbrot.go`
- 实现 `Render(width, height, maxIter int, char string) []string`
- 将复平面区域（实部 -2.5 到 1.0，虚部 -1.0 到 1.0）映射到输出尺寸
- 将迭代次数映射到字符渐变 " .:-=+*#%@"（如果提供了单个字符则使用该字符）
- 创建 `internal/mandelbrot/mandelbrot_test.go`，包含以下测试：
  - 输出尺寸匹配请求的 width/height
  - 集合内的已知点 (0,0) 映射到最大迭代次数字符
  - 集合外的已知点 (2,0) 映射到低迭代次数字符

**验证：**
- `go test ./internal/mandelbrot/...` 通过

---

### 任务 6：Mandelbrot CLI 集成

将 Mandelbrot 算法接入 CLI 子命令。

**执行：**
- 创建 `internal/cli/mandelbrot.go`，包含 `mandelbrot` 子命令
- 添加标志：`--width`（默认 80）、`--height`（默认 24）、`--iterations`（默认 100）、`--char`（默认 ""）
- 调用 `mandelbrot.Render()` 并将结果打印到 stdout

**验证：**
- `./fractals mandelbrot` 输出可辨认的 Mandelbrot 集合
- `./fractals mandelbrot --width 40 --height 12` 输出版本更小
- `./fractals mandelbrot --help` 显示标志文档

---

### 任务 7：字符集配置

确保 `--char` 标志在两个命令中行为一致。

**执行：**
- 验证 Sierpinski 的 `--char` 标志将字符传递给算法
- 对于 Mandelbrot，`--char` 应使用单个字符而非渐变字符集
- 为自定义字符输出添加测试

**验证：**
- `./fractals sierpinski --char '#'` 使用 '#' 字符
- `./fractals mandelbrot --char '.'` 在所有填充点使用 '.'
- 测试通过

---

### 任务 8：输入验证与错误处理

为无效输入添加验证。

**执行：**
- Sierpinski：size 必须 > 0，depth 必须 >= 0
- Mandelbrot：width/height 必须 > 0，iterations 必须 > 0
- 对无效输入返回清晰的错误信息
- 为错误用例添加测试

**验证：**
- `./fractals sierpinski --size 0` 打印错误，以非零状态退出
- `./fractals mandelbrot --width -1` 打印错误，以非零状态退出
- 错误信息清晰有用

---

### 任务 9：集成测试

添加调用 CLI 的集成测试。

**执行：**
- 创建 `cmd/fractals/main_test.go` 或 `test/integration_test.go`
- 为两个命令测试完整的 CLI 调用
- 验证输出格式和退出码
- 测试错误用例返回非零退出码

**验证：**
- `go test ./...` 通过所有测试，包括集成测试

---

### 任务 10：README

编写使用文档和示例。

**执行：**
- 创建 `README.md`，包含：
  - 项目描述
  - 安装：`go install ./cmd/fractals`
  - 两个命令的使用示例
  - 示例输出（小样本）

**验证：**
- README 准确描述该工具
- README 中的示例确实可用
