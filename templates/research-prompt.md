# 研究阶段 Prompt 模板

> **用途**: 对拆解后的 MEU 叶子节点进行多轮迭代深度研究
> **使用方法**: 将 `{task_tree_leaves}` 替换为 MEU 叶子节点列表，直接复制使用
> **输入**: MEU 叶子节点列表（来自拆解阶段输出）
> **输出**: 每个 MEU 的多轮研究记录、决策摘要、堵点分析

---

## 模板

将以下内容复制后，替换 `{task_tree_leaves}` 为实际的 MEU 叶子节点列表：

````
# 任务：对 MEU 任务树叶子节点进行多轮迭代深度研究

## 角色定义

你是一位资深技术研究员。你的职责是对每个最小执行单元（MEU）进行**多轮迭代**的深度技术调研，通过交叉验证确保研究结论的可靠性，并为后续的执行阶段提供充分的避坑指南和决策依据。

## 研究对象

以下是拆解阶段产出的 MEU 叶子节点列表：

{task_tree_leaves}

## 研究流程

### 第 1 轮：初始分析

对每个 MEU：
1. 利用 AI 知识进行初步分析
2. 识别堵点（6 种类型：技术/知识/工具/数据/集成/性能）
3. 为每个堵点制定搜索策略
4. 输出 research_plan（研究计划）

**输出格式**：
```yaml
research_plan:
  - task_id: "1.2.2"
    task_name: "..."
    ai_knowledge:
      confident: ["AI 有把握的结论"]
      uncertain: ["需要搜索验证的点"]
    planned_searches:
      - blocker: "堵点描述"
        query: "搜索查询语句"
        expected_sources: ["官方文档", "GitHub"]
    estimated_complexity: "medium"
```

### 第 2 轮：定向搜索与信息收集

对每个堵点：
1. 执行定向搜索（使用第 1 轮制定的搜索策略）
2. **每个堵点至少从 2 个独立信息源获取信息**
3. 记录信息源类型和关键发现
4. 做初步可信度评估

**输出格式**：
```yaml
search_results:
  - task_id: "1.2.2"
    findings:
      - blocker: "堵点描述"
        sources:
          - type: "官方文档"
            url/key_ref: "..."
            key_takeaway: "关键发现"
            reliability: "high"
          - type: "GitHub Issues"
            url/key_ref: "..."
            key_takeaway: "关键发现"
            reliability: "medium"
        preliminary_conclusion: "初步结论"
        confidence: "medium"  # high/medium/low
        needs_cross_validation: true
```

### 第 3 轮：交叉验证与深度分析

对第 2 轮收集的信息：
1. **交叉验证**：多个来源结论是否一致？
   - 一致 → 置信度提升为 high
   - 矛盾 → 进一步搜索或标记为 low confidence
2. **方案对比**：对每个堵点提供 >= 2 个候选方案，每个方案有 pros/cons
3. **代码验证**：对关键 API/方案提供最小可运行代码片段

**输出格式**：
```yaml
validated_findings:
  - task_id: "1.2.2"
    solutions:
      - blocker: "堵点描述"
        candidates:
          - name: "方案A"
            description: "..."
            pros: ["优点1", "优点2"]
            cons: ["缺点1"]
            feasibility: "high"
            code_snippet: |
              # 最小可运行代码
              ...
          - name: "方案B"
            description: "..."
            pros: ["优点1"]
            cons: ["缺点1", "缺点2"]
            feasibility: "medium"
        validation_result:
          sources_agree: true
          final_confidence: "high"
```

### 第 4 轮：结论整合与决策记录

整合所有轮次结果：
1. 为每个技术决策记录决策过程
2. 记录踩坑信息
3. 生成 decision_summary（决策摘要）

**最终输出格式**：
```yaml
research_results:
  - task_id: "1.2.2"
    task_name: "..."

    decision_summary:
      key_decisions:
        - decision: "选择了什么"
          alternatives: ["放弃了什么"]
          reasoning: "为什么"
          trade_offs: "trade-off 分析"
          confidence: "high"
      pitfalls_overview:
        - "踩坑记录1"
        - "踩坑记录2"

    blockers:
      - type: "技术堵点"
        description: "..."
        research_rounds:
          - round: 2
            action: "搜索了什么"
            sources: [...]
          - round: 3
            action: "交叉验证了什么"
            finding: "..."
        solutions:
          - name: "方案A"
            description: "..."
            pros: [...]
            cons: [...]
            feasibility: "high"
            code_snippet: |
              ...
        chosen_solution: "方案A"
        reasoning: "选择理由"
        pitfalls:
          - trap: "常见错误"
            correct_approach: "正确做法"
            source: "信息来源"
        status: "resolved"

    key_findings:
      - category: "approach|api|pattern|caution"
        content: "..."

    recommended_approach: |
      推荐方案描述

    confidence: "high"

    open_questions: []

    references:
      - "参考链接"
```

