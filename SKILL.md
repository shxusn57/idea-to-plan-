---
name: idea-to-action
description: >
  Transforms a vague idea into a structured, AI-executable plan through recursive decomposition,
  per-unit research, and benchmark-driven evaluation. Use this skill when the user has an idea,
  goal, or feature request that needs to be broken down into actionable steps, researched for
  blockers, and turned into an execution plan with measurable acceptance criteria. Typical
  triggers: "help me break this down", "research how to build this", "create an execution plan",
  or any request to turn a fuzzy concept into concrete deliverables.
---

# Idea-to-Action Plan

将一个模糊的想法，通过 AI 驱动的递归拆解 → 逐点研究 → Benchmark & Evaluation 定义，最终输出一份可直接交给 AI 执行的结构化计划。

## 前置条件

- Web 搜索能力（Step 2 研究阶段需要）
- 文件系统访问（执行阶段可能需要）

---

## Step 1: 递归拆解 [自由度: 中]

将用户的想法递归拆解为一棵最小可执行单元树。

> **执行本 Step 前，MUST 先读取**: [steps/step1-decompose.md](steps/step1-decompose.md)
> Prompt 模板: [templates/decompose-prompt.md](templates/decompose-prompt.md)
> 按需深入: [references/step1-node-reference.md](references/step1-node-reference.md)（节点字段详细参考）

### MEU 定义

叶子节点 MUST 同时满足以下 4 个条件：
- **单一职责** — 一个交付物
- **边界清晰** — 明确的输入/输出/范围
- **可独立执行** — 不需要隐含上下文
- **粒度适当** — 可在一次 AI 对话中完成（约 30 分钟 - 4 小时）

### 节点格式

```yaml
- id: "1.2.3"              # {父节点ID}.{序号}
  name: "动词 + 名词"        # 如 "实现用户认证"（MEU 必须动词开头）
  description: "..."         # 目标描述（等价于 objective）
  type: "task"              # task | decision | research | integration
                             # 叶子节点使用 type: leaf 标记为 MEU
  parent: "1.2"
  children: []
  dependencies: ["1.1.2"]   # 仅跨分支依赖；父子依赖隐含
  priority: "high"          # critical | high | medium | low
  estimated_complexity: "medium"  # trivial | low | medium | high | extreme
```

### 终止条件

满足以下**任意一条**时停止拆解：
- 所有 4 个 MEU 条件已满足
- 进一步拆解会产生无意义的碎片
- 信息不足无法继续（标记为待研究）

### Checklist

- [ ] 所有叶子节点满足 MEU 4 条件
- [ ] 树深度 3-5 层
- [ ] 无循环依赖
- [ ] 跨分支依赖已显式声明
- [ ] 包含非功能性任务（测试、文档、配置）

---

## Step 2: 多轮迭代研究 [自由度: 高]

对每个叶子节点，通过 4 轮迭代研究识别堵点、交叉验证方案、记录决策和踩坑信息。

> **执行本 Step 前，MUST 先读取**: [steps/step2-research.md](steps/step2-research.md)
> Prompt 模板: [templates/research-prompt.md](templates/research-prompt.md)
> 按需深入: [references/step2-blocker-types.md](references/step2-blocker-types.md)（堵点类型详解）、[references/step2-output-format.md](references/step2-output-format.md)（输出格式详解）、[references/step2-validation.md](references/step2-validation.md)（置信度与验证方法）

### 4 轮迭代流程

| 轮次 | 目标 | 关键要求 |
|------|------|---------|
| **第 1 轮** | 初始分析 | AI 推理识别堵点，制定搜索策略，输出 research_plan |
| **第 2 轮** | 定向搜索 | 每个堵点 >= 2 个独立信息源，记录来源类型和可信度 |
| **第 3 轮** | 交叉验证 | 多源结论对比，>= 2 个候选方案（含 pros/cons），关键 API 提供代码片段 |
| **第 4 轮** | 结论整合 | 记录决策过程（why A not B），生成踩坑记录和 decision_summary |

### 堵点分类

| 类型 | 说明 |
|------|------|
| **技术堵点** | 不确定 API 用法、参数或返回值 |
| **知识堵点** | 多个可行方案，尚未选择 |
| **工具堵点** | 不确定用什么工具/库 |
| **数据堵点** | 缺少必要的规格或示例 |
| **集成堵点** | 多组件间的兼容性不确定 |
| **性能堵点** | 不确定如何满足性能要求 |

### 核心规则

- 每个堵点：>= 2 个候选方案，附 pros/cons 和选择理由
- **CRITICAL**: 即使无堵点的 MEU 也 MUST 输出研究（`blockers: []`，`key_findings: >= 1`）
- **CRITICAL**: 每个 MEU MUST 有 `decision_summary`（决策摘要），包含 key_decisions 和 pitfalls_overview
- 研究策略：AI 推理 → 定向搜索 → 交叉验证 → 结论整合

### 输出格式

