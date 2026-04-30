# 研究输出格式详解

> 本文件是 Step 2（技术研究）的参考文件。包含输出字段的详细填写要求和完整示例。
> 从 [step2-research.md](../steps/step2-research.md) 加载核心指南后，需要了解输出字段的详细填写要求时按需读取。

---

## 目录

- [task_id](#task_id)
- [decision_summary](#decision_summary)
- [blockers](#blockers)
- [key_findings](#key_findings)
- [recommended_approach](#recommended_approach)
- [confidence](#confidence)
- [open_questions](#open_questions)
- [references](#references)
- [无堵点 MEU 的处理方式](#无堵点-meu-的处理方式)
- [完整研究示例](#完整研究示例)
  - [有堵点的示例（多轮迭代）](#有堵点的示例多轮迭代)
  - [无堵点的示例](#无堵点的示例)

---

每个 MEU 的研究结果必须遵循以下 YAML 结构。以下逐一说明每个字段的含义、类型和填写要求。

## task_id

- **类型**：string
- **必填**：是
- **说明**：对应任务树中 MEU 的节点 ID，如 `"1.2.3"`

## decision_summary

- **类型**：object
- **必填**：是（有堵点的 MEU 必填；无堵点的 MEU 可以为空对象 `{}`）
- **说明**：研究决策摘要，用于传递给 Plan 阶段。包含关键决策、踩坑概览和总体置信度。
- **子字段**：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `key_decisions` | array of objects | 关键技术决策列表 |
| `pitfalls_overview` | array of strings | 踩坑概览，简洁描述研究中发现的常见陷阱 |
| `confidence` | string | 总体置信度：`high` / `medium` / `low` |

- **key_decisions 子字段**：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `decision` | string | 决策内容 |
| `alternatives` | array of strings | 被否决的备选方案 |
| `reasoning` | string | 选择理由 |
| `trade_offs` | string | 选择此方案付出的代价 |
| `confidence` | string | 该决策的置信度 |

**填写示例**：

```yaml
decision_summary:
  key_decisions:
    - decision: "使用 watchdog 而非 pyinotify"
      alternatives: ["pyinotify（仅 Linux）", "自研轮询方案"]
      reasoning: "watchdog 跨平台支持好，社区活跃，API 简洁"
      trade_offs: "性能略低于 pyinotify，但可接受"
      confidence: "high"
    - decision: "使用队列模式解耦事件生产和消费"
      alternatives: ["直接回调", "asyncio 事件循环"]
      reasoning: "队列模式线程安全，解耦彻底，便于添加过滤逻辑"
      trade_offs: "引入少量延迟（毫秒级），不影响使用场景"
      confidence: "high"
  pitfalls_overview:
    - "Observer.start() 创建守护线程，必须调用 stop()"
    - "Vim 保存会触发多个事件，需要 debounce"
    - "Windows 文件锁定可能导致事件延迟"
  confidence: "high"
```

## blockers

- **类型**：array of objects
- **必填**：是（可以为空数组）
- **说明**：该 MEU 面临的堵点列表。每个堵点包含以下子字段：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `type` | string | 堵点类型，必须是以下 6 种之一：`技术堵点`、`知识堵点`、`工具堵点`、`数据堵点`、`集成堵点`、`性能堵点` |
| `description` | string | 堵点的详细描述，包括：遇到了什么问题、为什么这个问题阻碍了执行、尝试过什么方案 |
| `research_rounds` | array of objects | 研究过程记录，记录每轮研究的行动和发现 |
| `solutions` | array of objects | 候选方案列表，每个方案包含 pros/cons 分析 |
| `chosen_solution` | string | 最终选择的方案名称 |
| `reasoning` | string | 选择理由，包含 trade-off 分析 |
| `pitfalls` | array of objects | 踩坑记录 |
| `status` | string | 堵点状态：`resolved`（已解决）/ `partial`（部分解决）/ `unresolved`（未解决） |

**research_rounds 子字段**：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `round` | integer | 研究轮次编号（2 或 3） |
| `action` | string | 本轮执行的研究行动 |
| `sources` | array of objects | 信息来源列表（第 2 轮填写） |
| `finding` | string | 本轮的研究发现（第 3 轮填写） |
| `confidence_change` | string | 置信度变化，如 `"medium → high"`（第 3 轮填写） |

**sources 子字段**：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `type` | string | 信息源类型：`官方文档` / `GitHub` / `技术博客` / `StackOverflow` / `其他` |
| `url` | string | 信息来源 URL |
| `key_takeaway` | string | 从该来源获得的关键信息 |

**solutions 子字段**：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `name` | string | 方案名称 |
| `description` | string | 方案描述 |
| `pros` | array of strings | 优点列表（至少 2 个） |
| `cons` | array of strings | 缺点列表（至少 1 个） |
| `feasibility` | string | 可行性：`high` / `medium` / `low` |
| `code_snippet` | string | 最小可运行代码片段（可选） |

**pitfalls 子字段**：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `trap` | string | 常见错误描述 |
| `correct_approach` | string | 正确做法 |
| `source` | string | 信息来源 |

**填写示例**：

```yaml
blockers:
  - type: "技术堵点"
    description: |
      不确定 watchdog Observer 和 EventHandler 的使用方式，
      特别是递归监控的配置方法和事件类型的详细行为。
      需要确认：如何创建 EventHandler、如何注册到 Observer、
      recursive 参数的具体行为、支持哪些事件类型。
    research_rounds:
      - round: 2
        action: "搜索 watchdog Observer 官方文档和 GitHub 示例"
        sources:
          - type: "官方文档"
            url: "https://python-watchdog.readthedocs.io/en/stable/api.html"
            key_takeaway: "Observer 使用 schedule() 方法注册 EventHandler"
          - type: "GitHub"
            url: "https://github.com/gorakhargosh/watchdog"
            key_takeaway: "示例代码展示了完整的 Observer + EventHandler 用法"
      - round: 3
        action: "交叉验证 API 用法和事件类型"
        finding: "官方文档和 GitHub 示例一致确认 API 用法，事件类型完整"
        confidence_change: "medium → high"
    solutions:
      - name: "watchdog Observer + EventHandler"
        description: "使用 watchdog 的标准 Observer + EventHandler 模式"
        pros:
          - "跨平台支持（macOS/Linux/Windows）"
          - "API 设计简洁，学习曲线低"
          - "社区活跃，长期维护"
        cons:
          - "性能略低于平台原生方案"
        feasibility: "high"
        code_snippet: |
          from watchdog.observers import Observer
          from watchdog.events import FileSystemEventHandler

          class MyHandler(FileSystemEventHandler):
              def on_modified(self, event):
                  print(f"Modified: {event.src_path}")

          observer = Observer()
          observer.schedule(MyHandler(), path=".", recursive=True)
          observer.start()
    chosen_solution: "watchdog Observer + EventHandler"
    reasoning: |
      watchdog 是 Python 文件监控的事实标准，跨平台支持完善。
      相比 pyinotify（仅 Linux）和自研轮询方案（实时性差），
      watchdog 在功能、可维护性和社区支持方面综合最优。
      性能劣势在当前场景下可忽略。
    pitfalls:
      - trap: "Observer.start() 创建守护线程，程序退出时不自动清理"
        correct_approach: "使用 try/finally 确保 observer.stop() 被调用"
        source: "watchdog 官方文档"
      - trap: "Vim 保存会触发多个事件（临时文件 + 重命名）"
        correct_approach: "实现 debounce 去重机制"
        source: "技术博客 + GitHub Issues"
    status: resolved
```

## key_findings

- **类型**：array of objects
- **必填**：是（**即使没有堵点，此字段也必须有内容**）
- **说明**：研究过程中的关键发现。每个发现包含以下子字段：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `category` | string | 发现类别，必须是以下 4 种之一：`approach`（方案）、`api`（接口）、`pattern`（设计模式）、`caution`（注意事项/陷阱） |
| `content` | string | 发现的详细内容 |

**类别说明**：

| 类别 | 用途 | 填写示例 |
|---|---|---|
| `approach` | 推荐的实现方案、技术选型结论 | "使用 watchdog Observer 模式，将原生事件转换为统一的 FileEvent 数据类" |
| `api` | 具体的 API 用法、配置方法 | "核心类：watchdog.observers.Observer、watchdog.events.FileSystemEventHandler" |
| `pattern` | 推荐的设计模式、架构模式 | "使用队列模式解耦事件生产和消费：EventHandler 将事件放入 queue.Queue" |
| `caution` | 需要注意的陷阱、常见错误、边界情况 | "Observer.start() 会创建守护线程，退出时必须调用 observer.stop()" |

## recommended_approach

- **类型**：string
- **必填**：是
- **说明**：一段话总结推荐的技术方案。这是执行者最需要参考的信息，应该清晰、具体、可操作。
- **填写要求**：
  - 说明选择了什么技术/方案
  - 说明为什么选择这个方案（简要理由）
  - 说明关键的实现要点

**填写示例**：

```yaml
recommended_approach: |
  使用 watchdog 的 Observer + EventHandler 模式实现文件监控。
  选择 watchdog 是因为它是 Python 文件监控的事实标准，跨平台支持好。
  关键实现要点：将原生事件转换为统一的 FileEvent 数据类，
  使用队列模式解耦事件生产和消费，通过 schedule() 的 recursive 参数控制递归监控。
```

## confidence

- **类型**：string
- **必填**：是
- **可选值**：`high` / `medium` / `low`
- **说明**：对整个研究结论的总体置信度。详见 [step2-validation.md](step2-validation.md)。

## open_questions

- **类型**：array of objects
- **必填**：是（可以为空数组）
- **说明**：研究中**无法完全解决**的问题。这些问题需要在后续阶段处理。每个问题包含以下子字段：

| 子字段 | 类型 | 说明 |
|---|---|---|
| `question` | string | 问题的清晰描述 |
| `impact` | string | 影响级别：`blocking`（阻塞执行）/ `risk`（有风险但不阻塞）/ `minor`（影响较小） |
| `suggested_resolution` | string | 建议的解决方案（即使只是"执行时根据实际情况决定"） |

## references

- **类型**：array of strings
- **必填**：是（可以为空数组）
- **说明**：研究中参考的资料链接或文献引用。
- **填写要求**：
  - 优先提供官方文档链接
  - 包含关键的 Stack Overflow 回答链接（附答案 ID）
  - 包含参考的 GitHub 仓库链接

---

## 无堵点 MEU 的处理方式

**关键原则**：即使 `blockers` 为空数组，`key_findings` 字段**必须包含实质内容**。无堵点不代表无研究价值——研究阶段还承担着"为执行者提供充分上下文"的职责。

对于无堵点的 MEU，`key_findings` 应该包含以下类型的内容：

1. **实现要点**：关键步骤或代码结构建议
2. **技术细节**：具体的 API 用法、配置方法
3. **注意事项**：常见陷阱、边界情况
4. **最佳实践**：推荐的代码组织方式

**无堵点 MEU 的研究输出示例**：

```yaml
- task_id: "1.1.1"
  task_name: "创建项目结构和配置文件"
  decision_summary: {}
  blockers: []
  key_findings:
    - category: approach
      content: |
        使用 Python 3.11+ 标准项目结构：
        - src/filewatcher/ 用于存放源代码
        - tests/ 用于存放测试代码
        - pyproject.toml 用于统一的项目配置和依赖管理
        推荐使用 setuptools 作为构建后端，因为它是 Python 标准库的一部分。
    - category: api
      content: |
        pyproject.toml 关键配置：
        [project] 部分声明 name, version, requires-python = ">=3.11"
        [project.dependencies] 声明 watchdog
        [project.optional-dependencies] 的 dev 组声明 pytest, ruff
    - category: caution
      content: |
        1. 必须在 pyproject.toml 中声明 Python >= 3.11，
           因为后续代码会使用 match 语句等 3.11+ 特性
        2. src 布局需要在 pyproject.toml 中配置 [tool.setuptools.packages.find]
        3. .gitignore 需要包含 __pycache__/, *.pyc, .pytest_cache/, dist/, build/
  recommended_approach: |
    创建标准 Python 项目结构，使用 pyproject.toml 统一管理配置和依赖。
    声明 watchdog 为核心依赖，pytest 和 ruff 为开发依赖。
  confidence: high
  open_questions: []
  references:
    - "https://packaging.python.org/en/latest/tutorials/packaging-projects/"
```

---

## 完整研究示例

### 有堵点的示例（多轮迭代）

以下是一个存在多个堵点的 MEU 的完整研究输出，展示了多轮迭代研究流程和 `decision_summary` 结构：

```yaml
research_results:
  - task_id: "1.2.2"
    task_name: "实现文件系统监控模块"

    decision_summary:
      key_decisions:
        - decision: "使用 watchdog 而非 pyinotify"
          alternatives: ["pyinotify（仅 Linux）", "自研轮询方案"]
          reasoning: "watchdog 跨平台支持好，社区活跃，API 简洁"
          trade_offs: "性能略低于 pyinotify，但可接受"
          confidence: "high"
        - decision: "使用队列模式解耦事件生产和消费"
          alternatives: ["直接回调", "asyncio 事件循环"]
          reasoning: "队列模式线程安全，解耦彻底，便于添加过滤逻辑"
          trade_offs: "引入少量延迟（毫秒级），不影响使用场景"
          confidence: "high"
      pitfalls_overview:
        - "Observer.start() 创建守护线程，必须调用 stop()"
        - "Vim 保存会触发多个事件，需要 debounce"
        - "Windows 文件锁定可能导致事件延迟"
      confidence: "high"

    blockers:
      - type: "技术堵点"
        description: |
          不确定 watchdog Observer 和 EventHandler 的使用方式，
          特别是递归监控的配置方法和事件类型的详细行为。
          需要确认：如何创建 EventHandler、如何注册到 Observer、
          recursive 参数的具体行为、支持哪些事件类型。
        research_rounds:
          - round: 2
            action: "搜索 watchdog Observer 官方文档和 GitHub 示例"
            sources:
              - type: "官方文档"
                url: "https://python-watchdog.readthedocs.io/en/stable/api.html"
                key_takeaway: "Observer 使用 schedule(event_handler, path, recursive=False) 注册监控"
              - type: "GitHub"
                url: "https://github.com/gorakhargosh/watchdog"
                key_takeaway: "示例代码展示了完整的 Observer + EventHandler 用法"
          - round: 3
            action: "交叉验证 API 用法和事件类型"
            finding: "官方文档和 GitHub 示例一致确认 API 用法，事件类型完整覆盖创建/修改/删除"
            confidence_change: "medium → high"
        solutions:
          - name: "watchdog Observer + EventHandler"
            description: "使用 watchdog 的标准 Observer + EventHandler 模式"
            pros:
              - "跨平台支持（macOS/Linux/Windows）"
              - "API 设计简洁，学习曲线低"
              - "社区活跃，长期维护"
            cons:
              - "性能略低于平台原生方案"
            feasibility: "high"
            code_snippet: |
              from watchdog.observers import Observer
              from watchdog.events import FileSystemEventHandler

              class MyHandler(FileSystemEventHandler):
                  def on_created(self, event):
                      print(f"Created: {event.src_path}")
                  def on_modified(self, event):
                      print(f"Modified: {event.src_path}")
                  def on_deleted(self, event):
                      print(f"Deleted: {event.src_path}")

              observer = Observer()
              observer.schedule(MyHandler(), path=".", recursive=True)
              observer.start()
              try:
                  while True:
                      pass
              except KeyboardInterrupt:
                  observer.stop()
                  observer.join()
          - name: "pyinotify"
            description: "使用 pyinotify 库，基于 Linux inotify 接口"
            pros:
              - "Linux 平台上性能最优"
              - "底层控制力强"
            cons:
              - "仅支持 Linux，无法跨平台"
              - "项目已停止维护（最后更新 2015 年）"
            feasibility: "low"
            code_snippet: |
              import pyinotify
              class EventHandler(pyinotify.ProcessEvent):
                  def process_IN_MODIFY(self, event):
                      print(f"Modified: {event.pathname}")
              wm = pyinotify.WatchManager()
              handler = EventHandler()
              notifier = pyinotify.Notifier(wm, handler)
              wm.add_watch("/path/to/watch", pyinotify.IN_MODIFY, rec=True)
              notifier.loop()
        chosen_solution: "watchdog Observer + EventHandler"
        reasoning: |
          watchdog 是 Python 文件监控的事实标准，跨平台支持完善。
          pyinotify 仅限 Linux 且已停止维护，自研方案实时性差。
          watchdog 在功能、可维护性和社区支持方面综合最优。
        pitfalls:
          - trap: "Observer.start() 创建守护线程，程序退出时不自动清理"
            correct_approach: "使用 try/finally 或 context manager 确保 observer.stop() 被调用"
            source: "watchdog 官方文档 + Stack Overflow"
          - trap: "Vim 保存会触发多个事件（临时文件创建 + 重命名）"
            correct_approach: "实现 debounce 去重机制，合并短时间内的多个事件"
            source: "技术博客 + GitHub Issues"
        status: resolved

      - type: "集成堵点"
        description: |
          watchdog 在不同操作系统上使用不同的后端：
          macOS 使用 FSEvents，Linux 使用 inotify，Windows 使用 ReadDirectoryChangesW。
          需要确认不同后端的事件行为差异，特别是事件合并和时序特性。
        research_rounds:
          - round: 2
            action: "搜索 watchdog 跨平台事件行为差异"
            sources:
              - type: "官方文档"
                url: "https://python-watchdog.readthedocs.io/en/stable/..."
                key_takeaway: "macOS 使用 FSEvents，Linux 使用 inotify，Windows 使用 ReadDirectoryChangesW"
              - type: "技术博客"
                url: "https://martin.ankerl.com/..."
                key_takeaway: "FSEvents 有事件合并行为（coalescing），inotify 不会合并"
          - round: 3
            action: "交叉验证 macOS 和 Linux 事件差异"
            finding: "两个来源确认 macOS 有事件合并行为，Linux 不会合并。Windows 有文件锁定延迟"
            confidence_change: "low → high"
        solutions:
          - name: "应用层 debounce 去重"
            description: "在应用层实现 debounce 机制，消除平台差异"
            pros:
              - "统一处理所有平台的事件差异"
              - "实现简单，可配置去重时间窗口"
              - "对上层代码透明"
            cons:
              - "引入少量延迟（取决于 debounce 时间窗口）"
            feasibility: "high"
            code_snippet: |
              import threading, time

              class Debouncer:
                  def __init__(self, delay_ms=500):
                      self._delay = delay_ms / 1000.0
                      self._timer = None
                      self._lock = threading.Lock()

                  def debounce(self, callback):
                      with self._lock:
                          if self._timer:
                              self._timer.cancel()
                          self._timer = threading.Timer(self._delay, callback)
                          self._timer.start()
          - name: "平台特定处理"
            description: "针对不同平台编写特定的事件处理逻辑"
            pros:
              - "可以充分利用平台特性"
            cons:
              - "代码复杂度高，维护成本大"
              - "需要为每个平台单独测试"
            feasibility: "medium"
        chosen_solution: "应用层 debounce 去重"
        reasoning: |
          应用层 debounce 方案实现简单，对上层代码透明，
          统一消除了所有平台的事件差异。平台特定方案维护成本过高，
          在当前场景下不划算。debounce 引入的延迟（500ms）对文件监控场景可接受。
        pitfalls:
          - trap: "Windows 文件锁定可能导致事件延迟或丢失"
            correct_approach: "在 Windows 上增加事件超时重试机制"
            source: "watchdog GitHub Issues"
        status: resolved

    key_findings:
      - category: approach
        content: |
          推荐使用 watchdog Observer + EventHandler 模式：
          1. 定义 FileEventHandler 类，继承 FileSystemEventHandler
          2. 在事件处理方法中将 watchdog 原生事件转换为自定义的 FileEvent 数据类
          3. 使用 threading.Event 实现优雅关闭机制
          4. 使用 queue.Queue 解耦事件的生产和消费

      - category: api
        content: |
          watchdog 核心类和接口：
          - watchdog.observers.Observer: 监控调度器，管理监控线程的生命周期
          - watchdog.events.FileSystemEventHandler: 事件处理器基类
          - watchdog.events.FileCreatedEvent: 文件创建事件
          - watchdog.events.FileModifiedEvent: 文件修改事件
          - watchdog.events.FileDeletedEvent: 文件删除事件
          关键方法：
          - Observer.schedule(event_handler, path, recursive=False)
          - Observer.start() / Observer.stop()

      - category: pattern
        content: |
          推荐使用生产者-消费者模式：
          - EventHandler（生产者）将转换后的 FileEvent 放入 queue.Queue
          - 主线程（消费者）从队列中取出事件进行处理
          这种模式的好处：
          1. 解耦事件监听和事件处理
          2. 队列自带线程安全，无需额外加锁
          3. 可以在消费者端方便地添加过滤、去重等处理逻辑

      - category: caution
        content: |
          需要注意的陷阱和边界情况：
          1. Observer.start() 会创建守护线程，程序退出时必须调用 observer.stop()
             否则可能导致资源泄漏
          2. 某些编辑器（如 Vim）在保存时会触发多个事件（临时文件创建 + 重命名），
             这不是 bug，而是编辑器的正常行为，需要在应用层处理
          3. Windows 上的文件锁定可能导致事件延迟或丢失
          4. 大量文件同时变化时（如 git checkout），事件可能批量到达，
             需要确保事件处理不会成为瓶颈

    recommended_approach: |
      使用 watchdog 的 Observer + EventHandler 模式实现文件系统监控。
      选择 watchdog 是因为它是 Python 文件监控的事实标准，跨平台支持完善，
      且有活跃的社区维护。
      关键实现要点：
      1. 将 watchdog 原生事件转换为统一的 FileEvent 数据类（event_type, file_path, timestamp）
      2. 使用队列模式解耦事件生产和消费
      3. 通过 schedule() 的 recursive 参数控制递归监控
      4. 使用 threading.Event 实现优雅关闭
      5. 实现应用层 debounce 去重，消除跨平台事件差异

    confidence: high

    open_questions:
      - question: "在极端高频文件变化场景下（如日志密集写入，每秒数千个事件），事件队列是否会溢出？"
        impact: "risk"
        suggested_resolution: |
          执行时为队列设置最大长度（建议初始值 10000），
          超出时丢弃最旧的事件并输出警告日志。
          具体值需要根据实际场景的压力测试确定。

      - question: "watchdog 在 NFS/SMB 等网络文件系统上的行为如何？是否支持？"
        impact: "minor"
        suggested_resolution: |
          watchdog 对网络文件系统的支持有限，如果需要支持网络文件系统，
          可能需要使用轮询（PollingObserver）作为后备方案。
          当前版本先不考虑网络文件系统，后续按需添加。

    references:
      - "https://python-watchdog.readthedocs.io/en/stable/quickstart.html"
      - "https://python-watchdog.readthedocs.io/en/stable/api.html"
      - "https://github.com/gorakhargosh/watchdog"
      - "https://stackoverflow.com/questions/18624908/python-watchdog-monitoring-file-system"
```

### 无堵点的示例

以下是一个没有堵点但仍有研究价值的 MEU 的完整研究输出：

```yaml
research_results:
  - task_id: "1.2.1"
    task_name: "实现 CLI 参数解析模块"

    decision_summary: {}

    blockers: []

    key_findings:
      - category: approach
        content: |
          使用 Python 标准库 argparse 实现 CLI 参数解析。
          不使用第三方 CLI 框架（click、typer），以最小化项目依赖。
          argparse 是 Python 标准库的一部分，功能完全满足当前需求：
          支持位置参数、可选参数、类型验证、默认值、互斥参数等。
          将解析结果封装为 WatcherConfig 数据类，便于在模块间传递配置。

      - category: api
        content: |
          argparse 核心用法：
          ```python
          parser = argparse.ArgumentParser(
              description="文件监控 CLI 工具"
          )
          parser.add_argument('--path', required=True,
                              help='监控的目录路径')
          parser.add_argument('--recursive', action='store_true',
                              default=True, help='递归监控子目录')
          parser.add_argument('--filter', default='',
                              help='文件扩展名过滤，逗号分隔')
          parser.add_argument('--output-format',
                              choices=['json', 'text'],
                              default='text', help='输出格式')
          parser.add_argument('--debounce', type=int,
                              default=500, help='去重时间窗口(ms)')
          ```
          WatcherConfig 数据类：
          ```python
          @dataclass
          class WatcherConfig:
              path: Path
              recursive: bool = True
              filter_extensions: list[str] = field(default_factory=list)
              output_format: str = "text"
              debounce_ms: int = 500
          ```

      - category: caution
        content: |
          参数验证需要注意的问题：
          1. --filter 参数接收逗号分隔的字符串，需要转换为列表，
             并为每个扩展名添加点号前缀（如 "log" -> ".log"）
          2. --debounce 需要验证为正整数，负数或零应该报错
          3. --path 需要验证路径存在且为目录，不存在则报错并提示
          4. --output-format 使用 choices 参数限制可选值，argparse 会自动验证
          5. --recursive 使用 action='store_true'，注意不要和 default=True 冲突
             （store_true 在指定时设为 True，不指定时使用 default 值）

      - category: pattern
        content: |
          推荐的代码组织方式：
          1. parse_args(argv=None) 函数：接收参数列表，返回 WatcherConfig
          2. 在函数内部完成所有参数验证和转换
          3. 验证失败时抛出 argparse.ArgumentError，由 argparse 统一处理错误信息
          4. 将参数解析和业务逻辑分离，便于单元测试

    recommended_approach: |
      使用 Python 标准库 argparse 实现 CLI 参数解析，将解析结果封装为
      WatcherConfig 数据类。在解析函数中完成所有参数验证（路径存在性、
      值范围、类型转换），确保返回的 WatcherConfig 始终是有效的。
      不引入第三方 CLI 框架，以保持项目依赖最小化。

    confidence: high

    open_questions: []

    references:
      - "https://docs.python.org/3/library/argparse.html"
```
