# 执行阶段 Prompt 模板

> **用途**: 为单个 MEU 生成可直接执行的精简 Prompt
> **使用方法**: 将所有占位符替换为实际内容，直接复制使用
> **输入**: 项目信息、MEU 定义、研究结论、评估结果
> **输出**: 可直接交付给 LLM 执行的精简指令（约 60 行）

---

## 模板

将以下内容复制后，替换所有 `{占位符}` 为实际内容：

````
# 执行: {unit_name}

## 背景

- **项目**: {project_name}
- **批次**: 第 {batch} 批次
- **前置产出**: {completed_dependencies}

## 任务

{task_description}

## 上下文

> 读取以下文件获取详细上下文：
> - 任务定义: `framework/task-tree.yaml` → 节点 "{task_id}"
> - 研究结论: `framework/research.yaml` → 条目 "{task_id}"
> - 验收标准: `framework/benchmarks.yaml` → 条目 "{task_id}"

## 避坑要点

{pitfall_guide}

## 决策

{decision_log}

## 约束

{constraints}

## 产出

{output_requirements}

## 验收

{acceptance_criteria}
````

---

## 占位符说明

| 占位符 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `{unit_name}` | string | MUST | MEU 名称，格式如"MEU-003: 实现事件过滤和去重" |
| `{project_name}` | string | MUST | 项目名称 |
| `{batch}` | string | MUST | 批次号，如"3" |
| `{completed_dependencies}` | string | MUST | 已完成的前置 MEU 及关键产出（1-2 句话） |
| `{task_description}` | string | MUST | 目标描述（2-3 句话，不含范围和预期产出——这些在 framework/ 中） |
| `{task_id}` | string | MUST | MEU 的任务 ID，用于在 framework/ YAML 文件中定位对应条目 |
| `{pitfall_guide}` | string | MUST | 从 research 的 pitfalls 提取最关键的 3-5 条（❌→✅ 格式） |
| `{decision_log}` | string | MUST | 从 research 的 decision_summary 提取关键决策表（决策/选择/理由） |
| `{constraints}` | string | MUST | 技术限制列表 |
| `{output_requirements}` | string | MUST | 必须交付的文件列表 |
| `{acceptance_criteria}` | string | MUST | 引用 benchmarks.yaml 中的具体条目（如 "- [ ] B1.1: 文件存在"） |

---

## 填充示例

以下展示一个精简版的填充示例。项目为"文件监控 CLI 工具"，MEU 为"实现事件过滤和去重"。对比原来的 9 段式版本（约 120 行），精简版仅约 60 行——详细数据通过引用从 `framework/` 按需加载。

### 示例：文件监控 CLI 工具 — MEU-003

````
# 执行: MEU-003 — 实现事件过滤和去重

## 背景

- **项目**: filewatch — 命令行文件监控工具，实时输出结构化事件
- **批次**: 第 3 批次
- **前置产出**: MEU-001 定义了 `--watch`/`--filter`/`--exclude`/`--dedup-window` 参数规格；MEU-002 实现了基于 `notify` 的文件监听器，输出 `RawEvent` 事件流

## 任务

在文件监听核心之上构建事件过滤层，实现 glob 模式文件类型过滤、路径排除规则、基于滑动时间窗口的事件去重。产出可独立测试的 `src/filewatch/filter.py` 模块。

## 上下文

> 读取以下文件获取详细上下文：
> - 任务定义: `framework/task-tree.yaml` → 节点 "MEU-003"
> - 研究结论: `framework/research.yaml` → 条目 "MEU-003"
> - 验收标准: `framework/benchmarks.yaml` → 条目 "MEU-003"

## 避坑要点

- ❌ 使用 `time.time()` 作为去重时间戳 → ✅ 使用 `time.monotonic()`（不受 NTP 同步影响）
- ❌ 不加锁直接读写去重缓存字典 → ✅ 使用 `threading.Lock`（文件监听器在独立线程运行）
- ❌ 使用 `event.path`（Path 对象）作为缓存 key → ✅ 使用 `(str(event.path), event.event_type)` 元组
- ❌ 去重缓存无限增长 → ✅ 设置最大容量（建议 10000 条），超出时清理最旧的 50%
- ❌ 忽略编辑器多事件触发 → ✅ debounce 窗口 >= 500ms，覆盖 Vim/VSCode 保存过程

