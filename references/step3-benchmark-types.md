# Step 3 Benchmark 类型参考

本文档是评估阶段的**详细参考手册**，包含 5 种 Benchmark 类型的完整详解、模板、禁止描述完整列表、Evaluation 详解和完整示例。

> 本文档由 [step3-evaluate.md](../steps/step3-evaluate.md) 按需加载引用。

---

## 目录

- [1. Benchmark 类型详解](#1-benchmark-类型详解)
  - [1.1 file_existence（文件存在性）](#11-file_existence文件存在性)
  - [1.2 content_correctness（内容正确性）](#12-content_correctness内容正确性)
  - [1.3 functional_test（功能测试）](#13-functional_test功能测试)
  - [1.4 quality_standard（质量标准）](#14-quality_standard质量标准)
  - [1.5 compatibility_verification（兼容性验证）](#15-compatibility_verification兼容性验证)
- [2. 简单 MEU 的 Benchmark 模板](#2-简单-meu-的-benchmark-模板)
  - [2.1 配置/设计类 MEU](#21-配置设计类-meu)
  - [2.2 日志/输出类 MEU](#22-日志输出类-meu)
  - [2.3 UI/展示类 MEU](#23-ui展示类-meu)
  - [2.4 集成/组装类 MEU](#24-集成组装类-meu)
- [3. 禁止使用的模糊 Benchmark 描述完整列表](#3-禁止使用的模糊-benchmark-描述完整列表)
- [4. Evaluation 详解](#4-evaluation-详解)
  - [4.1 评估方法](#41-评估方法)
  - [4.2 评估步骤](#42-评估步骤)
  - [4.3 通过条件](#43-通过条件)
  - [4.4 失败处理](#44-失败处理)
- [5. 整体评估框架](#5-整体评估框架)
- [6. 完整的 Benchmark + Evaluation 示例](#6-完整的-benchmark--evaluation-示例)

---

## 1. Benchmark 类型详解

### 1.1 file_existence（文件存在性）

**用途**：验证预期的文件或目录是否被创建。

**适用场景**：
- 验证源代码文件是否创建
- 验证配置文件是否生成
- 验证目录结构是否正确
- 验证测试文件是否存在

**YAML 格式**：

```yaml
- id: "B1"
  type: "file_existence"
  description: "文件 src/filewatcher/cli.py 存在"
  check: "检查 src/filewatcher/cli.py 文件是否存在"
```

**设计要点**：
- 路径必须是相对于项目根目录的完整路径
- 如果需要验证目录，明确说明是目录而非文件
- 如果文件可能被放在多个位置之一，列出所有可能的位置

**更多示例**：

```yaml
# 验证目录存在
- id: "B1"
  type: "file_existence"
  description: "测试目录 tests/ 存在"
  check: "检查 tests/ 目录是否存在"

# 验证多个文件
- id: "B2"
  type: "file_existence"
  description: "所有核心模块文件存在"
  check: |
    检查以下文件全部存在：
    - src/filewatcher/cli.py
    - src/filewatcher/watcher.py
    - src/filewatcher/filter.py
    - src/filewatcher/output.py
    - src/filewatcher/models.py
```

### 1.2 content_correctness（内容正确性）

**用途**：验证文件内容是否包含预期的代码、配置或数据。

**适用场景**：
- 验证代码中是否定义了特定的函数/类/变量
- 验证配置文件中是否包含特定的配置项
- 验证代码是否使用了特定的技术方案
- 验证代码中是否避免了特定的反模式

**YAML 格式**：

```yaml
- id: "B2"
  type: "content_correctness"
  description: "cli.py 包含 ArgumentParser 且定义了所有必需参数"
  check: |
    检查 src/filewatcher/cli.py：
    1. 包含 argparse.ArgumentParser 的实例化
    2. 定义了 --path 参数（required=True）
    3. 定义了 --recursive 参数（default=True）
    4. 定义了 --filter 参数
    5. 定义了 --output-format 参数（choices=['json', 'text']）
    6. 定义了 --debounce 参数（type=int, default=500）
```

**设计要点**：
- 检查项应该具体到函数名、参数名、配置项名
- 如果需要验证"不使用"某个东西，也要明确说明（如"不使用 time.time()"）
- 可以使用 grep、AST 解析等工具辅助验证

### 1.3 functional_test（功能测试）

**用途**：通过运行代码或命令来验证运行时行为。

**适用场景**：
- 验证函数的输入输出行为
- 验证命令行工具的参数处理
- 验证 API 的请求响应
- 验证错误处理行为

**YAML 格式**：

```yaml
- id: "B3"
  type: "functional_test"
  description: "CLI 参数解析正确处理各种输入组合"
  check: |
    1. python -m filewatcher --path /tmp --output-format json -> 无报错退出
    2. python -m filewatcher --path /tmp --filter ".log,.txt" -> filter 值为 ['.log', '.txt']
    3. python -m filewatcher（不带 --path）-> 报错并提示必填参数
    4. python -m filewatcher --path /nonexistent -> 报错并提示路径不存在
```

**设计要点**：
- 测试用例应该覆盖正常路径和异常路径
- 每个测试用例应该有明确的输入和预期输出
- 如果测试需要特定的环境（如临时文件），说明环境准备步骤

### 1.4 quality_standard（质量标准）

**用途**：验证代码质量、测试覆盖率、性能等非功能性指标。

**适用场景**：
- 代码风格检查（linting）
- 代码格式检查（formatting）
- 测试覆盖率
- 性能基准
- 安全扫描

**YAML 格式**：

```yaml
- id: "B4"
  type: "quality_standard"
  description: "代码通过 linting 检查且满足覆盖率要求"
  check: |
    1. ruff check src/filewatcher/ -> 零错误
    2. pytest tests/ --cov=src/filewatcher -> 覆盖率 >= 80%
```

**设计要点**：
- 质量标准必须有明确的数值阈值
- 如果使用特定的 linting 工具，明确工具名称和版本
- 性能基准需要说明测试环境和数据规模

### 1.5 compatibility_verification（兼容性验证）

**用途**：验证多组件协作和端到端工作流。

**适用场景**：
- 验证模块间的集成是否正常
- 验证端到端的工作流程
- 验证跨平台兼容性
- 验证向后兼容性

**YAML 格式**：

```yaml
- id: "B5"
  type: "compatibility_verification"
  description: "主入口正确集成所有模块，CLI 启动和退出正常"
  check: |
    1. 运行 python -m filewatcher --path /tmp/test_dir
    2. 在 /tmp/test_dir 中创建/修改/删除文件
    3. 验证 stdout 输出对应的事件信息
    4. 按 Ctrl+C -> 优雅退出（无错误堆栈信息）
```

---

## 2. 简单 MEU 的 Benchmark 模板

对于常见的简单 MEU 类型，以下提供通用的 Benchmark 模板。使用时将 `{占位符}` 替换为实际内容。

### 2.1 配置/设计类 MEU

适用于：创建配置文件、设计数据结构、定义接口规范等任务。

**通用模板**：

```yaml
benchmarks:
  - id: "B1"
    type: "file_existence"
    description: "配置文件 {文件路径} 存在"
    check: "检查 {文件路径} 文件是否存在"

  - id: "B2"
    type: "content_correctness"
    description: "配置文件包含所有必需的字段且格式正确"
    check: |
      验证 {文件路径}：
      1. 包含字段 {字段1} = {值1}
      2. 包含字段 {字段2} = {值2}
      3. 文件语法有效（可通过 {验证命令} 验证）

  - id: "B3"
    type: "functional_test"
    description: "配置文件可以被正确加载和解析"
    check: |
      运行 {加载命令}，验证：
      1. 所有字段值与预期一致
      2. 缺少必需字段时报错
      3. 字段值类型错误时报错
```

**具体示例**——创建 pyproject.toml：

```yaml
benchmarks:
  - id: "B1"
    type: "file_existence"
    description: "pyproject.toml 文件存在"
    check: "检查 pyproject.toml 文件存在于项目根目录"

  - id: "B2"
    type: "content_correctness"
    description: "pyproject.toml 包含正确的项目配置"
    check: |
      验证 pyproject.toml：
      1. [project] 部分包含 name = "filewatcher"
      2. [project] 部分包含 requires-python = ">=3.11"
      3. [project.dependencies] 包含 watchdog
      4. [project.optional-dependencies] 的 dev 组包含 pytest 和 ruff
      5. [build-system] 使用 setuptools 作为构建后端

  - id: "B3"
    type: "functional_test"
    description: "项目可以被正确安装"
    check: |
      运行 pip install -e .，验证：
      1. 安装过程无报错
      2. python -c "import filewatcher" 可以正常导入
```

### 2.2 日志/输出类 MEU

适用于：实现日志输出、数据导出、报告生成等任务。

**通用模板**：

```yaml
benchmarks:
  - id: "B1"
    type: "file_existence"
    description: "输出模块 {文件路径} 存在"
    check: "检查 {文件路径} 文件是否存在"

  - id: "B2"
    type: "content_correctness"
    description: "模块实现了所有要求的输出格式"
    check: |
      验证 {文件路径}：
      1. 包含 {格式1} 输出函数，签名为 {签名1}
      2. 包含 {格式2} 输出函数，签名为 {签名2}
      3. 函数有正确的输入验证

  - id: "B3"
    type: "functional_test"
    description: "每种格式都能产生正确的输出"
    check: |
      使用测试数据验证：
      1. {格式1} 输出格式正确，包含所有必需字段
      2. {格式2} 输出格式正确，包含所有必需字段
      3. 空输入时的行为正确（不崩溃，输出空结果或提示）
      4. 特殊字符（引号、换行符、Unicode）处理正确
```

**具体示例**——实现日志输出模块：

```yaml
benchmarks:
  - id: "B1"
    type: "file_existence"
    description: "src/filewatcher/output.py 文件存在"
    check: "检查 src/filewatcher/output.py 文件是否存在"

  - id: "B2"
    type: "content_correctness"
    description: "output.py 实现了 text 和 json 两种输出格式"
    check: |
      验证 src/filewatcher/output.py：
      1. 包含 text 格式输出函数，接受 FileEvent 参数
      2. 包含 json 格式输出函数，接受 FileEvent 参数
      3. text 格式输出包含时间戳、事件类型、文件路径
      4. json 格式输出为合法的 JSON 对象，包含 timestamp, event_type, file_path 字段

  - id: "B3"
    type: "functional_test"
    description: "两种输出格式都能正确工作"
    check: |
      pytest tests/test_output.py -v
      1. text 格式：输入 FileEvent，验证输出包含 [CREATED] 等标记
      2. json 格式：输入 FileEvent，验证输出可被 json.loads() 解析
      3. 文件路径包含空格和中文时，两种格式都能正确处理
```

### 2.3 UI/展示类 MEU

适用于：实现前端组件、页面布局、数据可视化等任务。

**通用模板**：

```yaml
benchmarks:
  - id: "B1"
    type: "file_existence"
    description: "UI 组件 {文件路径} 存在"
    check: "检查 {文件路径} 文件是否存在"

  - id: "B2"
    type: "content_correctness"
    description: "组件实现了所有必需的 props 和交互"
    check: |
      验证 {文件路径}：
      1. 接受 {prop1}: {类型1} 属性
      2. 接受 {prop2}: {类型2} 属性
      3. 包含 {交互1} 事件处理
      4. 包含 {交互2} 事件处理

  - id: "B3"
    type: "functional_test"
    description: "组件渲染正确且交互正常"
    check: |
      使用测试工具验证：
      1. 正常 props 下组件正确渲染，DOM 结构符合预期
      2. 模拟 {交互1}，验证响应正确
      3. 模拟 {交互2}，验证响应正确
      4. 空 props / 边界值时组件有合理的降级处理
```

### 2.4 集成/组装类 MEU

适用于：将多个模块组装在一起、创建主入口、实现端到端流程等任务。

**通用模板**：

```yaml
benchmarks:
  - id: "B1"
    type: "file_existence"
    description: "集成文件 {文件路径} 存在"
    check: "检查 {文件路径} 文件是否存在"

  - id: "B2"
    type: "content_correctness"
    description: "集成代码正确导入并连接了各子模块"
    check: |
      验证 {文件路径}：
      1. 正确导入了 {模块1}、{模块2}、{模块3}
      2. 数据流逻辑正确：{模块1} 的输出传递给 {模块2}
      3. 包含错误处理和优雅退出逻辑
      4. 包含必要的信号处理（如 Ctrl+C）

  - id: "B3"
    type: "functional_test"
    description: "端到端流程正确工作"
    check: |
      完整流程测试：
      1. 启动程序 -> 无报错
      2. 执行核心操作 -> 输出正确
      3. 触发异常场景 -> 错误处理正确
      4. 正常退出 -> 清理资源，无残留
```

---

## 3. 禁止使用的模糊 Benchmark 描述完整列表

以下描述模式**禁止使用**。如果发现类似描述，必须改写为可量化的版本。

| 序号 | 禁止的描述 | 问题 | 改写为 |
|---|---|---|---|
| 1 | "代码质量很好" | 完全主观，无法量化 | "`ruff check` 零错误，`ruff format --check` 零差异" |
| 2 | "功能正常工作" | "正常"没有定义 | "输入 X 产生 Y，输入 Z 返回错误 E" |
| 3 | "性能可以接受" | 没有性能指标 | "处理 1000 条记录耗时 < 2 秒" |
| 4 | "UI 看起来不错" | 纯主观判断 | "在 320px/768px/1024px 宽度下无溢出" |
| 5 | "基本完成" | "基本"意味着可能没完成 | 列出具体的完成标准清单 |
| 6 | "遵循最佳实践" | 没有说明是哪个实践 | "遵循 {具体规范} 的第 {X} 条规则" |
| 7 | "错误处理完善" | 没有说明处理了哪些错误 | "场景 A 返回错误码 E1，场景 B 记录日志并继续" |
| 8 | "代码结构清晰" | 主观判断 | "每个函数不超过 30 行，模块间无循环依赖" |
| 9 | "文档完整" | 没有说明包含哪些内容 | "README 包含安装、使用、开发三个章节" |
| 10 | "测试充分" | 没有说明覆盖范围 | "单元测试覆盖率 >= 80%，覆盖所有公共 API" |
| 11 | "兼容性良好" | 没有说明兼容什么 | "在 Python 3.11 和 3.12 上测试通过" |
| 12 | "安全性足够" | 没有安全标准 | "通过 bandit 扫描零高危漏洞，无 SQL 注入风险" |

**自检方法**：对每个 Benchmark 描述问以下问题：
1. 两个不同的人独立评估，是否会给出相同的结果？
2. 是否存在任何歧义或多种理解方式？
3. 是否需要额外的上下文才能理解"通过"的标准？

如果任何一个问题的答案是"否"或"需要"，该描述就需要改写。

---

## 4. Evaluation 详解

### 4.1 评估方法

根据 Benchmark 类型选择合适的评估方法：

| Benchmark 类型 | 推荐评估方法 | 推荐工具 |
|---|---|---|
| `file_existence` | 自动化脚本 | `test -f`、`os.path.exists()`、`ls` |
| `content_correctness` | 自动化 + 人工审查 | `grep`、AST 解析、代码审查 |
| `functional_test` | 自动化测试 | `pytest`、`jest`、单元测试框架 |
| `quality_standard` | 自动化工具 | linter（ruff/eslint）、coverage 工具 |
| `compatibility_verification` | 自动化 + 手动测试 | E2E 测试框架、手动操作验证 |

**评估执行者**：
- 自动化 Benchmark：由评估脚本或 CI/CD 系统自动执行
- 需要人工审查的 Benchmark：由评估者按照 `check` 字段中的步骤手动执行

### 4.2 评估步骤

对每个 MEU 的评估按以下步骤执行：

1. **准备阶段**：
   - 确认所有依赖的 Benchmark 工具已安装
   - 确认测试环境已准备就绪
   - 阅读 MEU 的 Benchmark 列表

2. **执行阶段**：
   - 按 Benchmark ID 顺序逐个执行验证
   - 记录每个 Benchmark 的通过/不通过状态
   - 如果 Benchmark 不通过，记录具体的不通过原因

3. **汇总阶段**：
   - 统计通过和不通过的 Benchmark 数量
   - 如果存在不通过的 Benchmark，进行根因分析
   - 编写评估报告

### 4.3 通过条件

一个 MEU 的评估结果为"通过"，**必须同时满足以下所有条件**：

1. **所有 Benchmark 全部通过**：每个 Benchmark 的检查结果都是 `pass`
2. **无回归**：没有引入新的编译错误、运行时错误或功能退化
3. **交付物完整**：所有预期的文件、代码、文档都已产出
4. **与研究一致**：实现方案与研究推荐一致（如有偏差，必须有文档化的理由）

### 4.4 失败处理

当 MEU 评估不通过时，按以下流程处理：

**第一步：根因分析**

确定失败的根本原因：

| 根因类型 | 说明 | 处理方式 |
|---|---|---|
| 实现错误 | 代码逻辑有误，未正确实现功能 | 带着失败信息重新执行 MEU |
| Benchmark 设计缺陷 | Benchmark 本身有问题（条件不合理、描述模糊） | 修正 Benchmark，重新评估（不需要重新执行） |
| 研究结论有误 | 研究推荐的方案不正确或已过时 | 返回研究阶段，修正研究结果，然后重新执行 |
| 需求变更 | MEU 的需求发生了变化 | 返回拆解阶段，调整任务树，重新运行流程 |
| 依赖 MEU 有问题 | 上游依赖的 MEU 产出有问题 | 先修复依赖 MEU，然后重新执行当前 MEU |

**第二步：记录和跟踪**

- 在评估日志中记录失败详情
- 将 MEU 状态更新为 `failed`
- 创建修复任务，关联到原始 MEU

**第三步：修复和重新评估**

- 根据根因分析结果执行修复
- 修复完成后重新运行评估
- 重复此过程直到所有 Benchmark 通过

---

## 5. 整体评估框架

整体评估框架用于跟踪项目级别的评估进度和质量。

### 评估报告格式

```yaml
evaluation_report:
  project: "项目名称"
  evaluated_at: "2024-01-15T10:30:00Z"
  summary:
    total_meus: 10           # 总 MEU 数量
    passed: 8                # 通过数量
    failed: 1                # 失败数量
    skipped: 1               # 跳过数量
    pass_rate: "80%"         # 通过率
  meu_results:
    - task_id: "1.1.1"
      task_name: "创建项目结构"
      status: passed         # passed / failed / skipped
      benchmarks:
        - id: "B1"
          status: pass
        - id: "B2"
          status: pass
      evaluation_note: "所有 Benchmark 通过，代码质量良好"

    - task_id: "1.2.3"
      task_name: "实现事件过滤和去重"
      status: failed
      benchmarks:
        - id: "B1"
          status: pass
        - id: "B2"
          status: pass
        - id: "B3"
          status: pass
        - id: "B4"
          status: fail
          note: "debounce 在 100ms 以下时逻辑不正确"
      failure_analysis: |
        debounce 使用 time.time() 进行时间比较，但 time.time() 的精度
        在某些系统上不足以处理快速连续事件。应使用 time.monotonic()。
      next_action: "在执行 Prompt 中明确指定使用 time.monotonic()，重新执行"
```

### 评估流程图

```
MEU 执行完成
    │
    ▼
运行所有 Benchmark
    │
    ├── 全部通过 → 标记 MEU 为 completed → 进入下一个 MEU
    │
    └── 存在失败 → 根因分析
                      │
                      ├── 实现错误 → 重新执行 MEU
                      ├── Benchmark 缺陷 → 修正 Benchmark → 重新评估
                      ├── 研究结论有误 → 返回研究阶段
                      ├── 需求变更 → 返回拆解阶段
                      └── 依赖问题 → 先修复依赖 MEU
```

---

## 6. 完整的 Benchmark + Evaluation 示例

以下是一个完整的 MEU 的 Benchmark 设计和 Evaluation 计划示例：

```yaml
evaluation:
  task_id: "1.2.3"
  task_name: "实现事件过滤和去重"

  benchmarks:
    - id: "B1"
      type: "file_existence"
      description: "src/filewatcher/filter.py 文件存在"
      check: "检查 src/filewatcher/filter.py 文件是否存在"

    - id: "B2"
      type: "content_correctness"
      description: "filter.py 实现了扩展名过滤和 debounce 去重功能"
      check: |
        检查 src/filewatcher/filter.py：
        1. 包含 ExtensionFilter 类或 filter_by_extension() 函数
        2. 包含 Debouncer 类或 debounce() 函数
        3. ExtensionFilter 接受扩展名列表参数
        4. Debouncer 接受时间窗口参数（单位：毫秒）
        5. 使用 time.monotonic() 进行时间比较（不使用 time.time()）

    - id: "B3"
      type: "functional_test"
      description: "扩展名过滤功能正确"
      check: |
        pytest tests/test_filter.py::test_extension_filter -v
        1. 过滤 ".log" -> 保留 .log 文件，丢弃 .txt 文件
        2. 空扩展名列表 -> 保留所有文件
        3. 大小写不敏感匹配（.LOG 和 .log 都匹配）
        4. 多个扩展名过滤（".log,.txt" -> 只保留这两种）

    - id: "B4"
      type: "functional_test"
      description: "debounce 去重功能正确"
      check: |
        pytest tests/test_filter.py::test_debounce -v
        1. 同一文件在 500ms 内的多个事件合并为一个
        2. 同一文件超过 500ms 后的事件不合并
        3. 不同文件的事件不受 debounce 影响
        4. debounce 时间窗口参数可配置（如改为 200ms 也能正常工作）

    - id: "B5"
      type: "quality_standard"
      description: "代码通过 linting 检查"
      check: |
        1. ruff check src/filewatcher/filter.py -> 零错误
        2. ruff format --check src/filewatcher/filter.py -> 零差异

  evaluation_plan:
    method: "自动化测试"
    steps:
      - "确认 watchdog 依赖已安装（pip list | grep watchdog）"
      - "运行 ruff check 和 ruff format（验证 B5）"
      - "运行 pytest tests/test_filter.py（验证 B3、B4）"
      - "检查文件是否存在（验证 B1）"
      - "阅读源代码验证实现细节（验证 B2）"
    pass_criteria: "B1-B5 全部通过"
    failure_handling: |
      B1 失败：检查文件路径是否正确，是否遗漏了文件创建
      B2 失败：检查是否缺少某个功能实现，特别是 time.monotonic() 的使用
      B3 失败：检查过滤逻辑，特别是大小写处理和多扩展名场景
      B4 失败：检查 debounce 的时间比较逻辑，确认使用了 time.monotonic()
      B5 失败：运行 ruff format 自动修复格式问题，然后重新检查
```