```yaml
- task_id: "1.2.3"
  decision_summary:
    key_decisions:
      - decision: "选择了什么"
        alternatives: ["放弃了什么"]
        reasoning: "为什么"
        trade_offs: "trade-off"
        confidence: "high"
    pitfalls_overview:
      - "踩坑记录1"
  blockers:
    - type: "技术堵点"
      description: "..."
      solutions:
        - name: "方案A"
          pros: ["优点"]
          cons: ["缺点"]
          code_snippet: "..."
      chosen_solution: "方案A"
      reasoning: "选择理由"
      pitfalls:
        - trap: "常见错误"
          correct_approach: "正确做法"
      status: "resolved"
  key_findings:
    - category: "approach"     # approach | api | pattern | caution
      content: "..."
  recommended_approach: "..."
  confidence: "high"
  open_questions: []
  references: []
```

### Checklist

- [ ] 每个叶子节点都完成了 4 轮研究
- [ ] 每个堵点 >= 2 个独立信息源
- [ ] 每个堵点 >= 2 个候选方案（含 pros/cons）
- [ ] 每个 MEU 有 decision_summary（含 key_decisions + pitfalls_overview）
- [ ] high 复杂度 MEU 有代码片段
- [ ] 置信度标注准确（有交叉验证支撑）

---

## Step 3: 定义评估标准 [自由度: 中]

基于研究结果，为每个 MEU 定义可量化的 Benchmark。

> **执行本 Step 前，MUST 先读取**: [steps/step3-evaluate.md](steps/step3-evaluate.md)
> Prompt 模板: [templates/evaluation-prompt.md](templates/evaluation-prompt.md)
> 按需深入: [references/step3-benchmark-types.md](references/step3-benchmark-types.md)（Benchmark 类型详解与模板）

### Benchmark 规则

- **所有 Benchmark 都必须达成** — 没有"可选"标准
- 每个 MEU：>= 2 个 Benchmark
- 每个 Benchmark MUST 可量化、可测试
- 每个 Benchmark MUST 追溯到 Step 2 的研究结论

### Benchmark 类型

| 类型 | 示例 |
|------|------|
| **文件存在** | "文件 `src/app.py` 存在" |
| **内容正确性** | "cli.py 定义了 `--path` 且 `required=True`" |
| **功能测试** | "`pytest tests/` 通过，覆盖率 >= 80%" |
| **质量标准** | "`ruff check src/` 零错误" |
| **集成验证** | "端到端：创建文件 → stdout 出现事件" |

### 禁止模式

NEVER 使用："功能正常"、"质量好"、"性能可接受"、"完整"。
INSTEAD 使用："响应 < 200ms"、"覆盖率 >= 80%"、"文件包含 X 行匹配模式 Y"。

### Checklist

- [ ] 每个 MEU 有 >= 2 个 Benchmark
- [ ] 所有 Benchmark 可量化
- [ ] 无模糊描述
- [ ] 每个 Benchmark 追溯到研究结论

---

## Step 4: 生成执行计划 [自由度: 低]

将 Step 1-3 的所有成果整合为一份结构化的 AI 可执行计划。

> **执行本 Step 前，MUST 先读取**: [steps/step4-plan.md](steps/step4-plan.md)
> Prompt 模板: [templates/execution-prompt.md](templates/execution-prompt.md)
> 按需深入: [references/step4-prompt-structure.md](references/step4-prompt-structure.md)（9 段式 Prompt 详细结构）、[references/step4-risk-management.md](references/step4-risk-management.md)（风险与开放问题管理）

### 执行顺序

- 基于 BFS 的拓扑排序（Kahn 算法）
- 同批次 MEU 可并行执行
- 检测并拒绝循环依赖

### 三层输出架构

Step 4 的产出 MUST 按三层分离架构组织，区分 AI 框架、人类报告和执行 Prompt：

```
{project-name}/
├── framework/                    # 层级 1：AI 引用的结构化数据
│   ├── task-tree.yaml            # 任务树（Step 1 产出）
│   ├── research.yaml             # 研究结论（Step 2 产出）
│   ├── benchmarks.yaml           # 验收标准（Step 3 产出）
│   └── dependency-graph.mermaid  # 依赖关系图
├── REPORT.md                     # 层级 2：人类阅读的项目报告
└── prompts/                      # 层级 3：给 AI 的短小执行指令
    ├── 1.1.1.md                  # 每个 MEU 一个独立 Prompt
    ├── 1.2.1.md
    └── ...
```

> **详细规范**: [references/output-architecture.md](references/output-architecture.md)

