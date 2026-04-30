# 9 段式 Prompt 详细结构与 JSON 字段说明

本文档是 Step 4 执行计划生成的参考文档，包含 9 段式 Prompt 结构的详细填写要求、JSON 输出字段说明和完整的执行 Prompt 示例。

> **从属关系**：本文档由 [step4-plan.md](../steps/step4-plan.md) 按需加载。如需了解执行顺序生成方法、输出格式选择、研究价值传导机制等核心内容，请先阅读主文档。

---

## 目录

- [1. MEU 执行 Prompt 的 9 段式结构](#1-meu-执行-prompt-的-9-段式结构)
  - [1.1 第一段：项目背景](#11-第一段项目背景)
  - [1.2 第二段：任务描述](#12-第二段任务描述)
  - [1.3 第三段：上下文信息](#13-第三段上下文信息)
  - [1.4 第四段：避坑专区](#14-第四段避坑专区)
  - [1.5 第五段：决策日志](#15-第五段决策日志)
  - [1.6 第六段：分步执行指南](#16-第六段分步执行指南)
  - [1.7 第七段：验收标准](#17-第七段验收标准)
  - [1.8 第八段：约束条件](#18-第八段约束条件)
  - [1.9 第九段：输出要求](#19-第九段输出要求)
- [2. JSON 输出的完整字段说明](#2-json-输出的完整字段说明)
  - [2.1 顶层字段](#21-顶层字段)
  - [2.2 execution_order[] 字段](#22-execution_order-字段)
  - [2.3 meus[] 字段](#23-meus-字段)
  - [2.4 meus[].prompt 字段](#24-meusprompt-字段)
  - [2.5 meus[].prompt.acceptance_criteria 字段](#25-meuspromptacceptance_criteria-字段)
  - [2.6 risks[] 字段](#26-risks-字段)
  - [2.7 open_questions[] 字段](#27-open_questions-字段)
- [3. 完整执行 Prompt 示例](#3-完整执行-prompt-示例)

---

## 1. MEU 执行 Prompt 的 9 段式结构

每个 MEU 的执行 Prompt **必须包含且仅包含以下 9 个段落**。不允许省略任何一个段落，也不允许添加额外的段落。

> **设计理念**：传统的执行计划只有"任务清单 + 执行步骤"，执行 AI 无法获知研究阶段发现的坑点、方案优劣和边界条件。9 段式结构在原有 7 段式基础上新增"避坑专区"和"决策日志"，将 Plan 从单纯的"任务清单"升级为"避坑指南 + 执行蓝图"，确保研究阶段的知识无损传递给执行 AI。

### 1.1 第一段：项目背景

**目的**：将当前 MEU 放在整体项目的上下文中，让执行者理解"为什么要做这个任务"。

**必须包含的信息**：
- 项目名称和一句话描述
- 当前 MEU 所属的模块/子系统
- 当前执行批次编号（如"第 3 批次，共 5 批次"）
- 已完成的前置 MEU 及其关键产出摘要

**示例**：

```markdown
## 项目背景

- **项目名称**：文件监控 CLI 工具（filewatch）——监控指定目录的文件变化，输出结构化事件
- **所属模块**：核心事件处理模块
- **执行批次**：第 3 批次，共 5 批次
- **已完成的前置任务**：
  - MEU-1.1.1 "创建项目结构和配置文件" → 产出：标准 Python 项目结构、pyproject.toml
  - MEU-1.2.2 "实现文件系统监控模块" → 产出：watcher.py、FileEvent 数据类、事件队列
```

### 1.2 第二段：任务描述

**目的**：明确定义当前 MEU 必须完成什么。

**必须包含的信息**：
- 任务名称（与任务树一致）
- 一句话目标
- 范围：包含什么、不包含什么
- 预期产出

**示例**：

```markdown
## 任务描述

**名称**：实现事件过滤和去重模块

**目标**：在文件监听器之上构建事件过滤层，支持按文件扩展名过滤、路径排除和时间窗口去重。

**范围**：
- 包含：glob 模式过滤、路径排除、debounce 去重、过滤器链组合
- 不包含：事件聚合、跨进程同步、基于文件内容的变更检测

**预期产出**：可独立测试的过滤模块 `src/filewatcher/filter.py`
```

### 1.3 第三段：上下文信息

**目的**：提供执行所需的技术上下文和研究结论。

**必须包含的信息**：
- 技术栈和版本约束
- 研究阶段的关键发现（来自研究输出）
- 关键设计决策（来自评估阶段）
- 对现有文件/代码的引用

**示例**：

```markdown
## 上下文信息

**技术栈**：Python 3.11+，watchdog 库

**研究结论**（置信度：high）：
- 使用扩展名列表进行过滤，匹配时忽略大小写
- 使用 time.monotonic()（非 time.time()）进行时间比较
- debounce 使用滑动窗口算法，同一文件在时间窗口内的多个事件合并为一个
- 不同文件的事件不受 debounce 影响

**现有代码引用**：
- `src/filewatcher/models.py` 中的 FileEvent 数据类：event_type, file_path, timestamp
- `src/filewatcher/watcher.py` 中的事件队列：queue.Queue
```

### 1.4 第四段：避坑专区

**目的**：将研究阶段发现的坑点、常见错误和边界条件直接传递给执行 AI，避免重复踩坑。这是从"任务清单"到"避坑指南"的关键升级。

**必须包含的信息**：
- 必须避免的错误（带正确做法和来源）
- 需要注意的边界情况
- 推荐做法

**格式要求**：
- "必须避免的错误"使用 `❌ 错误 / ✅ 正确 / 📖 来源` 三行格式
- "边界情况"使用无序列表
- "推荐做法"使用无序列表
- 段落开头使用引用块（`>`）说明信息来源

**示例**：

```markdown
## 避坑专区

> 以下内容来自研究阶段的踩坑记录，执行时务必注意。

**必须避免的错误**：
- ❌ **错误**：直接使用 `time.time()` 进行时间比较
  ✅ **正确**：使用 `time.monotonic()`，因为 `time.time()` 会受系统时间调整影响
  📖 **来源**：研究阶段交叉验证（官方文档 + StackOverflow 高票回答）

- ❌ **错误**：在 EventHandler 中直接处理业务逻辑
  ✅ **正确**：将事件放入队列，由消费者线程处理。EventHandler 只做事件转换
  📖 **来源**：watchdog GitHub Issues #123

**需要注意的边界情况**：
- Vim/VSCode 等编辑器保存时会触发多个事件（临时文件创建 + 重命名），debounce 必须处理
- macOS FSEvents 会合并快速连续事件，Linux inotify 不会——debounce 策略需要兼容两者
- Windows 上文件锁定可能导致事件延迟或丢失，不建议在 Windows 上依赖实时性

**推荐做法**：
- 使用 `threading.Event` 实现优雅关闭，而非 `os._exit()`
- 队列设置最大长度（建议 10000），防止内存溢出
```

**填写指南**：

| 子段 | 数据来源 | 填写时机 |
|---|---|---|
| 必须避免的错误 | Research 阶段的 `blockers[].pitfalls`、`key_findings[category=caution]` | Plan 生成时从 Research 输出中提取 |
| 需要注意的边界情况 | Research 阶段的 `key_findings[category=caution]`、`open_questions[impact=risk]` | Plan 生成时从 Research 输出中提取 |
| 推荐做法 | Research 阶段的 `key_findings[category=approach]`、`recommended_approach` | Plan 生成时从 Research 输出中提取 |

**如果某个子段没有内容**：保留子段标题，在下方注明"研究阶段未发现相关坑点"，不要省略子段。

### 1.5 第五段：决策日志

**目的**：记录本 MEU 涉及的关键技术决策及其理由，帮助执行 AI 理解"为什么这么做"，避免执行者推翻已评估的方案。

**必须包含的信息**：
- 决策表格（决策、选择、放弃的方案、理由、Trade-off）
- 决策依据来源

**格式要求**：
- 使用 Markdown 表格
- 段落开头使用引用块（`>`）说明信息来源
- 决策依据来源使用无序列表

**示例**：

```markdown
## 决策日志

> 以下记录了本 MEU 涉及的关键技术决策及其理由，帮助理解"为什么这么做"。

| 决策 | 选择 | 放弃的方案 | 理由 | Trade-off |
|------|------|-----------|------|-----------|
| 文件监控库 | watchdog | pyinotify, 自研轮询 | 跨平台支持好，社区活跃 | 性能略低于 pyinotify（毫秒级差异） |
| 事件传递模式 | 队列模式 | 直接回调, asyncio | 线程安全，解耦彻底 | 引入少量延迟（毫秒级） |
| 去重算法 | 滑动窗口 | 固定窗口, 令牌桶 | 精确控制去重粒度 | 实现稍复杂 |

**决策依据来源**：
- watchdog vs pyinotify：官方文档对比 + GitHub Star 数 + StackOverflow 社区推荐
- 队列模式 vs 回调：研究阶段 pros/cons 分析（详见 research.yaml）
- 滑动窗口 vs 固定窗口：算法复杂度分析 + 实际场景模拟
```

**填写指南**：

| 子段 | 数据来源 | 填写时机 |
|---|---|---|
| 决策表格 | Research 阶段的 `decision_summary.key_decisions` | Plan 生成时从 Research 输出中提取 |
| 决策依据来源 | Research 阶段的 `references`、评估阶段的对比分析 | Plan 生成时从 Research 输出中提取 |

**如果本 MEU 没有关键技术决策**：保留段落标题，在下方注明"本 MEU 无关键技术决策，遵循项目统一技术选型"，不要省略段落。

### 1.6 第六段：分步执行指南

**目的**：提供明确的、有序的操作指令。

**必须包含的信息**：
- 编号的步骤列表，每个步骤以动词开头
- 每个步骤的输入和预期输出
- 关键步骤的注意事项
- 步骤之间的依赖关系

**示例**：

```markdown
## 分步执行指南

1. **创建** `src/filewatcher/filter.py` 文件
2. **实现** `ExtensionFilter` 类：
   - 输入：扩展名列表（如 [".log", ".txt"]）
   - 输出：过滤函数，接受 FileEvent，返回 bool（是否保留）
   - 注意：空列表表示保留所有事件；匹配时忽略大小写
3. **实现** `Debouncer` 类：
   - 输入：时间窗口（毫秒）
   - 输出：去重函数，接受 FileEvent，返回 bool（是否为非重复事件）
   - 注意：使用 time.monotonic() 进行时间比较
   - 注意：基于 file_path 去重，不同文件的事件互不影响
4. **实现** `FilterChain` 类：
   - 组合多个过滤器，按顺序应用
   - 输入：过滤器列表
   - 输出：组合后的过滤函数
5. **编写** 单元测试 `tests/test_filter.py`，覆盖所有公共 API
```

### 1.7 第七段：验收标准

**目的**：提供清晰、可验证的完成条件。

**必须包含的信息**：
- 功能性验收标准（必须全部通过）——使用 `- [ ]` 清单格式
- 质量标准（建议满足）
- 性能标准（如适用）

**示例**：

```markdown
## 验收标准

**功能性标准**（必须全部通过）：
- [ ] `src/filewatcher/filter.py` 文件存在
- [ ] 包含 `ExtensionFilter` 类，支持扩展名列表过滤
- [ ] 包含 `Debouncer` 类，支持时间窗口去重
- [ ] 扩展名匹配忽略大小写（.LOG 和 .log 都匹配）
- [ ] debounce 使用 time.monotonic() 进行时间比较
- [ ] `pytest tests/test_filter.py` 全部通过

**质量标准**（建议满足）：
- [ ] `ruff check src/filewatcher/filter.py` 零错误
- [ ] 测试覆盖率 >= 80%
```

### 1.8 第八段：约束条件

**目的**：定义执行的限制和规则。

**必须包含的信息**：
- 技术约束（语言版本、依赖限制）
- 架构约束（需要遵循的设计模式）
- 兼容性约束
- 安全约束（如适用）

**示例**：

```markdown
## 约束条件

- Python 版本 >= 3.11（使用了 match 语句等新特性）
- 只使用标准库和 watchdog，不引入其他第三方依赖
- 使用 time.monotonic() 而非 time.time()（研究阶段明确要求）
- 每个类/函数必须有 docstring
- 不修改 `src/filewatcher/models.py` 和 `src/filewatcher/watcher.py`
```

### 1.9 第九段：输出要求

**目的**：明确定义需要交付的具体产出物。

**必须包含的信息**：
- 代码文件及其路径
- 测试文件及其路径
- 文档更新（如有）
- 配置变更（如有）

**示例**：

```markdown
## 输出要求

**必须交付的文件**：
- `src/filewatcher/filter.py` —— 过滤和去重模块的源代码
- `tests/test_filter.py` —— 单元测试文件

**不需要交付**：
- 不需要修改 README.md
- 不需要修改 pyproject.toml
- 不需要修改其他已有文件
```

---

## 2. JSON 输出的完整字段说明

以下是 JSON 格式执行计划的完整字段说明。使用表格形式描述，不使用 JSON Schema。

### 2.1 顶层字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `project_name` | string | 是 | 项目名称 |
| `project_description` | string | 是 | 项目的一句话描述 |
| `version` | string | 是 | 版本号，格式 `MAJOR.MINOR.PATCH` |
| `generated_at` | string | 是 | 生成时间，ISO 8601 格式 |
| `total_meus` | integer | 是 | MEU 总数 |
| `total_batches` | integer | 是 | 执行批次总数 |
| `execution_order` | array | 是 | 分批执行顺序 |
| `meus` | array | 是 | 所有 MEU 的详细信息 |
| `risks` | array | 建议 | 风险列表 |
| `open_questions` | array | 建议 | 开放问题列表 |

### 2.2 execution_order[] 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `batch_number` | integer | 是 | 批次编号，从 1 开始 |
| `meu_ids` | string[] | 是 | 该批次中可并行执行的 MEU ID 列表 |
| `can_parallel` | boolean | 是 | 当前版本始终为 `true` |
| `estimated_duration` | string | 建议 | 预估耗时，如 `"2-4h"` |

### 2.3 meus[] 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | 是 | MEU ID，与任务树一致 |
| `name` | string | 是 | MEU 名称 |
| `description` | string | 是 | 任务描述 |
| `batch` | integer | 是 | 所属批次编号 |
| `dependencies` | string[] | 是 | 依赖的 MEU ID 列表 |
| `evaluation_score` | number | 是 | 评估得分，0-10 分 |
| `evaluation_summary` | string | 是 | 评估结论摘要 |
| `research_findings` | string | 是 | 研究结论摘要 |
| `prompt` | object | 是 | 9 段式执行 Prompt 的结构化数据 |
| `estimated_complexity` | string | 建议 | 预估复杂度：`low` / `medium` / `high` |

### 2.4 meus[].prompt 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `project_background` | string | 是 | 第一段：项目背景 |
| `task_description` | string | 是 | 第二段：任务描述 |
| `context` | string | 是 | 第三段：上下文信息 |
| `pitfall_guide` | string | 是 | 第四段：避坑专区 |
| `decision_log` | string | 是 | 第五段：决策日志 |
| `steps` | string[] | 是 | 第六段：分步执行指南 |
| `acceptance_criteria` | object | 是 | 第七段：验收标准 |
| `constraints` | string[] | 是 | 第八段：约束条件 |
| `output_requirements` | string[] | 是 | 第九段：输出要求 |

### 2.5 meus[].prompt.acceptance_criteria 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `functional` | string[] | 是 | 功能性验收标准（全部必须通过） |
| `quality` | string[] | 建议 | 质量标准（建议满足） |
| `performance` | string[] | 建议 | 性能标准（如适用） |

### 2.6 risks[] 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | 是 | 风险 ID，格式 `RISK-XXX` |
| `description` | string | 是 | 风险描述 |
| `severity` | string | 是 | 严重度：`low` / `medium` / `high` / `critical` |
| `affected_meus` | string[] | 是 | 受影响的 MEU ID 列表 |
| `mitigation` | string | 建议 | 缓解方案 |
| `status` | string | 建议 | 状态：`open` / `mitigated` / `accepted` |

### 2.7 open_questions[] 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | 是 | 问题 ID，格式 `OQ-XXX` |
| `question` | string | 是 | 问题描述 |
| `related_meus` | string[] | 是 | 相关的 MEU ID 列表 |
| `suggested_answer` | string | 建议 | 建议的答案或处理方向 |
| `blocking` | boolean | 是 | 是否阻塞执行 |

---

## 3. 完整执行 Prompt 示例

以下是一个完整的 MEU 执行 Prompt 示例（Prompt 格式），展示 9 段式结构的完整内容：

```markdown
## 项目背景

- **项目名称**：文件监控 CLI 工具（filewatch）——监控指定目录的文件变化，输出结构化事件
- **所属模块**：核心事件处理模块
- **执行批次**：第 3 批次，共 6 批次
- **已完成的前置任务**：
  - MEU-1.1.1 "创建项目结构和配置文件" → 产出：标准 Python 项目结构、pyproject.toml、watchdog 依赖已声明
  - MEU-1.2.2 "实现文件系统监控模块" → 产出：watcher.py（Observer + EventHandler 模式）、FileEvent 数据类（event_type, file_path, timestamp）、事件队列（queue.Queue）

## 任务描述

**名称**：实现事件过滤和去重模块

**目标**：在文件监听器之上构建事件过滤层，支持按文件扩展名过滤和时间窗口去重。

**范围**：
- 包含：扩展名列表过滤（大小写不敏感）、时间窗口 debounce 去重、过滤器链组合
- 不包含：事件聚合统计、跨进程同步、基于文件内容的变更检测、路径排除（后续版本）

**预期产出**：可独立测试的过滤模块 `src/filewatcher/filter.py`，包含完整的单元测试 `tests/test_filter.py`

## 上下文信息

**技术栈**：Python 3.11+，watchdog 库（已安装）

**研究结论**（置信度：high）：
- 扩展名过滤：接受扩展名列表参数，空列表表示保留所有事件，匹配时忽略大小写
- 时间窗口去重：使用 time.monotonic()（不是 time.time()）进行时间比较
- debounce 算法：基于 file_path 去重，同一文件在时间窗口内的多个事件合并为一个，不同文件的事件互不影响
- 推荐使用过滤器链模式：将过滤器和去重器组合为链式处理器

**现有代码引用**：
- `src/filewatcher/models.py`：FileEvent 数据类，字段为 event_type（str）、file_path（str）、timestamp（float）
- `src/filewatcher/watcher.py`：事件队列 queue.Queue，从中获取原始 FileEvent

## 避坑专区

> 以下内容来自研究阶段的踩坑记录，执行时务必注意。

**必须避免的错误**：
- ❌ **错误**：直接使用 `time.time()` 进行时间比较
  ✅ **正确**：使用 `time.monotonic()`，因为 `time.time()` 会受系统时间调整影响（NTP 同步、手动修改系统时间等）
  📖 **来源**：研究阶段交叉验证（Python 官方文档 time.monotonic() 说明 + StackOverflow 高票回答）

- ❌ **错误**：在 EventHandler 中直接处理业务逻辑（过滤、去重等）
  ✅ **正确**：将事件放入队列，由消费者线程处理。EventHandler 只做事件类型转换（watchdog 事件 → FileEvent）
  📖 **来源**：watchdog GitHub Issues #123、watchdog 官方文档推荐模式

- ❌ **错误**：使用 `dict` 存储去重时间戳且不清理过期条目
  ✅ **正确**：使用滑动窗口去重，每次检查时清理超出时间窗口的旧条目，防止内存泄漏
  📖 **来源**：研究阶段性能分析（长时间运行场景模拟）

**需要注意的边界情况**：
- Vim/VSCode 等编辑器保存时会触发多个事件（临时文件 .swp 创建 + 重命名覆盖原文件），debounce 必须正确处理这种模式
- macOS FSEvents 会合并快速连续事件（可能只报告最终状态），Linux inotify 不会——debounce 策略需要兼容两者
- Windows 上文件锁定可能导致事件延迟或丢失，不建议在 Windows 上依赖实时性
- 符号链接（symlink）可能触发重复事件（源文件和链接目标各触发一次），需要考虑是否去重
- 网络文件系统（NFS、SMB）的事件通知不可靠，当前版本不支持

**推荐做法**：
- 使用 `threading.Event` 实现优雅关闭，而非 `os._exit()` 或 `sys.exit()`
- 队列设置最大长度（建议 10000），防止内存溢出；超出时记录告警并丢弃最旧的事件
- Debouncer 的内部字典定期清理（每次调用时检查即可，无需额外线程）
- 过滤器链使用函数组合模式，每个过滤器是独立的纯函数，便于测试

## 决策日志

> 以下记录了本 MEU 涉及的关键技术决策及其理由，帮助理解"为什么这么做"。

| 决策 | 选择 | 放弃的方案 | 理由 | Trade-off |
|------|------|-----------|------|-----------|
| 去重算法 | 滑动窗口（基于 file_path） | 固定窗口, 令牌桶, 布隆过滤器 | 精确控制去重粒度，实现简单，内存占用可控 | 实现稍复杂于固定窗口；极端场景下内存占用高于令牌桶 |
| 过滤器组合模式 | 函数组合（FilterChain） | 责任链模式, 装饰器模式 | 代码简洁，易于测试，符合函数式编程风格 | 不支持动态增删过滤器（但当前场景不需要） |
| 过滤器接口 | 独立函数 + 类两种形式 | 仅函数, 仅类 | 函数适合简单过滤（如扩展名过滤），类适合有状态过滤（如 Debouncer） | 接口不统一，但更符合 Python 惯用法 |
| 时间比较方式 | time.monotonic() | time.time(), time.perf_counter() | 不受系统时间调整影响，适合测量时间间隔 | 无法转换为 wall clock 时间（但本场景不需要） |

**决策依据来源**：
- 滑动窗口 vs 固定窗口 vs 令牌桶：研究阶段算法复杂度分析 + 实际场景模拟（详见 research.yaml 中的 debounce 算法对比）
- 函数组合 vs 责任链：研究阶段 pros/cons 分析，函数组合在 Python 中更惯用
- time.monotonic() vs time.time()：Python 官方文档推荐 + StackOverflow 高票回答交叉验证

## 分步执行指南

1. **创建** `src/filewatcher/filter.py` 文件，添加模块 docstring
2. **实现** `filter_by_extension(events, extensions)` 函数：
   - 输入：事件迭代器、扩展名列表（如 [".log", ".txt"]）
   - 输出：过滤后的事件生成器
   - 逻辑：空列表 → 保留所有；非空列表 → 只保留匹配扩展名的事件（大小写不敏感）
3. **实现** `Debouncer` 类：
   - `__init__(self, window_ms: int = 500)`：初始化，接受时间窗口参数
   - `__call__(self, event: FileEvent) -> bool`：判断事件是否为非重复事件
   - 内部维护 `dict[str, float]` 记录每个文件路径的最后处理时间
   - **关键**：使用 `time.monotonic()` 而非 `time.time()`
   - **关键**：每次调用时清理超出时间窗口的旧条目，防止内存泄漏
4. **实现** `debounce_events(events, window_ms)` 函数：
   - 输入：事件迭代器、时间窗口（毫秒）
   - 输出：去重后的事件生成器
   - 内部使用 Debouncer 类
5. **实现** `FilterChain` 类：
   - `__init__(self, *filters)`：接受多个过滤函数
   - `__call__(self, event: FileEvent) -> bool`：按顺序应用所有过滤器，全部通过才保留
6. **创建** `tests/test_filter.py`，编写以下测试：
   - `test_extension_filter_basic`：基本过滤功能
   - `test_extension_filter_empty_list`：空列表保留所有
   - `test_extension_filter_case_insensitive`：大小写不敏感
   - `test_debounce_same_file`：同文件去重
   - `test_debounce_different_file`：不同文件不去重
   - `test_debounce_configurable_window`：可配置时间窗口
   - `test_debounce_memory_cleanup`：长时间运行后内存不泄漏
   - `test_filter_chain`：过滤器链组合

## 验收标准

**功能性标准**（必须全部通过）：
- [ ] `src/filewatcher/filter.py` 文件存在
- [ ] 包含 `filter_by_extension()` 函数，接受扩展名列表参数
- [ ] 包含 `Debouncer` 类，接受时间窗口参数
- [ ] 包含 `debounce_events()` 函数
- [ ] 包含 `FilterChain` 类，支持组合多个过滤器
- [ ] 扩展名匹配忽略大小写
- [ ] debounce 使用 `time.monotonic()` 进行时间比较
- [ ] Debouncer 在长时间运行后不会内存泄漏（旧条目被清理）
- [ ] `pytest tests/test_filter.py -v` 全部通过

**质量标准**（建议满足）：
- [ ] `ruff check src/filewatcher/filter.py` 零错误
- [ ] `ruff format --check src/filewatcher/filter.py` 零差异
- [ ] 测试覆盖率 >= 80%

## 约束条件

- Python 版本 >= 3.11
- 只使用标准库（time, collections），不引入新的第三方依赖
- **必须**使用 `time.monotonic()` 而非 `time.time()`
- 每个公开函数/类必须有 docstring
- 不修改 `src/filewatcher/models.py` 和 `src/filewatcher/watcher.py`
- 不修改 `src/filewatcher/cli.py` 和 `src/filewatcher/output.py`

## 输出要求

**必须交付**：
- `src/filewatcher/filter.py` —— 过滤和去重模块源代码
- `tests/test_filter.py` —— 单元测试文件

**不需要交付**：
- 不需要修改 README.md
- 不需要修改 pyproject.toml
- 不需要修改任何已有文件
```
