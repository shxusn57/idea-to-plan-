# Step 2：技术研究 — 核心指南

> 本文件是 AI 执行 Step 2（技术研究）时首先加载的文件。包含流程概览、核心规则和输出骨架。
> 详细内容请按需加载对应的参考文件。

---

## 目录

- [四轮迭代流程概览](#四轮迭代流程概览)
- [堵点分类表](#堵点分类表)
- [核心规则](#核心规则)
- [输出格式骨架](#输出格式骨架)
- [Checklist](#checklist)
- [按需加载指令](#按需加载指令)

---

## 四轮迭代流程概览

```
第 1 轮：初始分析（AI 推理）
  │ 输出：堵点列表 + 研究计划 + 初始建议
  ▼
第 2 轮：定向搜索与信息收集
  │ 输出：每个堵点的多源信息 + 可信度评估
  ▼
第 3 轮：交叉验证与深度分析
  │ 输出：验证后的结论 + 方案对比（pros/cons）+ 代码片段
  ▼
第 4 轮：结论整合与决策记录
  │ 输出：决策记录 + 踩坑记录 + decision_summary
  ▼
最终研究结果（YAML 格式）
```

### 第 1 轮：初始分析（AI 推理）

- **目的**：利用 AI 自身知识储备，对 MEU 进行初步分析，识别知识盲区（堵点），生成研究计划
- **操作**：阅读 MEU 描述 → 分析候选方案（1-3 个） → 识别堵点 → 提供初始建议 → 生成研究计划
- **关键规则**：必须区分"有把握的建议"和"不确定的猜测"；完全不了解的领域标记为"需要网络搜索"；可能过时的知识标记为"需要验证时效性"

### 第 2 轮：定向搜索与信息收集

- **目的**：针对第 1 轮识别的堵点，通过定向搜索获取准确、最新的信息
- **操作**：对每个堵点执行定向搜索，记录信息源类型和可信度
- **关键规则**：每个堵点至少 2 个独立信息源；必须记录信息源类型；必须检查结果时效性（优先 2 年内）；不足则标记为 `unresolved`
- **详细查询模板和信息源分类** → 见 [step2-validation.md](references/step2-validation.md)

### 第 3 轮：交叉验证与深度分析

- **目的**：对第 2 轮信息进行交叉验证，对候选方案做 pros/cons 分析，提供最小可运行代码片段
- **操作**：交叉验证关键结论 → 方案对比分析（至少 2 个方案） → 代码片段准备 → 置信度评估
- **关键规则**：多源一致 → `high`；来源矛盾 → 进一步搜索或 `low`；单源支持 → `medium`
- **详细验证方法和信息源优先级** → 见 [step2-validation.md](references/step2-validation.md)

### 第 4 轮：结论整合与决策记录

- **目的**：整合所有轮次研究结果，记录决策过程和踩坑信息，生成决策摘要
- **操作**：汇总前 3 轮发现 → 记录"为什么选 A 不选 B" → 整理常见错误和避坑方法 → 创建 `decision_summary`

### 回溯机制

- 第 2 轮发现遗漏堵点 → 回到第 1 轮补充分析
- 第 3 轮发现信息有误 → 回到第 2 轮补充搜索
- 第 4 轮决策依据不足 → 回到第 2 或第 3 轮补充研究
- 每个堵点最多回溯 1 次；仍无法解决则标记为 `unresolved`

---

## 堵点分类表

堵点（Blocker）是指在执行 MEU 之前**必须解决**的知识盲区或技术不确定性。

| 类型 | 一句话说明 |
|---|---|
| **技术堵点** | 不确定应该使用哪个 API、其参数、返回值或行为细节 |
| **知识堵点** | 存在多个可行方案，尚未做出技术选型决策 |
| **工具堵点** | 不确定应该使用哪个工具、库或框架，或对所选工具的最佳实践不清楚 |
| **数据堵点** | 缺乏执行任务所需的具体数据或规格信息 |
| **集成堵点** | 不确定多个组件/系统之间的集成方式或兼容性 |
| **性能堵点** | 不确定如何实现特定业务逻辑或算法以满足性能要求 |

> **详细识别信号、典型示例和研究策略** → 见 [step2-blocker-types.md](references/step2-blocker-types.md)

---

## 核心规则

1. **MUST 区分事实与猜测**：绝不能将猜测当作确认的结论。有把握的建议和不确定的猜测必须明确区分。
2. **MUST 多源验证**：每个堵点至少从 2 个独立信息源获取信息，绝不依赖单一来源。
3. **MUST 记录信息源**：必须记录信息源类型（官方文档 / GitHub / 技术博客 / StackOverflow / 其他）和 URL。
4. **MUST 检查时效性**：优先选择最近 2 年内的内容，快速迭代的领域尤其要注意版本号。
5. **MUST 无堵点也输出 key_findings**：即使 `blockers` 为空，`key_findings` 也必须包含实质内容（实现要点、技术细节、注意事项、最佳实践）。
6. **NEVER 编造信息**：如果 AI 对某个领域完全不了解，直接标记为"需要网络搜索"，不要编造信息。
7. **NEVER 无限回溯**：每个堵点最多回溯 1 次，避免无限循环。回溯后仍无法解决的标记为 `unresolved`。

---

## 输出格式骨架

每个 MEU 的研究结果必须遵循以下 YAML 结构：

```yaml
research_results:
  - task_id: "1.2.3"                    # MEU 节点 ID
    task_name: "任务名称"

    decision_summary:                     # 研究决策摘要（有堵点必填，无堵点可为 {}）
      key_decisions: [...]                # 关键技术决策列表
      pitfalls_overview: [...]            # 踩坑概览
      confidence: "high"                  # 总体置信度

    blockers:                             # 堵点列表（可为空数组）
      - type: "技术堵点"                  # 6 种类型之一
        description: "..."                # 详细描述
        research_rounds: [...]            # 研究过程记录
        solutions: [...]                  # 候选方案（含 pros/cons）
        chosen_solution: "..."            # 最终选择
        reasoning: "..."                  # 选择理由
        pitfalls: [...]                   # 踩坑记录
        status: "resolved"                # resolved / partial / unresolved

    key_findings:                         # 关键发现（必须有内容）
      - category: "approach"              # approach / api / pattern / caution
        content: "..."

    recommended_approach: "..."           # 推荐方案（一段话总结）
    confidence: "high"                    # high / medium / low
    open_questions: [...]                 # 未解决的问题（可为空数组）
    references: [...]                     # 参考资料链接（可为空数组）
```

> **字段详细填写要求、子字段结构和填写示例** → 见 [step2-output-format.md](references/step2-output-format.md)

---

## Checklist

完成研究后，逐项检查：

- [ ] 每个 MEU 是否都经过了 4 轮迭代分析？
- [ ] 每个堵点是否至少有 2 个独立信息源？
- [ ] 信息源是否记录了类型和 URL？
- [ ] 搜索结果是否检查了时效性（优先 2 年内）？
- [ ] 堵点类型是否正确归类（6 种之一）？
- [ ] 每个堵点是否至少有 2 个候选方案的 pros/cons 分析？
- [ ] 是否记录了踩坑信息（pitfalls）？
- [ ] `decision_summary` 是否包含关键决策和 trade-off 分析？
- [ ] `key_findings` 是否有实质内容（即使无堵点）？
- [ ] `recommended_approach` 是否清晰、具体、可操作？
- [ ] `confidence` 是否与实际研究深度匹配？
- [ ] `open_questions` 中的问题是否都有 `impact` 和 `suggested_resolution`？
- [ ] `references` 是否包含关键信息源链接？

---

## 按需加载

> **按需加载**：
> - 遇到不确定的堵点类型时 → 读取 [step2-blocker-types.md](references/step2-blocker-types.md)
> - 需要了解输出字段的详细填写要求时 → 读取 [step2-output-format.md](references/step2-output-format.md)
> - 需要了解置信度定义或开放问题处理策略时 → 读取 [step2-validation.md](references/step2-validation.md)