**核心原则**：
- **framework/** 存放结构化数据，AI 按需 Read 引用，不重复加载
- **REPORT.md** 面向人类，包含任务拆解、关键决策、风险、预期效果、进度追踪
- **prompts/** 每个 MEU 一个文件，~50-80 行，通过引用 framework/ 获取上下文（不内联）

### MEU 执行 Prompt 结构（精简 8 段式）

每个 Prompt 文件 MUST 包含以下 8 个段落（比原 9 段式精简，因为上下文通过引用获取）：

1. **背景** — 项目名称、批次、前置产出（1-2 句话）
2. **任务** — 目标描述（2-3 句话）
3. **上下文** — 引用指令（告诉 AI 去读 framework/ 的哪个文件的哪个条目）
4. **避坑要点** — 从 research 的 pitfalls 提取最关键的 3-5 条
5. **决策** — 从 research 的 decision_summary 提取关键决策表
6. **约束** — 技术限制列表
7. **产出** — 必须交付的文件
8. **验收** — 引用 benchmarks.yaml 中的具体条目

### Checklist

- [ ] framework/ 包含 4 个结构化文件
- [ ] REPORT.md 包含任务拆解、关键决策、风险、预期效果
- [ ] prompts/ 下每个 MEU 有独立 Prompt 文件（~50-80 行）
- [ ] Prompt 通过引用 framework/ 获取上下文（不内联）
- [ ] 避坑要点来自 research 的 pitfalls
- [ ] 决策来自 research 的 decision_summary
- [ ] 验收标准引用 benchmarks.yaml

---

## Benchmark 未通过时的处理

**CRITICAL**: 最多 3 轮重试，之后升级为人工审核。

| 重试轮次 | 处理方式 |
|---------|---------|
| 第 1 次 | 自动诊断原因，调整后重新执行 |
| 第 2 次 | 回溯到 Step 2，重新评估堵点和方案 |
| 第 3 次 | 标记 `reviewer: "human"`，等待人工介入 |

记录所有失败，追加 `failure_log`（原因 + 采取的调整措施）。

---

## Step 5: 质量评估 [自由度: 低]

生成计划后，运行两轮评估以确保质量。

> **执行本 Step 前，MUST 先读取**: [steps/step5-quality.md](steps/step5-quality.md)
> 按需深入: [references/step5-evaluation-detail.md](references/step5-evaluation-detail.md)（完整评估框架与 Prompt 模板）

### 评估 1: Prompt 执行合规性检查

验证最终输出是否准确执行了 4 个 Step 的 prompt 要求。

**23 个 Benchmark，覆盖 4 个 Step** — 每个检查特定的必填字段/格式/约束：

| Step | Benchmark | 检查内容 |
|------|-----------|---------|
| Step 1 | B1.1-B1.4 | 节点格式、MEU 合规性、依赖正确性、命名规范 |
| Step 2 | B2.1-B2.9 | 研究覆盖率、方案数量、选择完整性、无堵点处理、置信度、研究轮次、信息源、决策摘要、踩坑记录 |
| Step 3 | B3.1-B3.5 | Benchmark 数量、可测试性、量化率、研究追溯、评估完整性 |
| Step 4 | B4.1-B4.6 | 执行顺序、Prompt 覆盖、Prompt 结构、输出格式、避坑专区、决策日志 |

**方法**: LLM-as-judge，每个 Benchmark yes/no 判定。目标合规率 >= 90%。

### 评估 2: Plan 质量评估

评估计划是否真正可执行、研究是否充分。

**15 个 Benchmark，4 个维度** — 加权评分（1-5 分制）：

| 维度 | 权重 | Benchmark | 检查内容 |
|------|------|-----------|---------|
| 可执行性 | 30% | Q1.1-Q1.3 | 步骤具体性、依赖可行性、产出明确性 |
| 研究充分性 | 25% | Q2.1-Q2.4 | 堵点识别、方案可行性、分析深度、开放问题处理 |
| 验收标准质量 | 25% | Q3.1-Q3.4 | 可测试性、可量化性、维度覆盖、研究一致性 |
| 计划完整性 | 20% | Q4.1-Q4.4 | MEU 覆盖、上下文传递、风险识别、整体评估 |

**方法**: LLM-as-judge，Rubric 评分。目标加权平均 >= 4.0/5.0。

### 评估 Checklist

- [ ] 评估 1 合规率 >= 90%
- [ ] 评估 2 加权得分 >= 4.0/5.0
- [ ] 两轮评估都产出结构化报告
- [ ] 未通过的 Benchmark 有补救措施
- [ ] 评估结果已归档用于迭代追踪

---

## Critical Rules

- 所有 Benchmark 都必须达成 — 没有"可选"标准
- Benchmark MUST 可测试 — NEVER 使用模糊描述
- Benchmark MUST 追溯到研究结论
- 只有叶子节点（children: []）需要研究和 Benchmark
- 无堵点 MEU MUST NOT 跳过研究
- `dependencies` 字段：仅跨分支依赖（父子依赖隐含）
- 最多 3 轮重试，之后 MUST 升级为人工审核

## Common Pitfalls

- **过度拆解**：1-2 步能完成的任务 NEVER 继续拆
- **Benchmark 不可测试**："用户体验好"不是 Benchmark；"加载时间 < 3s"才是
- **忽略开放问题**：Step 2 的 open_questions MUST 在执行前解决
- **依赖遗漏**：跨分支依赖容易遗漏，拆解后 MUST 检查
- **研究不充分**：仅依赖 AI 推理不够 — MUST 用网络搜索验证