## 决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 去重算法 | 滑动时间窗口 | 精确控制去重粒度，实现简单 |
| 过滤器组合 | 责任链模式 | 支持动态添加/移除，链式 API 优雅 |
| 过滤器接口 | 返回 `Optional[RawEvent]` | 语义清晰：None = 丢弃，RawEvent = 放行 |
| 时间比较 | `time.monotonic()` | 不受系统时间调整影响 |

## 约束

- MUST 使用 Python 3.11+ 特性
- MUST NOT 引入新的第三方依赖
- MUST NOT 修改 `watcher.py` 中的任何代码
- MUST 使用 `threading.Lock` 保证线程安全
- MUST 使用 `from __future__ import annotations`

## 产出

1. `src/filewatch/filter.py` — 包含 `BaseFilter`、`GlobFilter`、`DedupFilter`、`FilterChain`
2. `tests/test_filter.py` — 单元测试，覆盖率 >= 90%
3. `src/filewatch/__init__.py` — 更新导出

## 验收

- [ ] B3.1: GlobFilter 正确匹配 include/exclude 模式
- [ ] B3.2: DedupFilter 在 500ms 窗口内正确去重
- [ ] B3.3: FilterChain 按顺序执行，任一返回 None 则终止
- [ ] B3.4: 单元测试覆盖率 >= 90%
- [ ] B3.5: 通过 `mypy --strict` 类型检查
- [ ] B3.6: 10,000 事件/秒吞吐量下缓存内存 < 10MB
````

---

## 使用说明

### 如何使用此模板

1. **前置条件**：MUST 已完成拆解、研究、评估三个阶段，且 `framework/` 目录下已生成对应的 YAML 文件
2. **收集信息**：为每个占位符准备实际内容
   - 从拆解阶段获取：MEU 定义、批次信息
   - 从研究阶段获取：关键 pitfalls（提取 3-5 条）、关键决策（提取决策表）
   - 从评估阶段获取：验收标准条目 ID、产出建议、约束条件
3. **复制模板**：复制上方代码块中的模板部分（非示例部分）
4. **替换占位符**：将所有 `{占位符}` 替换为实际内容
5. **验证引用**：确认 `{task_id}` 在 `framework/task-tree.yaml`、`framework/research.yaml`、`framework/benchmarks.yaml` 中均存在对应条目
6. **验证传导质量**：对照下方「研究价值传导检查清单」确认研究阶段信息已完整传递
7. **交付执行**：将填充后的 Prompt 交付给 LLM 执行

### 占位符填充指南

#### {unit_name}

格式：`MEU-XXX — 任务名称`

来源：拆解阶段的 MEU 定义

示例：`MEU-003 — 实现事件过滤和去重`

#### {project_name}

内容：项目名称 + 一句话描述

来源：原始想法或项目文档

示例：`filewatch — 命令行文件监控工具，实时输出结构化事件`

#### {batch}

内容：批次编号

来源：执行计划中的批次信息

示例：`3`

#### {completed_dependencies}

内容：已完成的前置 MEU 及关键产出，1-2 句话概括

来源：执行计划中的批次信息 + 前置 MEU 的实际产出

示例：`MEU-001 定义了 --watch/--filter/--exclude/--dedup-window 参数规格；MEU-002 实现了基于 notify 的文件监听器，输出 RawEvent 事件流`

#### {task_description}

内容：目标描述，2-3 句话，不含范围和预期产出（这些在 framework/ 中）

来源：拆解阶段的 MEU 描述

示例：`在文件监听核心之上构建事件过滤层，实现 glob 模式文件类型过滤、路径排除规则、基于滑动时间窗口的事件去重。产出可独立测试的 src/filewatch/filter.py 模块。`

#### {task_id}

内容：MEU 的任务 ID，用于在 framework/ YAML 文件中定位对应条目

来源：拆解阶段的 MEU 定义

示例：`MEU-003`

#### {pitfall_guide}

内容：从 research 的 pitfalls 提取最关键的 3-5 条，使用 ❌→✅ 格式

来源：research_results 中对应 MEU 的 pitfalls + key_findings[category=caution]

格式示例：
```markdown
- ❌ 使用 `time.time()` 作为去重时间戳 → ✅ 使用 `time.monotonic()`（不受 NTP 同步影响）
- ❌ 不加锁直接读写缓存 → ✅ 使用 `threading.Lock`
```

**如果本 MEU 无关键 pitfalls**：保留段落标题，在下方注明"研究阶段未发现关键坑点"，不要省略段落。

