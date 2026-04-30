# 置信度、验证方法与开放问题处理

> 本文件是 Step 2（技术研究）的参考文件。包含信息源优先级、交叉验证方法、置信度定义、开放问题处理策略和研究深度要求。
> 从 [step2-research.md](../steps/step2-research.md) 加载核心指南后，需要了解置信度定义或开放问题处理策略时按需读取。

---

## 目录

- [第 2 轮：定向搜索 — 信息源与查询模板](#第-2-轮定向搜索--信息源与查询模板)
- [第 3 轮：交叉验证 — 详细方法](#第-3-轮交叉验证--详细方法)
- [置信度级别的定义和使用建议](#置信度级别的定义和使用建议)
- [开放问题的处理策略](#开放问题的处理策略)
- [研究深度要求表](#研究深度要求表)

---

## 第 2 轮：定向搜索 — 信息源与查询模板

### 查询模板

不同类型的堵点使用不同的查询模板：

| 堵点类型 | 查询模板 | 示例 |
|---|---|---|
| API 用法 | `{库名} {功能名} 官方文档` | `watchdog Observer recursive 官方文档` |
| 技术选型 | `{方案A} vs {方案B} {使用场景} 对比` | `WebSocket vs SSE 实时通知 对比` |
| 最佳实践 | `{技术} {场景} 最佳实践 {年份}` | `Python 文件监控 最佳实践 2024` |
| 兼容性 | `{库名} {目标环境} 兼容性` | `watchdog macOS Linux 兼容性` |
| 性能优化 | `{技术} {场景} 性能优化` | `Python 高频文件事件 去重 性能` |

### 信息源类型分类

| 信息源类型 | 代号 | 可信度基准 |
|---|---|---|
| 官方文档 | `official_doc` | 高——最权威，但可能不够详细 |
| GitHub 仓库 / 示例代码 | `github` | 高——可直接运行验证，最实用 |
| 权威技术书籍 | `book` | 高——经过编辑审核，但可能滞后 |
| 技术博客（附代码示例） | `tech_blog` | 中——需要判断作者可信度 |
| Stack Overflow 高票回答 | `stackoverflow` | 中——需要检查回答日期和投票数 |
| 其他 | `other` | 低——需要交叉验证 |

### 第 2 轮必须遵守的规则

- **每个堵点至少从 2 个独立信息源获取信息**——绝不依赖单一来源
- **必须记录信息源类型**：官方文档 / GitHub / 技术博客 / StackOverflow / 其他
- **必须对搜索结果做初步可信度评估**：评估信息的时效性、来源权威性、内容完整性
- **必须检查结果时效性**：优先选择最近 2 年内的内容，技术文档尤其要注意版本号
- 如果搜索结果不足以解决堵点，将该堵点标记为 `unresolved`，并在 `open_questions` 中记录

### 第 2 轮输出格式

```yaml
# 第 2 轮：定向搜索与信息收集（内部输出）
round_2_search:
  - blocker: "watchdog Observer 和 EventHandler 的使用方式"
    searches:
      - query: "watchdog Observer schedule recursive 官方文档"
        results:
          - source_type: "official_doc"
            url: "https://python-watchdog.readthedocs.io/en/stable/api.html"
            key_takeaway: "Observer.schedule(event_handler, path, recursive=False) 注册监控"
            credibility: "high"
            recency: "2024"
          - source_type: "github"
            url: "https://github.com/gorakhargosh/watchdog/blob/master/docs/source/api.rst"
            key_takeaway: "示例代码展示了完整的 Observer + EventHandler 用法"
            credibility: "high"
            recency: "2024"
        preliminary_assessment: "两个来源一致确认 API 用法，信息可信"

      - query: "watchdog FileSystemEventHandler on_created on_modified 示例"
        results:
          - source_type: "stackoverflow"
            url: "https://stackoverflow.com/questions/18624908"
            key_takeaway: "实际使用示例展示了事件处理的完整流程"
            credibility: "medium"
            recency: "2023"
          - source_type: "tech_blog"
            url: "https://realpython.com/..."
            key_takeaway: "详细教程覆盖了常见陷阱和最佳实践"
            credibility: "medium"
            recency: "2024"
        preliminary_assessment: "两个来源补充了实际使用经验和陷阱信息"

  - blocker: "macOS 和 Linux 文件事件行为差异"
    searches:
      - query: "watchdog FSEvents inotify event differences macOS Linux"
        results:
          - source_type: "official_doc"
            url: "https://python-watchdog.readthedocs.io/en/stable/..."
            key_takeaway: "macOS 使用 FSEvents，Linux 使用 inotify，Windows 使用 ReadDirectoryChangesW"
            credibility: "high"
            recency: "2024"
          - source_type: "tech_blog"
            url: "https://martin.ankerl.com/..."
            key_takeaway: "FSEvents 有事件合并行为（coalescing），inotify 不会合并"
            credibility: "medium"
            recency: "2023"
        preliminary_assessment: "两个来源确认事件合并行为差异，需要进一步验证细节"
```

---

## 第 3 轮：交叉验证 — 详细方法

### 操作方法

1. **交叉验证**：对第 2 轮收集的关键结论，从不同来源进行比对
2. **方案对比分析**：对每个堵点的候选方案做 pros/cons 分析（至少 2 个方案）
3. **代码片段准备**：对关键 API/方案提供最小可运行代码片段
4. **置信度评估**：根据交叉验证结果更新每个结论的置信度

### 交叉验证规则

- 如果多个独立来源结论一致 → 置信度标记为 `high`
- 如果来源结论矛盾 → **必须**进一步搜索或标记为 `low confidence`
- 如果只有一个来源支持某个结论 → 置信度标记为 `medium`，需要额外验证
- 版本差异导致的信息不一致 → 明确标注适用的版本范围

### 信息源优先级（从高到低）

| 优先级 | 信息源类型 | 可靠性说明 |
|---|---|---|
| 1 | 官方文档 | 最权威，但可能不够详细 |
| 2 | 官方示例代码 / GitHub 仓库 | 可直接运行验证，最实用 |
| 3 | 权威技术书籍 | 经过编辑审核，质量高但可能滞后 |
| 4 | 高质量技术博客（附代码示例和测试） | 实用性强，但需要判断作者可信度 |
| 5 | Stack Overflow 高票回答 | 需要检查回答日期和投票数，过时回答可能误导 |

### 方案对比分析要求

对每个堵点，必须列出至少 2 个候选方案，并对每个方案分析：

| 分析维度 | 说明 |
|---|---|
| `name` | 方案名称 |
| `description` | 方案描述 |
| `pros` | 优点列表（至少 2 个） |
| `cons` | 缺点列表（至少 1 个） |
| `feasibility` | 可行性评估：`high` / `medium` / `low` |
| `code_snippet` | 最小可运行代码片段（如有） |

### 第 3 轮输出格式

```yaml
# 第 3 轮：交叉验证与深度分析（内部输出）
round_3_validation:
  cross_validation:
    - claim: "watchdog 使用 Observer + EventHandler 模式"
      sources_supporting:
        - "官方文档 API 参考"
        - "GitHub 示例代码"
        - "Stack Overflow 高票回答"
      sources_contradicting: []
      conclusion: "多个来源一致确认"
      confidence: "high"

    - claim: "macOS FSEvents 会合并快速连续发生的事件"
      sources_supporting:
        - "watchdog 官方文档"
        - "技术博客对比分析"
      sources_contradicting: []
      conclusion: "两个来源确认，但细节描述不完全一致"
      confidence: "high"

    - claim: "queue.Queue 可以安全地用于多线程事件传递"
      sources_supporting:
        - "Python 官方文档（threading 安全保证）"
        - "技术博客最佳实践"
      sources_contradicting: []
      conclusion: "Python 标准库明确保证线程安全"
      confidence: "high"

  solution_analysis:
    - blocker: "文件监控库选型"
      solutions:
        - name: "watchdog"
          description: "使用 watchdog 库的 Observer + EventHandler 模式"
          pros:
            - "跨平台支持（macOS/Linux/Windows）"
            - "社区活跃，长期维护"
            - "API 设计简洁，学习曲线低"
          cons:
            - "性能略低于平台原生方案（如 pyinotify）"
            - "网络文件系统支持有限"
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

        - name: "pyinotify"
          description: "使用 pyinotify 库，基于 Linux inotify 接口"
          pros:
            - "Linux 平台上性能最优"
            - "底层控制力强，可精细配置"
          cons:
            - "仅支持 Linux，无法跨平台"
            - "项目维护不活跃，最后更新于 2015 年"
            - "API 相对复杂"
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

        - name: "自研轮询方案"
          description: "使用 os.listdir() 定时轮询目录变化"
          pros:
            - "零外部依赖"
            - "适用于任何文件系统（包括网络文件系统）"
          cons:
            - "实时性差，依赖轮询间隔"
            - "资源消耗高（频繁扫描大目录）"
            - "需要自行实现变化检测逻辑"
          feasibility: "medium"
          code_snippet: |
            import os, time, hashlib

            def get_dir_state(path):
                state = {}
                for f in os.listdir(path):
                    fp = os.path.join(path, f)
                    state[f] = os.path.getmtime(fp)
                return state

            while True:
                current = get_dir_state(".")
                if current != previous:
                    print("Changes detected!")
                previous = current
                time.sleep(1)

      comparison_summary: |
        watchdog 在跨平台支持、社区活跃度和 API 简洁性方面显著优于其他方案。
        pyinotify 仅限 Linux 且已停止维护。自研轮询方案实时性差。
        推荐使用 watchdog。
```

---

## 置信度级别的定义和使用建议

### 级别定义

| 级别 | 定义 | 在执行阶段的使用方式 |
|---|---|---|
| `high` | 结论可靠，有官方文档或多个独立来源交叉验证 | 可以直接在执行 Prompt 中引用，执行者无需额外验证 |
| `medium` | 结论很可能正确，有单一可靠来源支持 | 在执行 Prompt 中引用，建议执行者做基本验证 |
| `low` | 仅供参考，需要额外验证才能用于实现 | **必须**在执行 Prompt 中包含验证步骤，执行者不能直接采用 |

### 降低置信度的因素

- 信息来源不可靠或已过时
- 研究结果存在矛盾（不同来源给出不同结论）
- AI 知识截止日期可能遗漏近期变化
- 快速迭代的领域（AI 框架、云服务 API、前端工具链）
- 缺乏可运行的示例代码

### 提高置信度的因素

- 官方文档明确说明
- 多个独立来源一致确认
- 存在可运行的示例代码
- AI 对该领域有丰富且高确定性的知识
- 有实际的基准测试数据支持

### 置信度与执行 Prompt 的关系

| 置信度 | 执行 Prompt 中的处理方式 |
|---|---|
| `high` | 直接写入"推荐方案"部分，不附加验证要求 |
| `medium` | 写入"推荐方案"部分，附加"建议验证 XXX"的提示 |
| `low` | 写入"参考信息"部分（而非"推荐方案"），附加"必须先验证 XXX 再实现"的要求 |

---

## 开放问题的处理策略

开放问题（Open Questions）是研究中**无法完全解决**的问题。这些问题需要在后续阶段（评估阶段或执行阶段）进行处理。

### 按影响级别分类

| 影响级别 | 定义 | 处理策略 |
|---|---|---|
| `blocking` | **阻塞执行**——必须在执行前解决，否则无法完成任务 | 考虑添加一个专门的研究 MEU；或者在执行 Prompt 中要求执行者首先解决该问题 |
| `risk` | **有风险但不阻塞**——可能影响实现质量或导致返工 | 在执行 Prompt 中标记为风险点，要求执行者记录实际处理方式 |
| `minor` | **影响较小**——对实现影响不大，执行者可以自行决定 | 在执行 Prompt 中简要提及，允许执行者根据实际情况灵活处理 |

### 开放问题的常见来源

1. **需要实际环境验证**：代码在目标环境中是否能正常运行（如特定操作系统、特定硬件）
2. **依赖外部决策**：需要其他团队或利益相关者的决策（如 API 接口设计、数据格式确认）
3. **信息在公开渠道不可获得**：商业 API 的内部行为、私有系统的技术细节
4. **需要原型验证**：性能是否满足要求、方案是否可行，只有实际实现后才能确认

### 记录要求

每个开放问题**必须**包含以下信息：

1. **清晰的问题描述**：让读者一看就知道问题是什么
2. **对执行的具体影响**：说明如果这个问题不解决，会如何影响执行
3. **建议的解决方案**：即使只是一个方向性的建议（如"执行时根据实际测试结果决定"），也比没有建议好

**记录示例**：

```yaml
open_questions:
  - question: "在极端高频文件变化场景下（如日志密集写入），事件队列是否会溢出？"
    impact: "risk"
    suggested_resolution: |
      执行时设置队列最大长度，超出时丢弃最旧的事件并输出警告日志。
      具体队列大小需要根据实际场景测试确定，建议初始值设为 10000。
```

---

## 研究深度要求表

根据 MEU 的复杂度，研究深度应有不同要求：

| MEU 复杂度 | 判断标准 | 研究深度要求 |
|---|---|---|
| **简单** | 无堵点或仅有 1 个低风险堵点 | 第 1 轮分析 + 第 4 轮整合即可，第 2、3 轮可简化 |
| **中等** | 有 1-2 个堵点，涉及技术选型或 API 调研 | 完整 4 轮迭代，每个堵点至少 2 个信息源 |
| **复杂** | 有 3 个以上堵点，或涉及性能优化、跨系统集成 | 完整 4 轮迭代，每个堵点至少 3 个信息源，必须有代码片段验证 |
