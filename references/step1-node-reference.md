# Step 1 节点字段参考

本文档是任务拆解阶段的**详细参考手册**，包含节点字段完整参考、依赖关系详解、常见错误与避免方法、以及完整任务树示例。

> 本文档由 [step1-decompose.md](../steps/step1-decompose.md) 按需加载引用。

---

## 目录

- [1. 节点字段完整参考](#1-节点字段完整参考)
  - [1.1 字段总表](#11-字段总表)
  - [1.2 优先级详细指南](#12-优先级详细指南)
  - [1.3 复杂度详细指南](#13-复杂度详细指南)
  - [1.4 状态流转](#14-状态流转)
- [2. 依赖关系详解](#2-依赖关系详解)
  - [2.1 父子依赖（隐式）](#21-父子依赖隐式)
  - [2.2 跨分支依赖（显式）](#22-跨分支依赖显式)
  - [2.3 拆解后的依赖检查](#23-拆解后的依赖检查)
- [3. 拆解的常见错误与避免方法](#3-拆解的常见错误与避免方法)
- [4. 完整任务树示例](#4-完整任务树示例)

---

## 1. 节点字段完整参考

### 1.1 字段总表

任务树中的每个节点（包括根节点、分支节点和叶子节点）都包含以下字段：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | 是 | 节点唯一标识符。格式为 `{父节点id}.{序号}`，根节点为 `1`。例如：`1`、`1.1`、`1.2.3` |
| `name` | string | 是 | 简洁的任务名称，不超过 50 个字符。建议使用"动词 + 名词"格式，如"实现用户认证模块" |
| `description` | string | 是 | 详细的任务描述，包含：输入、输出、技术要求、验收标准。对于叶子节点（MEU），描述必须足够详细，使执行者可以独立完成 |
| `type` | string | 是 | 节点类型：`root`（根节点）/ `branch`（分支节点）/ `leaf`（叶子节点，即 MEU） |
| `parent` | string\|null | 是 | 父节点 ID。根节点的 parent 为 `null` |
| `children` | string[] | 是 | 子节点 ID 列表。叶子节点的 children 为空数组 `[]` |
| `dependencies` | string[] | 是 | 跨分支依赖的节点 ID 列表。**注意**：不要在此字段中填写父子依赖（父子依赖是隐式的）。叶子节点如果没有跨分支依赖则为空数组 |
| `priority` | string | 是 | 优先级：`critical` / `high` / `medium` / `low` |
| `estimated_complexity` | string | 是 | 预估复杂度：`trivial` / `low` / `medium` / `high` / `extreme` |
| `status` | string | 是 | 当前状态：`pending` / `researching` / `ready` / `in_progress` / `completed` / `blocked` |

### 1.2 优先级详细指南

| 级别 | 含义 | 使用场景 |
|---|---|---|
| `critical` | 阻塞所有其他任务，或属于项目核心功能 | 项目初始化、核心架构设计、关键路径上的任务 |
| `high` | 重要但不阻塞其他任务 | 主要功能模块、重要的技术选型研究 |
| `medium` | 常规功能，按标准顺序执行 | 大多数功能实现任务、一般性测试 |
| `low` | 锦上添花，可以在核心功能完成后再做 | 文档编写、UI 美化、非关键的性能优化 |

### 1.3 复杂度详细指南

| 级别 | 含义 | 典型特征 |
|---|---|---|
| `trivial` | 简单的配置或文件创建，无复杂逻辑 | 创建 `.gitignore`、编写简单的配置文件、定义常量 |
| `low` | 单一功能，逻辑清晰，无外部依赖 | 实现一个简单函数、编写一个 CLI 参数解析器 |
| `medium` | 需要设计思考，可能涉及多个组件协调 | 实现一个包含多个方法的 Service 类、编写中等复杂度的数据处理逻辑 |
| `high` | 复杂业务逻辑，需要架构设计，多系统交互 | 实现认证授权系统、设计数据库 Schema 和迁移策略 |
| `extreme` | 高技术风险，可能需要原型验证，涉及性能/安全 | 实现高并发消息处理、设计加密方案、性能关键路径优化 |

### 1.4 状态流转

```
pending → researching → ready → in_progress → completed
                ↓                        ↓
             blocked ←───────────────────┘
                ↓
             ready（问题解决后）
```

- `pending`：初始状态，等待处理
- `researching`：正在进行技术研究
- `ready`：研究完成，可以开始执行
- `in_progress`：正在执行中
- `completed`：执行完成
- `blocked`：被阻塞（依赖未满足或遇到问题）

---

## 2. 依赖关系详解

任务树中的依赖关系分为两种：**父子依赖（隐式）** 和 **跨分支依赖（显式）**。正确理解和使用这两种依赖是保证执行顺序正确性的关键。

### 2.1 父子依赖（隐式）

父子关系天然构成依赖关系——**子节点隐式依赖于其所有祖先节点**。这种依赖**不需要**在 `dependencies` 字段中声明。

**规则**：
- 子节点隐式依赖于其直接父节点和所有祖先节点
- 兄弟节点之间**没有**隐式依赖（除非通过 `dependencies` 字段显式声明）

**图示**：

```
1. 项目根节点
├── 1.1 技术选型
│   ├── 1.1.1 研究前端框架    ← 隐式依赖 1.1
│   └── 1.1.2 研究后端框架    ← 隐式依赖 1.1
└── 1.2 功能实现
    ├── 1.2.1 前端开发        ← 隐式依赖 1.2, 1.1.1
    └── 1.2.2 后端开发        ← 隐式依赖 1.2, 1.1.2
```

在上述示例中：
- `1.1.1` 隐式依赖 `1.1`（必须先完成技术选型，才能研究前端框架）
- `1.2.1` 隐式依赖 `1.2` 和 `1.1.1`（必须先完成功能实现分支的启动和前端框架研究）
- `1.2.1` 和 `1.2.2` 之间**没有**隐式依赖（它们是兄弟节点）

### 2.2 跨分支依赖（显式）

当一个节点需要依赖**非祖先节点**的其他节点时，必须在 `dependencies` 字段中**显式声明**。

**规则**：
- 只声明**直接依赖**，不需要声明传递依赖（如果 A 依赖 B，B 依赖 C，A 只需要声明依赖 B，不需要声明依赖 C）
- 只声明**跨分支**的依赖，不声明父子依赖

**图示**：

```
1. 项目根节点
├── 1.1 前端模块
│   └── 1.1.1 实现 API 调用层
└── 1.2 后端模块
    ├── 1.2.1 定义 API 接口规范
    └── 1.2.2 实现 API 接口
```

在上述示例中：
- `1.1.1`（前端 API 调用层）需要知道 `1.2.1`（后端 API 接口规范）定义的接口格式
- 因此 `1.1.1` 需要在 `dependencies` 中显式声明 `["1.2.1"]`

```yaml
- id: "1.1.1"
  name: "实现 API 调用层"
  dependencies:
    - "1.2.1"    # 跨分支依赖：需要后端 API 接口规范
```

### 2.3 拆解后的依赖检查

完成拆解后，必须进行以下四项检查：

| 检查项 | 说明 | 检查方法 |
|---|---|---|
| **无环性** | 依赖图中不存在循环依赖 | 使用拓扑排序算法验证，如果排序失败说明存在环 |
| **完整性** | 每个被引用的依赖 ID 都对应一个已存在的节点 | 遍历所有 `dependencies` 字段，检查引用的 ID 是否都在任务树中 |
| **必要性** | 每个显式声明的依赖都是真正需要的 | 对每个依赖问"如果去掉这个依赖，执行者是否能正常完成任务？" |
| **方向性** | 依赖方向符合逻辑（实现依赖设计，而非设计依赖实现） | 检查是否存在"低层级任务依赖高层级任务"的倒置情况 |

---

## 3. 拆解的常见错误与避免方法

### 3.1 粒度不一致

**问题描述**：同一层级的节点粒度差异过大，有的节点是一个完整的功能模块，有的只是一个简单的配置文件。

**错误示例**：

```
1. 项目
├── 1.1 实现用户系统        ← 太大，包含注册、登录、权限等多个功能
├── 1.2 创建 config.json    ← 太小，只是一个配置文件
└── 1.3 部署                ← 太模糊，没有说明部署什么、部署到哪里
```

**避免方法**：
- 对同一层级的所有节点统一应用 MEU 四条件检查
- 如果某个节点明显比同层其他节点大或小，调整其层级或拆分/合并
- 参考粒度指南：同一层级的节点预估工作量差异不应超过 2 倍

### 3.2 遗漏隐含任务

**问题描述**：只列出了核心功能任务，遗漏了项目初始化、配置、测试、文档、错误处理等隐含任务。

**避免方法**：

完成拆解后，逐项检查以下清单：

- [ ] 项目初始化 / 脚手架搭建
- [ ] 配置文件和环境变量
- [ ] 错误处理和异常情况
- [ ] 日志记录
- [ ] 测试（单元测试 / 集成测试）
- [ ] 文档（用户文档 / 开发文档）
- [ ] 部署 / 运维相关任务
- [ ] 依赖安装和版本管理

### 3.3 过早涉及实现细节

**问题描述**：在拆解阶段就开始考虑代码级别的实现细节，导致拆解过于技术化而忽略了任务本身的逻辑结构。

**避免方法**：
- **拆解阶段回答"做什么"**，**研究阶段回答"怎么做"**
- 如果拆解过程中发现需要技术细节才能决定如何拆分，将该节点标记为"需要研究"，先进行研究
- 拆解描述中使用业务语言而非技术实现语言（例如用"用户认证"而不是"JWT 中间件"）

### 3.4 过度耦合的依赖

**问题描述**：跨分支依赖过多，导致几乎所有任务都必须串行执行，失去了并行执行的可能性。

**避免方法**：
- 合并紧耦合的任务为一个 MEU
- 在耦合的任务之间引入接口/契约抽象（例如先定义 API 接口规范，前端和后端分别实现）
- 添加抽象层来解耦（例如通过配置文件、适配器模式等）

### 3.5 忽略非功能性需求

**问题描述**：任务树中只包含功能性任务，忽略了性能、安全、可用性等非功能性需求。

**避免方法**：
- 在拆解时同步审视非功能性需求清单
- 为每个重要的非功能性需求创建对应的 MEU
- 常见的非功能性需求 MEU 类型：
  - 性能优化（响应时间、吞吐量、资源占用）
  - 安全加固（认证、授权、数据加密、输入验证）
  - 可用性保障（错误恢复、降级策略、健康检查）
  - 可维护性（日志、监控、文档）

---

## 4. 完整任务树示例

以下是一个"文件监控 CLI 工具"的完整任务树示例，包含 14 个节点，展示了从根节点到叶子节点的完整拆解过程。

```yaml
task_tree:
  - id: "1"
    name: "文件监控 CLI 工具"
    description: |
      一个命令行工具，用于监控指定目录中的文件变化事件。
      支持事件过滤（按文件扩展名）、去重（时间窗口去重）和自定义输出格式（文本/JSON）。
      技术栈：Python 3.11+，watchdog 库。
      目标用户：开发者和运维人员。
    type: root
    parent: null
    children: ["1.1", "1.2", "1.3", "1.4"]
    dependencies: []
    priority: critical
    estimated_complexity: high
    status: pending

  - id: "1.1"
    name: "项目初始化"
    description: |
      创建项目的基础结构，包括目录布局、配置文件和依赖管理。
      这是所有后续任务的基础。
    type: branch
    parent: "1"
    children: ["1.1.1", "1.1.2"]
    dependencies: []
    priority: critical
    estimated_complexity: low
    status: pending

  - id: "1.1.1"
    name: "创建项目结构和配置文件"
    description: |
      创建标准 Python 项目结构：
      - src/filewatcher/       （源代码目录）
      - tests/                 （测试目录）
      - pyproject.toml         （项目配置和依赖管理，使用 setuptools）
      - .gitignore             （Git 忽略规则）
      在 pyproject.toml 中声明：
      - Python >= 3.11
      - 核心依赖：watchdog
      - 开发依赖：pytest, ruff
    type: leaf
    parent: "1.1"
    children: []
    dependencies: []
    priority: critical
    estimated_complexity: trivial
    status: pending

  - id: "1.1.2"
    name: "配置开发环境和代码规范"
    description: |
      配置开发工具链：
      - ruff（代码检查和格式化）：创建 ruff.toml，配置行长度 120、导入排序规则
      - pytest（测试框架）：创建 pytest.ini 和 conftest.py，配置测试发现规则
      输出文件：ruff.toml, pytest.ini, conftest.py
    type: leaf
    parent: "1.1"
    children: []
    dependencies: ["1.1.1"]
    priority: medium
    estimated_complexity: trivial
    status: pending

  - id: "1.2"
    name: "核心模块开发"
    description: |
      开发四个核心模块：CLI 参数解析、文件系统监控、事件过滤与去重、日志输出。
      这四个模块相互独立，可以并行开发（通过接口约束解耦）。
    type: branch
    parent: "1"
    children: ["1.2.1", "1.2.2", "1.2.3", "1.2.4"]
    dependencies: ["1.1"]
    priority: critical
    estimated_complexity: high
    status: pending

  - id: "1.2.1"
    name: "实现 CLI 参数解析模块"
    description: |
      使用 Python 标准库 argparse 实现 CLI 参数解析：
      - --path（必填）：监控的目录路径，需要验证路径存在且为目录
      - --recursive（可选，默认 True）：是否递归监控子目录
      - --filter（可选）：文件扩展名过滤列表，逗号分隔，如 ".log,.txt"
      - --output-format（可选，默认 text）：输出格式，可选 json 或 text
      - --debounce（可选，默认 500）：去重时间窗口，单位毫秒，需要正整数验证
      将解析结果封装为 WatcherConfig 数据类。
      输出文件：src/filewatcher/cli.py, src/filewatcher/models.py（WatcherConfig）
    type: leaf
    parent: "1.2"
    children: []
    dependencies: ["1.1.1"]
    priority: critical
    estimated_complexity: low
    status: pending

  - id: "1.2.2"
    name: "实现文件系统监控模块"
    description: |
      基于 watchdog 库实现文件系统监控：
      - 使用 Observer + EventHandler 模式
      - 监听 created、modified、deleted 三种事件
      - 将 watchdog 原生事件转换为统一的 FileEvent 数据类（event_type, file_path, timestamp）
      - 支持通过 schedule() 参数控制递归监控
      - 使用 queue.Queue 解耦事件生产和消费
      - 使用 threading.Event 实现优雅关闭
      输出文件：src/filewatcher/watcher.py, src/filewatcher/models.py（FileEvent）
    type: leaf
    parent: "1.2"
    children: []
    dependencies: ["1.1.1"]
    priority: critical
    estimated_complexity: medium
    status: pending

  - id: "1.2.3"
    name: "实现事件过滤和去重模块"
    description: |
      实现两层事件处理：
      1. 扩展名过滤：根据 --filter 参数指定的扩展名列表过滤事件
         - 空列表表示不过滤（保留所有事件）
         - 匹配时忽略大小写（.LOG 和 .log 都匹配）
      2. 时间窗口去重：使用 time.monotonic()（不是 time.time()）进行时间比较
         - 同一文件在指定时间窗口内（默认 500ms）的多个事件合并为一个
         - 不同文件的事件不受去重影响
         - 时间窗口参数可配置
      输出文件：src/filewatcher/filter.py
    type: leaf
    parent: "1.2"
    children: []
    dependencies: ["1.2.2"]
    priority: high
    estimated_complexity: medium
    status: pending

  - id: "1.2.4"
    name: "实现日志输出模块"
    description: |
      实现两种输出格式的日志输出：
      1. 人类可读文本格式（text）：输出到 stdout，每行一个事件，格式如
         "[2024-01-15 10:30:00] [CREATED] /path/to/file.txt"
      2. 结构化 JSON 格式（json）：输出到 stdout，每行一个 JSON 对象，字段包含
         timestamp, event_type, file_path
      通过 --output-format 参数选择输出格式。
      输出文件：src/filewatcher/output.py
    type: leaf
    parent: "1.2"
    children: []
    dependencies: ["1.2.3"]
    priority: high
    estimated_complexity: low
    status: pending

  - id: "1.3"
    name: "集成和主入口"
    description: |
      将所有核心模块集成到主入口点，实现完整的端到端工作流。
    type: branch
    parent: "1"
    children: ["1.3.1"]
    dependencies: ["1.2"]
    priority: critical
    estimated_complexity: medium
    status: pending

  - id: "1.3.1"
    name: "实现主入口和模块集成"
    description: |
      创建程序主入口，串联所有模块：
      - 创建 src/filewatcher/__main__.py（支持 python -m filewatcher 方式运行）
      - 创建 src/filewatcher/app.py（主应用逻辑）
      执行流程：
      1. 解析 CLI 参数（cli.py）
      2. 初始化文件监控器（watcher.py）
      3. 连接事件过滤和去重（filter.py）
      4. 连接日志输出（output.py）
      5. 注册 Ctrl+C 信号处理，实现优雅退出
      6. 启动监控循环
    type: leaf
    parent: "1.3"
    children: []
    dependencies: ["1.2.1", "1.2.3", "1.2.4"]
    priority: critical
    estimated_complexity: medium
    status: pending

  - id: "1.4"
    name: "测试和文档"
    description: |
      编写单元测试和用户文档，确保代码质量和可用性。
    type: branch
    parent: "1"
    children: ["1.4.1", "1.4.2"]
    dependencies: ["1.3"]
    priority: medium
    estimated_complexity: medium
    status: pending

  - id: "1.4.1"
    name: "编写单元测试"
    description: |
      为所有核心模块编写单元测试：
      - tests/test_cli.py：测试参数解析的各种输入组合
      - tests/test_watcher.py：测试文件事件监听（使用临时目录）
      - tests/test_filter.py：测试扩展名过滤和去重逻辑
      - tests/test_output.py：测试两种输出格式的正确性
      覆盖率要求 >= 80%。
      输出文件：tests/test_*.py
    type: leaf
    parent: "1.4"
    children: []
    dependencies: ["1.3.1"]
    priority: medium
    estimated_complexity: medium
    status: pending

  - id: "1.4.2"
    name: "编写用户文档"
    description: |
      编写 README.md，包含以下章节：
      - 项目简介（一句话说明工具用途）
      - 安装方法（pip install 和从源码安装）
      - 使用示例（各种参数组合的命令行示例）
      - 开发指南（环境搭建、运行测试、代码规范）
      输出文件：README.md
    type: leaf
    parent: "1.4"
    children: []
    dependencies: ["1.3.1"]
    priority: low
    estimated_complexity: low
    status: pending
```

**任务树结构可视化**：

```
1. 文件监控 CLI 工具
├── 1.1 项目初始化
│   ├── 1.1.1 创建项目结构和配置文件          [leaf, trivial]
│   └── 1.1.2 配置开发环境和代码规范          [leaf, trivial]
├── 1.2 核心模块开发
│   ├── 1.2.1 实现 CLI 参数解析模块           [leaf, low]
│   ├── 1.2.2 实现文件系统监控模块            [leaf, medium]
│   ├── 1.2.3 实现事件过滤和去重模块          [leaf, medium]
│   └── 1.2.4 实现日志输出模块               [leaf, low]
├── 1.3 集成和主入口
│   └── 1.3.1 实现主入口和模块集成            [leaf, medium]
└── 1.4 测试和文档
    ├── 1.4.1 编写单元测试                   [leaf, medium]
    └── 1.4.2 编写用户文档                   [leaf, low]
```

**依赖关系可视化**：

```
1.1.1 ──→ 1.1.2
  │
  ├──→ 1.2.1 ──→ 1.3.1 ──→ 1.4.1
  │                         ──→ 1.4.2
  ├──→ 1.2.2 ──→ 1.2.3 ──→ 1.2.4 ──→ 1.3.1
  │
  └──→ 1.2（分支依赖 1.1）
```