#### {decision_log}

内容：从 research 的 decision_summary 提取关键决策表（决策/选择/理由三列）

来源：research_results 中对应 MEU 的 decision_summary.key_decisions

格式示例：
```markdown
| 决策 | 选择 | 理由 |
|------|------|------|
| 去重算法 | 滑动时间窗口 | 精确控制去重粒度，实现简单 |
```

**如果本 MEU 无关键技术决策**：保留段落标题，在下方注明"本 MEU 无关键技术决策，遵循项目统一技术选型"，不要省略段落。

#### {constraints}

内容：技术限制列表

来源：项目技术栈 + 评估阶段的设计决策

格式：使用无序列表，每条以 MUST/SHOULD/NEVER/MAY 开头

#### {output_requirements}

内容：必须交付的文件列表

来源：拆解阶段的 MEU 范围 + 评估阶段的产出建议

格式：使用编号列表

#### {acceptance_criteria}

内容：引用 benchmarks.yaml 中的具体条目，使用复选框格式

来源：评估阶段的 benchmarks.yaml

格式示例：
```markdown
- [ ] B3.1: GlobFilter 正确匹配 include/exclude 模式
- [ ] B3.2: 单元测试覆盖率 >= 90%
```

### 执行顺序建议

MUST 按照执行计划中生成的批次顺序执行 MEU。每个 MEU 的执行 Prompt MUST 在其所有依赖 MEU 完成后才可使用。

```
批次 1 → 执行 MEU-001 Prompt → 完成
批次 2 → 并行执行 MEU-002, MEU-004 Prompt → 完成
批次 3 → 执行 MEU-003 Prompt → 完成
  ...
```

### 研究价值传导检查清单

在填充完成后，应检查以下传导质量指标，确保研究阶段的信息无损传递给执行 AI：

| 检查项 | 通过标准 | 不通过的后果 |
|--------|---------|-------------|
| Research 中的每个 `key_decision` 都有对应的决策条目 | 100% 覆盖 | 执行 AI 可能推翻已评估的方案 |
| Research 中的关键 `pitfall` 都已写入 Prompt 避坑要点或 `framework/research.yaml` | 100% 覆盖 | 执行 AI 会重复踩坑 |
| `confidence=low` 的结论是否在避坑要点有额外提醒 | 全部提醒 | 执行 AI 可能过度依赖不确定的结论 |
| Prompt 中的上下文引用是否正确指向 `framework/` 中的对应条目 | 全部可解析 | 执行 AI 无法加载所需的详细数据 |
| `framework/` 中的结构化数据与 Prompt 中的摘要信息一致 | 无矛盾 | 执行 AI 收到冲突信息，影响判断 |

### 与其他阶段的关系

```
拆解阶段                    研究阶段                    评估阶段                    执行阶段
┌──────────┐           ┌──────────┐           ┌──────────┐           ┌──────────┐
│ 任务树    │           │ 研究报告  │           │ 评估报告  │           │ 执行 Prompt│
│          │           │          │           │          │           │          │
│ MEU 定义  │──────────→│ 研究结论  │           │ 方案选择  │           │          │
│ 模块信息  │           │ 踩坑记录  │──────────→│ 设计决策  │──────────→│ 8 段式结构 │
│ 批次信息  │           │ 决策摘要  │           │ 验收标准  │           │          │
│ 依赖关系  │           │ 开放问题  │           │ 产出建议  │           │          │
└──────────┘           └──────────┘           └──────────┘           └──────────┘
                                                    │                       │
                                                    │                       ▼
                                              ┌──────────┐           ┌──────────┐
                                              │          │           │ 执行产出  │
                                              │ 评估结论  │──────────→│ 代码文件  │
                                              │          │           │ 测试文件  │
                                              └──────────┘           │ 文档更新  │
                                                                     └──────────┘

信息传导路径：
  research.key_decisions      → 决策（第五段）+ framework/research.yaml
  research.pitfalls           → 避坑要点（第四段）+ framework/research.yaml
  research.caution_findings   → 避坑要点（第四段）+ framework/research.yaml
  research.approach_findings  → framework/research.yaml（通过上下文引用按需加载）
  research.open_questions     → 约束（第六段）+ 避坑要点（第四段）
  evaluation.acceptance       → framework/benchmarks.yaml（通过验收引用按需加载）
  evaluation.constraints      → 约束（第六段）
  evaluation.outputs          → 产出（第七段）
```