## 研究深度要求

| MEU 复杂度 | 第 2 轮搜索 | 第 3 轮验证 | 代码片段 |
|-----------|-----------|-----------|---------|
| low | 1-2 次搜索 | 1 个信息源确认 | 可选 |
| medium | 2-3 次搜索 | 2 个独立来源交叉验证 | 推荐 |
| high | 3-5 次搜索 | 3+ 来源交叉验证 | 必须 |

## 质量检查

在输出最终结果之前，MUST 逐项确认：

- [ ] 每个 MEU 都完成了 4 轮研究流程
- [ ] 每个堵点至少有 2 个独立信息源
- [ ] 每个堵点至少有 2 个候选方案（含 pros/cons）
- [ ] 关键决策有 decision_summary 记录
- [ ] 踩坑记录（pitfalls）不为空（至少 1 条）
- [ ] high 复杂度 MEU 有代码片段
- [ ] 置信度标注准确（有交叉验证支撑）
- [ ] 所有信息源类型已记录

## 约束条件

- MUST 使用中文输出
- MUST 基于实际可获取的技术信息，NEVER 编造
- MUST 区分"有把握的结论"和"不确定的猜测"
- MUST 为每个决策提供 reasoning
- MUST NOT 在研究中直接修改 MEU 定义
````

---

## 使用说明

### 如何使用此模板

1. **前置条件**：MUST 先完成拆解阶段，获得 MEU 任务树
2. **提取叶子节点**：从拆解输出中提取所有叶子节点 MEU（即没有子节点的 MEU）
3. **复制**：复制上方代码块中的全部内容
4. **替换占位符**：将 `{task_tree_leaves}` 替换为提取的 MEU 叶子节点列表
5. **粘贴执行**：将替换后的内容粘贴到 LLM 对话窗口中执行
6. **检查输出**：对照质量检查清单验证输出质量

### 与其他阶段的关系

```
拆解阶段                    研究阶段（4 轮迭代）                执行阶段
┌──────────┐              ┌──────────────────────┐            ┌──────────┐
│ MEU 任务树 │──叶子节点──→│ 第1轮: 初始分析       │            │          │
│ 依赖关系   │              │ 第2轮: 定向搜索       │──决策摘要──→│ 执行计划  │
│           │              │ 第3轮: 交叉验证       │──避坑指南──→│          │
│           │              │ 第4轮: 结论整合       │            │          │
└──────────┘              └──────────────────────┘            └──────────┘
```

### 填写 {task_tree_leaves} 的建议

`{task_tree_leaves}` SHOULD 直接从拆解阶段的输出中复制 MEU 明细表。格式示例：

```
{task_tree_leaves} =

| MEU ID | 名称 | 描述 | 依赖 | 所属模块 | 预估复杂度 |
|--------|------|------|------|----------|------------|
| MEU-001 | 设计 CLI 参数解析方案 | 设计命令行参数结构 | 无 | 项目基础 | low |
| MEU-002 | 搭建项目骨架和开发环境 | 创建项目目录结构、配置文件 | 无 | 项目基础 | low |
| MEU-003 | 实现文件系统监听核心 | 基于文件系统事件监听 | MEU-001, MEU-002 | 核心功能 | high |
| MEU-004 | 实现事件过滤和去重 | glob 模式过滤和时间窗口去重 | MEU-003 | 核心功能 | medium |
| MEU-005 | 实现输出格式化模块 | JSON 和表格输出格式 | MEU-001 | 核心功能 | low |
| MEU-006 | 编写单元测试 | 核心模块单元测试 | MEU-003, MEU-004, MEU-005 | 质量保障 | medium |
| MEU-007 | 集成测试与端到端验证 | 完整流程集成测试 | MEU-006 | 质量保障 | medium |
| MEU-008 | 编写用户文档和使用示例 | README 和使用指南 | MEU-007 | 交付 | low |
```
