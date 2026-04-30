# Step 5 评估详细参考

本文档是质量评估阶段的**详细参考手册**，包含每个 Benchmark 的详细定义、评分 Rubric、LLM-as-judge Prompt 模板、评估配置示例和评估流程。

> 本文档由 [step5-quality.md](../steps/step5-quality.md) 按需加载引用。

---

## 目录

- [1. 合规性评估 Benchmark 详细定义](#1-合规性评估-benchmark-详细定义)
  - [Step 1 拆解合规性](#step-1-拆解合规性)
  - [Step 2 研究合规性](#step-2-研究合规性)
  - [Step 3 评估合规性](#step-3-评估合规性)
  - [Step 4 计划合规性](#step-4-计划合规性)
- [2. 质量评估 Benchmark 详细评分 Rubric](#2-质量评估-benchmark-详细评分-rubric)
  - [合规性评估 Rubric（1-5 分）](#合规性评估-rubric1-5-分)
  - [质量评估 Rubric（1-5 分）](#质量评估-rubric1-5-分)
- [3. LLM-as-judge 评估 Prompt 模板](#3-llm-as-judge-评估-prompt-模板)
  - [合规性评估 Prompt](#合规性评估-prompt)
  - [质量评估 Prompt](#质量评估-prompt)
- [4. 评估配置 YAML 示例](#4-评估配置-yaml-示例)
- [5. 评估结果报告 YAML 格式示例](#5-评估结果报告-yaml-格式示例)
- [6. 评估流程与触发条件](#6-评估流程与触发条件)

---

## 1. 合规性评估 Benchmark 详细定义

### Step 1 拆解合规性

#### B1.1 节点格式合规率

- **检查内容**：任务树中所有节点是否包含全部必填字段
- **必填字段**：`id`、`name`、`objective/description`、`parent`、`children`、`dependencies`、`is_meu/type`
- **字段说明**：`description` 等价于 `objective`，`type: leaf` 等价于 `is_meu: true`
- **目标**：100%
- **判定方式**：逐一检查每个节点，缺少任一必填字段即为不合规

#### B1.2 MEU 判定合规率

- **检查内容**：所有叶子节点（`is_meu: true`）是否满足 4 个 MEU 条件
- **MEU 条件**：
  1. 目标单一且明确
  2. 可由单个 AI 会话独立完成
  3. 产出物可独立验证
  4. 不依赖未完成的其他 MEU（依赖已声明且可满足）
- **目标**：100%
- **判定方式**：对每个叶子节点逐一判断 4 个条件是否全部满足

#### B1.3 依赖关系正确率

- **检查内容**：
  - 跨分支依赖是否正确（引用的节点 ID 存在且确实属于不同分支）
  - 父子依赖是否未重复（子节点的 dependencies 中不应包含父节点 ID）
- **目标**：100%
- **判定方式**：遍历所有节点的 dependencies 字段进行验证

#### B1.4 命名规范合规率

- **检查内容**：所有叶子节点（MEU）名称符合"动词+名词"格式（>= 90%），根节点和分支节点不纳入命名规范检查（它们是组织性节点，不是执行单元）
- **示例**：`解析配置文件`、`实现文件监听`、`编写单元测试`
- **反例**：`配置文件`、`文件监听`、`测试`
- **目标**：>= 90%
- **判定方式**：仅检查叶子节点名称是否以动词开头

### Step 2 研究合规性

#### B2.1 研究覆盖率

- **检查内容**：所有叶子节点是否都有对应的研究结果
- **目标**：100%
- **判定方式**：对比任务树叶子节点与研究结果中的节点 ID，检查是否有遗漏

#### B2.2 堵点方案数量

- **检查内容**：每个识别出的堵点是否提供了 >= 2 个候选方案，且每个堵点是否有 pros/cons 分析
- **目标**：100%
- **判定方式**：遍历所有堵点，检查每个堵点是否提供了 >= 2 个候选方案（可以在 `resolution` 中列出多个方案，或使用 `solutions` 数组），并检查每个堵点是否有 pros/cons 分析

#### B2.3 方案选择完整性

- **检查内容**：每个堵点有明确的选择结果（`chosen_solution`/`resolution`）和选择理由（`reasoning`/分析过程），且每个堵点是否有 reasoning（选择理由）
- **目标**：100%
- **判定方式**：逐一检查每个堵点的方案选择记录，确认每个堵点均有 reasoning 字段或等效的选择理由

#### B2.4 无堵点节点处理

- **检查内容**：对于没有堵点的节点，是否记录了 `key_findings` 且数量 >= 1
- **目标**：100%
- **判定方式**：检查无堵点节点是否有 `key_findings` 字段且非空

#### B2.5 置信度标注率

- **检查内容**：所有研究结论是否标注了置信度等级
- **置信度等级**：`high`、`medium`、`low`
- **目标**：100%
- **判定方式**：检查每个节点的 `confidence` 字段是否存在且值合法

#### B2.6 研究轮次完整性

- **检查内容**：每个 MEU 是否完成了 4 轮研究流程（初始分析、定向搜索、交叉验证、结论整合）
- **目标**：100%
- **判定方式**：检查 research.yaml 中是否有 `research_rounds` 或等效的多轮研究记录

#### B2.7 信息源多样性

- **检查内容**：每个堵点是否有 >= 2 个独立信息源
- **目标**：100%
- **判定方式**：检查 research.yaml 中每个堵点的 `sources` 数组长度

#### B2.8 决策摘要完整性

- **检查内容**：每个 MEU 是否有 `decision_summary`（含 `key_decisions` 和 `pitfalls_overview`）
- **目标**：100%
- **判定方式**：检查 research.yaml 中每个 MEU 是否有 `decision_summary` 字段

#### B2.9 踩坑记录

- **检查内容**：有堵点的 MEU 是否记录了踩坑信息（`pitfalls`）
- **目标**：>= 80%
- **判定方式**：检查 research.yaml 中每个堵点是否有 `pitfalls` 数组

### Step 3 评估合规性

#### B3.1 Benchmark 覆盖率

- **检查内容**：每个 MEU 是否定义了 >= 2 个 Benchmark
- **目标**：100%
- **判定方式**：统计每个 MEU 的 `benchmarks` 数组长度

#### B3.2 Benchmark 可测试率

- **检查内容**：每个 Benchmark 是否有 `test_method`/`check` 字段（描述验证方法的字段）
- **目标**：100%
- **判定方式**：逐一检查每个 Benchmark 的 `test_method` 或 `check` 是否存在且非空

#### B3.3 Benchmark 量化率

- **检查内容**：每个 Benchmark 是否包含 `metric`/`threshold` 字段，或 `check` 字段中包含可量化标准（如数值、布尔判定、存在性检查）
- **目标**：>= 80%
- **判定方式**：统计包含量化指标的 Benchmark 占比（包括 `metric`/`threshold` 字段或 `check` 中的可量化描述）

#### B3.4 研究关联率

- **检查内容**：Benchmark 是否关联到研究阶段的结论
- **目标**：>= 80%
- **判定方式**：检查 Benchmark 的 `research_reference` 字段是否存在

#### B3.5 Evaluation 完整率

- **检查内容**：每个 MEU 是否有 `evaluation_steps`/`pass_criteria` 或等效的验收标准聚合描述
- **目标**：100%
- **判定方式**：逐一检查每个 MEU 的评估配置

### Step 4 计划合规性

#### B4.1 执行顺序完整性

- **检查内容**：执行计划是否包含基于依赖关系的拓扑排序
- **目标**：存在（Yes/No）
- **判定方式**：检查计划中是否有明确的执行顺序，且顺序符合依赖关系

#### B4.2 Prompt 覆盖率

- **检查内容**：每个 MEU 是否有对应的执行 Prompt
- **目标**：100%
- **判定方式**：对比任务树叶子节点与执行计划中的 Prompt 条目

#### B4.3 Prompt 结构完整性

- **检查内容**：每个执行 Prompt 是否包含 9 段式结构
- **9 段式结构**：
  1. 项目背景
  2. 任务描述
  3. 上下文信息
  4. 避坑专区
  5. 决策日志
  6. 分步执行指南
  7. 验收标准
  8. 约束条件
  9. 输出要求
- **目标**：100%
- **判定方式**：逐一检查每个执行 Prompt 是否包含全部 9 个段落

#### B4.4 三种格式输出

- **检查内容**：执行计划是否以三种格式输出
- **三种格式**：
  1. 完整执行计划文档（Markdown）
  2. Prompt 列表（可以是独立文件，也可以嵌入在执行计划文档中）
  3. 依赖关系图（Mermaid、Graphviz 或 ASCII 文本图）
- **目标**：存在（Yes/No）
- **判定方式**：检查输出中是否包含三种格式的文件

#### B4.5 避坑专区完整性

- **检查内容**：每个 MEU 的 Prompt 是否包含避坑专区段落
- **目标**：100%
- **判定方式**：检查 execution-plan.md 中每个 Prompt 是否有"避坑专区"标题

#### B4.6 决策日志完整性

- **检查内容**：每个 MEU 的 Prompt 是否包含决策日志段落
- **目标**：100%
- **判定方式**：检查 execution-plan.md 中每个 Prompt 是否有"决策日志"标题

---

## 2. 质量评估 Benchmark 详细评分 Rubric

### 合规性评估 Rubric（1-5 分）

以下 Rubric 用于对合规性评估的每个 Benchmark 进行细化评分。当 Yes/No 判定不够细致时，可使用此 Rubric 进行更精确的评估。

#### 5 分 — 完全合规

- 所有检查项 100% 符合要求
- 无任何遗漏或错误
- 格式规范、字段完整

#### 4 分 — 基本合规

- 合规率 >= 90%
- 存在少量非关键性问题（如个别命名不规范）
- 不影响整体执行

#### 3 分 — 部分合规

- 合规率 >= 70% 且 < 90%
- 存在多个非关键性问题或少量关键性问题
- 需要修复后才能正常执行

#### 2 分 — 大量不合规

- 合规率 >= 50% 且 < 70%
- 存在多个关键性问题
- 需要大幅修改

#### 1 分 — 严重不合规

- 合规率 < 50%
- 存在严重结构性问题
- 基本无法使用，需要重做

### 质量评估 Rubric（1-5 分）

以下 Rubric 适用于 Plan 质量评估中的每个 Benchmark。

#### 5 分 — 优秀

- 完全达到甚至超越目标
- 细节丰富，考虑周全
- 可直接用于生产环境执行
- 体现了深度思考和前瞻性

**示例**：
- 步骤具体性：95%+ 的步骤包含具体 action，包含工具/库版本、命令示例
- 堵点识别率：识别了所有关键堵点及多个次要堵点，每个都有详细分析

#### 4 分 — 良好

- 达到目标要求
- 细节充分，无明显遗漏
- 可用于执行，可能需要少量补充
- 体现了较好的思考

**示例**：
- 步骤具体性：90%+ 的步骤包含具体 action
- 堵点识别率：覆盖了 80%+ 的关键堵点

#### 3 分 — 合格

- 基本达到目标，但存在明显不足
- 有一定细节，但有关键遗漏
- 需要补充修改后才能执行
- 体现了基本思考

**示例**：
- 步骤具体性：70%-90% 的步骤包含具体 action
- 堵点识别率：覆盖了 60%-80% 的关键堵点

#### 2 分 — 不足

- 未达到目标
- 细节匮乏，遗漏较多
- 需要大量补充才能执行
- 思考不够深入

**示例**：
- 步骤具体性：50%-70% 的步骤包含具体 action
- 堵点识别率：仅覆盖 40%-60% 的关键堵点

#### 1 分 — 不可接受

- 远未达到目标
- 几乎没有有用的细节
- 无法用于执行
- 缺乏思考

**示例**：
- 步骤具体性：< 50% 的步骤包含具体 action
- 堵点识别率：< 40% 的关键堵点被识别

---

## 3. LLM-as-judge 评估 Prompt 模板

### 合规性评估 Prompt

```markdown
你是一个严格的合规性评估专家。你的任务是评估一份 Idea-to-Action 计划的各阶段输出是否符合 Prompt 指令要求。

## 评估对象

- 任务树文件：{task_tree_path}
- 研究结果文件：{research_path}
- 评估标准文件：{evaluation_path}
- 执行计划文件：{execution_plan_path}

## 评估方法

对以下 23 个 Benchmark 逐一进行 Yes/No 判定。判定时必须引用具体的输出内容作为依据。

## Benchmark 列表

### Step 1 拆解合规性

**B1.1 节点格式合规率**
- 检查：所有节点是否包含 id、name、objective/description、parent、children、dependencies、is_meu/type 全部必填字段（`description` 等价于 `objective`，`type: leaf` 等价于 `is_meu: true`）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B1.2 MEU 判定合规率**
- 检查：所有叶子节点是否满足 4 个 MEU 条件（目标单一、可独立完成、产出可验证、依赖可满足）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B1.3 依赖关系正确率**
- 检查：跨分支依赖引用正确，父子依赖未重复
- 判定：[ ] Yes [ ] No
- 依据：__________

**B1.4 命名规范合规率**
- 检查：所有叶子节点（MEU）名称符合"动词+名词"格式（>= 90%），根节点和分支节点不纳入检查
- 判定：[ ] Yes [ ] No
- 依据：__________

### Step 2 研究合规性

**B2.1 研究覆盖率**
- 检查：所有叶子节点都有研究结果
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.2 堵点方案数量**
- 检查：每个堵点 >= 2 个候选方案（可以在 resolution 中列出多个方案，或使用 solutions 数组），且每个堵点是否有 pros/cons 分析
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.3 方案选择完整性**
- 检查：每个堵点有明确的选择结果（chosen_solution/resolution）和选择理由（reasoning/分析过程），且每个堵点是否有 reasoning（选择理由）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.4 无堵点节点处理**
- 检查：无堵点节点有 key_findings 且 >= 1
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.5 置信度标注率**
- 检查：所有研究结论标注了置信度（high/medium/low）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.6 研究轮次完整性**
- 检查：每个 MEU 是否完成了 4 轮研究流程（初始分析、定向搜索、交叉验证、结论整合），检查 research.yaml 中是否有 research_rounds 或等效的多轮研究记录
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.7 信息源多样性**
- 检查：每个堵点是否有 >= 2 个独立信息源，检查 research.yaml 中每个堵点的 sources 数组长度
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.8 决策摘要完整性**
- 检查：每个 MEU 是否有 decision_summary（含 key_decisions 和 pitfalls_overview），检查 research.yaml 中每个 MEU 是否有 decision_summary 字段
- 判定：[ ] Yes [ ] No
- 依据：__________

**B2.9 踩坑记录**
- 检查：有堵点的 MEU 是否记录了踩坑信息（pitfalls），检查 research.yaml 中每个堵点是否有 pitfalls 数组
- 判定：[ ] Yes [ ] No
- 依据：__________

### Step 3 评估合规性

**B3.1 Benchmark 覆盖率**
- 检查：每个 MEU >= 2 个 Benchmark
- 判定：[ ] Yes [ ] No
- 依据：__________

**B3.2 Benchmark 可测试率**
- 检查：每个 Benchmark 有 test_method/check（描述验证方法的字段）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B3.3 Benchmark 量化率**
- 检查：>= 80% 的 Benchmark 有 metric/threshold 字段，或 check 字段中包含可量化标准（如数值、布尔判定、存在性检查）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B3.4 研究关联率**
- 检查：>= 80% 的 Benchmark 关联到研究结论
- 判定：[ ] Yes [ ] No
- 依据：__________

**B3.5 Evaluation 完整率**
- 检查：每个 MEU 有 evaluation_steps/pass_criteria 或等效的验收标准聚合描述
- 判定：[ ] Yes [ ] No
- 依据：__________

### Step 4 计划合规性

**B4.1 执行顺序完整性**
- 检查：存在基于依赖关系的拓扑排序
- 判定：[ ] Yes [ ] No
- 依据：__________

**B4.2 Prompt 覆盖率**
- 检查：每个 MEU 有执行 Prompt
- 判定：[ ] Yes [ ] No
- 依据：__________

**B4.3 Prompt 结构完整性**
- 检查：每个执行 Prompt 包含 9 段式结构（项目背景、任务描述、上下文信息、避坑专区、决策日志、分步执行指南、验收标准、约束条件、输出要求）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B4.4 三种格式输出**
- 检查：输出包含 Markdown 文档、Prompt 列表（独立文件或嵌入在文档中）、依赖关系图（Mermaid/Graphviz/ASCII）
- 判定：[ ] Yes [ ] No
- 依据：__________

**B4.5 避坑专区完整性**
- 检查：每个 MEU 的 Prompt 是否包含避坑专区段落，检查 execution-plan.md 中每个 Prompt 是否有"避坑专区"标题
- 判定：[ ] Yes [ ] No
- 依据：__________

**B4.6 决策日志完整性**
- 检查：每个 MEU 的 Prompt 是否包含决策日志段落，检查 execution-plan.md 中每个 Prompt 是否有"决策日志"标题
- 判定：[ ] Yes [ ] No
- 依据：__________

## 输出要求

请以 YAML 格式输出评估结果：

```yaml
compliance_evaluation:
  evaluator: "LLM-as-judge"
  timestamp: "{timestamp}"
  overall_result: "PASS|FAIL"
  compliance_rate: {pass_count}/23
  benchmarks:
    B1.1:
      name: "节点格式合规率"
      result: "Yes|No"
      evidence: "具体依据..."
    # ... 其余 Benchmark 同上
  summary:
    passed: [{passed_ids}]
    failed: [{failed_ids}]
    recommendations: ["改进建议..."]
```
```

### 质量评估 Prompt

```markdown
你是一个资深的计划质量评估专家。你的任务是从可执行性、研究充分性、验收标准质量、计划完整性四个维度，对一份 Idea-to-Action 执行计划进行深度质量评估。

## 评估对象

- 任务树文件：{task_tree_path}
- 研究结果文件：{research_path}
- 评估标准文件：{evaluation_path}
- 执行计划文件：{execution_plan_path}

## 评估维度与 Benchmark

### 维度 1：可执行性（权重 30%）

**Q1.1 步骤具体性**（目标 >= 90%）
- 评分标准：
  - 5 分：>= 95% 的步骤包含具体 action
  - 4 分：>= 90% 的步骤包含具体 action
  - 3 分：>= 75% 的步骤包含具体 action
  - 2 分：>= 60% 的步骤包含具体 action
  - 1 分：< 60% 的步骤包含具体 action
- 评分：___ 分
- 依据：__________

**Q1.2 依赖可行性**（目标 = 0 违反）
- 评分标准：
  - 5 分：0 次依赖违反
  - 4 分：0 次依赖违反
  - 3 分：1 次依赖违反
  - 2 分：2-3 次依赖违反
  - 1 分：> 3 次依赖违反
- 评分：___ 分
- 依据：__________

**Q1.3 产出明确性**（目标 >= 90%）
- 评分标准：
  - 5 分：>= 95% 的 MEU 有明确产出
  - 4 分：>= 90% 的 MEU 有明确产出
  - 3 分：>= 75% 的 MEU 有明确产出
  - 2 分：>= 60% 的 MEU 有明确产出
  - 1 分：< 60% 的 MEU 有明确产出
- 评分：___ 分
- 依据：__________

### 维度 2：研究充分性（权重 25%）

**Q2.1 堵点识别率**（目标 >= 80%）
- 评分标准：
  - 5 分：识别了所有关键堵点及次要堵点
  - 4 分：覆盖 >= 80% 的关键堵点
  - 3 分：覆盖 >= 60% 的关键堵点
  - 2 分：覆盖 >= 40% 的关键堵点
  - 1 分：< 40% 的关键堵点被识别
- 评分：___ 分
- 依据：__________

**Q2.2 方案可行性**（目标 >= 80%）
- 评分标准：
  - 5 分：>= 95% 的方案可行
  - 4 分：>= 80% 的方案可行
  - 3 分：>= 65% 的方案可行
  - 2 分：>= 50% 的方案可行
  - 1 分：< 50% 的方案可行
- 评分：___ 分
- 依据：__________

**Q2.3 研究深度**（目标 >= 90%）
- 评分标准：
  - 5 分：每个堵点有 >= 2 个方案（含 pros/cons + 代码片段），有交叉验证记录，有 decision_summary
  - 4 分：每个堵点有 >= 2 个方案（含 pros/cons），有交叉验证记录
  - 3 分：每个堵点有 >= 2 个方案（含 pros/cons）
  - 2 分：每个堵点有方案但无 pros/cons
  - 1 分：堵点无方案或无研究
- 评分：___ 分
- 依据：__________

**Q2.4 开放问题处理**（目标 >= 80%）
- 评分标准：
  - 5 分：所有开放问题都有明确处理策略
  - 4 分：>= 80% 的开放问题有处理策略
  - 3 分：>= 65% 的开放问题有处理策略
  - 2 分：>= 50% 的开放问题有处理策略
  - 1 分：< 50% 的开放问题有处理策略
- 评分：___ 分
- 依据：__________

### 维度 3：验收标准质量（权重 25%）

**Q3.1 可测试性**（目标 100%）
- 评分标准：
  - 5 分：100% 的 Benchmark 有 test_method/check
  - 4 分：>= 95% 的 Benchmark 有 test_method/check
  - 3 分：>= 85% 的 Benchmark 有 test_method/check
  - 2 分：>= 70% 的 Benchmark 有 test_method/check
  - 1 分：< 70% 的 Benchmark 有 test_method/check
- 评分：___ 分
- 依据：__________

**Q3.2 可量化性**（目标 >= 80%）
- 评分标准：
  - 5 分：>= 95% 的 Benchmark 有量化指标（metric/threshold 字段或 check 中的可量化描述）
  - 4 分：>= 80% 的 Benchmark 有量化指标
  - 3 分：>= 65% 的 Benchmark 有量化指标
  - 2 分：>= 50% 的 Benchmark 有量化指标
  - 1 分：< 50% 的 Benchmark 有量化指标
- 评分：___ 分
- 依据：__________

**Q3.3 覆盖全面性**（目标 >= 3 维度）
- 评分标准：
  - 5 分：每个 MEU 平均 >= 4 个质量维度
  - 4 分：每个 MEU 平均 >= 3 个质量维度
  - 3 分：每个 MEU 平均 >= 2 个质量维度
  - 2 分：每个 MEU 平均 >= 1.5 个质量维度
  - 1 分：每个 MEU 平均 < 1.5 个质量维度
- 评分：___ 分
- 依据：__________

**Q3.4 与研究一致性**（目标 >= 80%）
- 评分标准：
  - 5 分：>= 95% 的 Benchmark 与研究结论一致
  - 4 分：>= 80% 的 Benchmark 与研究结论一致
  - 3 分：>= 65% 的 Benchmark 与研究结论一致
  - 2 分：>= 50% 的 Benchmark 与研究结论一致
  - 1 分：< 50% 的 Benchmark 与研究结论一致
- 评分：___ 分
- 依据：__________

### 维度 4：计划完整性（权重 20%）

**Q4.1 MEU 覆盖率**（目标 100%）
- 评分标准：
  - 5 分：100% MEU 被覆盖
  - 4 分：>= 95% MEU 被覆盖
  - 3 分：>= 85% MEU 被覆盖
  - 2 分：>= 70% MEU 被覆盖
  - 1 分：< 70% MEU 被覆盖
- 评分：___ 分
- 依据：__________

**Q4.2 上下文传递**（目标 >= 90%）
- 评分标准：
  - 5 分：9 段式完整，避坑专区和决策日志内容丰富，来自 research 的 decision_summary 和 pitfalls
  - 4 分：9 段式完整，避坑专区和决策日志有内容但不够详细
  - 3 分：9 段式完整但避坑专区/决策日志内容空洞
  - 2 分：7 段式（缺少避坑专区或决策日志）
  - 1 分：严重缺失段落
- 评分：___ 分
- 依据：__________

**Q4.3 风险识别**（目标 >= 3 条）
- 评分标准：
  - 5 分：>= 5 条风险，每条有缓解措施
  - 4 分：>= 3 条风险，每条有缓解措施
  - 3 分：>= 3 条风险，部分有缓解措施
  - 2 分：1-2 条风险
  - 1 分：无风险识别
- 评分：___ 分
- 依据：__________

**Q4.4 整体评估**（目标：存在）
- 评分标准：
  - 5 分：整体评估全面，包含总结、关键决策、预期产出、里程碑
  - 4 分：整体评估完整，包含总结和关键决策
  - 3 分：有基本总结
  - 2 分：整体评估不完整
  - 1 分：无整体评估
- 评分：___ 分
- 依据：__________

## 输出要求

请以 YAML 格式输出评估结果：

```yaml
quality_evaluation:
  evaluator: "LLM-as-judge"
  timestamp: "{timestamp}"
  overall_result: "PASS|FAIL"
  weighted_average: {score}/5.0
  dimensions:
    executability:
      weight: 0.30
      benchmarks:
        Q1.1:
          name: "步骤具体性"
          score: {1-5}
          evidence: "具体依据..."
        Q1.2:
          name: "依赖可行性"
          score: {1-5}
          evidence: "具体依据..."
        Q1.3:
          name: "产出明确性"
          score: {1-5}
          evidence: "具体依据..."
      average_score: {avg}/5.0
    research_thoroughness:
      weight: 0.25
      benchmarks:
        Q2.1:
          name: "堵点识别率"
          score: {1-5}
          evidence: "具体依据..."
        Q2.2:
          name: "方案可行性"
          score: {1-5}
          evidence: "具体依据..."
        Q2.3:
          name: "研究深度"
          score: {1-5}
          evidence: "具体依据..."
        Q2.4:
          name: "开放问题处理"
          score: {1-5}
          evidence: "具体依据..."
      average_score: {avg}/5.0
    acceptance_quality:
      weight: 0.25
      benchmarks:
        Q3.1:
          name: "可测试性"
          score: {1-5}
          evidence: "具体依据..."
        Q3.2:
          name: "可量化性"
          score: {1-5}
          evidence: "具体依据..."
        Q3.3:
          name: "覆盖全面性"
          score: {1-5}
          evidence: "具体依据..."
        Q3.4:
          name: "与研究一致性"
          score: {1-5}
          evidence: "具体依据..."
      average_score: {avg}/5.0
    plan_completeness:
      weight: 0.20
      benchmarks:
        Q4.1:
          name: "MEU 覆盖率"
          score: {1-5}
          evidence: "具体依据..."
        Q4.2:
          name: "上下文传递"
          score: {1-5}
          evidence: "具体依据..."
        Q4.3:
          name: "风险识别"
          score: {1-5}
          evidence: "具体依据..."
        Q4.4:
          name: "整体评估"
          score: {1-5}
          evidence: "具体依据..."
      average_score: {avg}/5.0
  summary:
    strengths: ["优势..."]
    weaknesses: ["不足..."]
    recommendations: ["改进建议..."]
```
```

---

## 4. 评估配置 YAML 示例

```yaml
# evaluation-config.yaml
# Idea-to-Action 评估配置

evaluation:
  name: "Idea-to-Action Plan 评估"
  version: "1.0"
  description: "对 Idea-to-Action Skill 的输出进行质量评估"

  # 评估触发条件
  triggers:
    - after_decomposition: true      # 拆解完成后触发 Step 1 合规性评估
    - after_research: true           # 研究完成后触发 Step 2 合规性评估
    - after_evaluation: true         # 评估标准完成后触发 Step 3 合规性评估
    - after_plan_generation: true    # 计划生成完成后触发 Step 4 合规性评估 + 质量评估
    - on_demand: true                # 支持手动触发

  # 评估体系配置
  frameworks:

    # 体系 1：合规性评估
    compliance:
      enabled: true
      method: "llm_as_judge"
      judge_model: "gpt-4o"
      pass_threshold: 0.90           # 90% 合规率通过
      benchmarks:
        - id: "B1.1"
          name: "节点格式合规率"
          step: 1
          target: 1.0
          required_fields:
            - id
            - name
            - objective/description
            - parent
            - children
            - dependencies
            - is_meu/type
          field_aliases:
            - description: objective
            - "type: leaf": "is_meu: true"

        - id: "B1.2"
          name: "MEU 判定合规率"
          step: 1
          target: 1.0
          meu_conditions:
            - "目标单一且明确"
            - "可由单个 AI 会话独立完成"
            - "产出物可独立验证"
            - "不依赖未完成的其他 MEU"

        - id: "B1.3"
          name: "依赖关系正确率"
          step: 1
          target: 1.0

        - id: "B1.4"
          name: "命名规范合规率"
          step: 1
          target: 0.90
          pattern: "动词+名词"
          scope: "仅叶子节点（MEU），根节点和分支节点不纳入检查"

        - id: "B2.1"
          name: "研究覆盖率"
          step: 2
          target: 1.0

        - id: "B2.2"
          name: "堵点方案数量"
          step: 2
          target: 1.0
          min_solutions: 2
          check_note: "可以在 resolution 中列出多个方案，或使用 solutions 数组"
          additional_check: "每个堵点是否有 pros/cons 分析"

        - id: "B2.3"
          name: "方案选择完整性"
          step: 2
          target: 1.0
          required_fields:
            - chosen_solution/resolution
            - reasoning/分析过程
          additional_check: "每个堵点是否有 reasoning（选择理由）"

        - id: "B2.4"
          name: "无堵点节点处理"
          step: 2
          target: 1.0
          min_key_findings: 1

        - id: "B2.5"
          name: "置信度标注率"
          step: 2
          target: 1.0
          valid_values:
            - high
            - medium
            - low

        - id: "B2.6"
          name: "研究轮次完整性"
          step: 2
          target: 1.0
          check_note: "每个 MEU 是否完成了 4 轮研究流程（初始分析、定向搜索、交叉验证、结论整合），检查 research.yaml 中是否有 research_rounds 或等效的多轮研究记录"

        - id: "B2.7"
          name: "信息源多样性"
          step: 2
          target: 1.0
          min_sources: 2
          check_note: "检查 research.yaml 中每个堵点的 sources 数组长度"

        - id: "B2.8"
          name: "决策摘要完整性"
          step: 2
          target: 1.0
          required_fields:
            - decision_summary
          check_note: "每个 MEU 是否有 decision_summary（含 key_decisions 和 pitfalls_overview）"

        - id: "B2.9"
          name: "踩坑记录"
          step: 2
          target: 0.80
          check_note: "有堵点的 MEU 是否记录了踩坑信息（pitfalls），检查 research.yaml 中每个堵点是否有 pitfalls 数组"

        - id: "B3.1"
          name: "Benchmark 覆盖率"
          step: 3
          target: 1.0
          min_benchmarks_per_meu: 2

        - id: "B3.2"
          name: "Benchmark 可测试率"
          step: 3
          target: 1.0
          check_fields:
            - test_method
            - check

        - id: "B3.3"
          name: "Benchmark 量化率"
          step: 3
          target: 0.80
          check_note: "metric/threshold 字段，或 check 字段中包含可量化标准（如数值、布尔判定、存在性检查）"

        - id: "B3.4"
          name: "研究关联率"
          step: 3
          target: 0.80

        - id: "B3.5"
          name: "Evaluation 完整率"
          step: 3
          target: 1.0
          required_fields:
            - evaluation_steps/pass_criteria
            - 等效的验收标准聚合描述

        - id: "B4.1"
          name: "执行顺序完整性"
          step: 4
          target: "exists"

        - id: "B4.2"
          name: "Prompt 覆盖率"
          step: 4
          target: 1.0

        - id: "B4.3"
          name: "Prompt 结构完整性"
          step: 4
          target: 1.0
          required_sections: 9
          section_names:
            - 项目背景
            - 任务描述
            - 上下文信息
            - 避坑专区
            - 决策日志
            - 分步执行指南
            - 验收标准
            - 约束条件
            - 输出要求

        - id: "B4.4"
          name: "三种格式输出"
          step: 4
          target: "exists"
          formats:
            - markdown
            - prompt_list  # 可以是独立文件，也可以嵌入在执行计划文档中
            - dependency_graph  # Mermaid、Graphviz 或 ASCII 文本图

        - id: "B4.5"
          name: "避坑专区完整性"
          step: 4
          target: 1.0
          check_note: "每个 MEU 的 Prompt 是否包含避坑专区段落，检查 execution-plan.md 中每个 Prompt 是否有'避坑专区'标题"

        - id: "B4.6"
          name: "决策日志完整性"
          step: 4
          target: 1.0
          check_note: "每个 MEU 的 Prompt 是否包含决策日志段落，检查 execution-plan.md 中每个 Prompt 是否有'决策日志'标题"

    # 体系 2：质量评估
    quality:
      enabled: true
      method: "llm_as_judge"
      judge_model: "gpt-4o"
      pass_threshold: 4.0             # 加权平均 4.0/5.0 通过
      dimensions:
        - id: "executability"
          name: "可执行性"
          weight: 0.30
          benchmarks:
            - id: "Q1.1"
              name: "步骤具体性"
              target: 0.90
            - id: "Q1.2"
              name: "依赖可行性"
              target: 0
            - id: "Q1.3"
              name: "产出明确性"
              target: 0.90

        - id: "research_thoroughness"
          name: "研究充分性"
          weight: 0.25
          benchmarks:
            - id: "Q2.1"
              name: "堵点识别率"
              target: 0.80
            - id: "Q2.2"
              name: "方案可行性"
              target: 0.80
            - id: "Q2.3"
              name: "研究深度"
              target: 0.90
            - id: "Q2.4"
              name: "开放问题处理"
              target: 0.80

        - id: "acceptance_quality"
          name: "验收标准质量"
          weight: 0.25
          benchmarks:
            - id: "Q3.1"
              name: "可测试性"
              target: 1.0
            - id: "Q3.2"
              name: "可量化性"
              target: 0.80
            - id: "Q3.3"
              name: "覆盖全面性"
              target: 3
            - id: "Q3.4"
              name: "与研究一致性"
              target: 0.80

        - id: "plan_completeness"
          name: "计划完整性"
          weight: 0.20
          benchmarks:
            - id: "Q4.1"
              name: "MEU 覆盖率"
              target: 1.0
            - id: "Q4.2"
              name: "上下文传递"
              target: 0.90
            - id: "Q4.3"
              name: "风险识别"
              target: 3
            - id: "Q4.4"
              name: "整体评估"
              target: "exists"

  # 评估行为配置
  behavior:
    auto_fix_on_fail: false           # 评估失败时是否自动修复
    max_retry: 2                      # 最大重试次数
    verbose_output: true              # 是否输出详细评估依据
    save_intermediate: true           # 是否保存中间评估结果
```

---

## 5. 评估结果报告 YAML 格式示例

```yaml
# evaluation-report.yaml
# Idea-to-Action 评估结果报告

report:
  id: "eval-20260428-001"
  timestamp: "2026-04-28T10:30:00Z"
  project: "filewatch-cli"
  evaluator: "LLM-as-judge (GPT-4o)"
  evaluation_config: "evaluation-config.yaml"

  # 输入文件
  inputs:
    task_tree: "examples/filewatch-cli/task-tree.yaml"
    research: "examples/filewatch-cli/research.yaml"
    evaluation: "examples/filewatch-cli/evaluation.yaml"
    execution_plan: "examples/filewatch-cli/execution-plan.md"

  # ============================================================
  # 评估体系 1：合规性评估结果
  # ============================================================
  compliance_evaluation:
    overall_result: "PASS"
    compliance_rate: "22/23 (95.7%)"

    benchmarks:
      B1.1:
        name: "节点格式合规率"
        result: "Yes"
        evidence: "检查了全部 12 个节点，均包含全部必填字段（objective/description、is_meu/type 等效字段均接受）"
      B1.2:
        name: "MEU 判定合规率"
        result: "Yes"
        evidence: "7 个叶子节点均满足 4 个 MEU 条件"
      B1.3:
        name: "依赖关系正确率"
        result: "Yes"
        evidence: "3 个跨分支依赖引用正确，无父子依赖重复"
      B1.4:
        name: "命名规范合规率"
        result: "No"
        evidence: "7 个叶子节点中 6 个符合'动词+名词'格式（85.7%），未达 90% 目标。根节点和分支节点不纳入检查。不符合的叶子节点：'配置管理'、'日志系统'"
      B2.1:
        name: "研究覆盖率"
        result: "Yes"
        evidence: "7 个叶子节点均有对应研究结果"
      B2.2:
        name: "堵点方案数量"
        result: "Yes"
        evidence: "5 个堵点均有 >= 2 个候选方案（通过 resolution 或 solutions 数组提供），且每个堵点均有 pros/cons 分析"
      B2.3:
        name: "方案选择完整性"
        result: "Yes"
        evidence: "5 个堵点均有明确的选择结果（chosen_solution/resolution）和选择理由（reasoning/分析过程），每个堵点均有 reasoning 字段"
      B2.4:
        name: "无堵点节点处理"
        result: "Yes"
        evidence: "2 个无堵点节点均有 key_findings（分别 2 条和 3 条）"
      B2.5:
        name: "置信度标注率"
        result: "Yes"
        evidence: "7 个节点均标注了置信度（4 high, 2 medium, 1 low）"
      B2.6:
        name: "研究轮次完整性"
        result: "Yes"
        evidence: "7 个 MEU 均完成了 4 轮研究流程，research.yaml 中均有 research_rounds 记录"
      B2.7:
        name: "信息源多样性"
        result: "Yes"
        evidence: "5 个堵点均有 >= 2 个独立信息源（sources 数组长度均 >= 2）"
      B2.8:
        name: "决策摘要完整性"
        result: "Yes"
        evidence: "7 个 MEU 均有 decision_summary 字段，含 key_decisions 和 pitfalls_overview"
      B2.9:
        name: "踩坑记录"
        result: "Yes"
        evidence: "5 个有堵点的 MEU 均记录了 pitfalls 数组"
      B3.1:
        name: "Benchmark 覆盖率"
        result: "Yes"
        evidence: "7 个 MEU 各有 2-3 个 Benchmark"
      B3.2:
        name: "Benchmark 可测试率"
        result: "Yes"
        evidence: "全部 18 个 Benchmark 均有 test_method 或 check 字段"
      B3.3:
        name: "Benchmark 量化率"
        result: "Yes"
        evidence: "18 个 Benchmark 中 16 个有 metric/threshold 字段或 check 中包含可量化标准（88.9%）"
      B3.4:
        name: "研究关联率"
        result: "Yes"
        evidence: "18 个 Benchmark 中 15 个有 research_reference（83.3%）"
      B3.5:
        name: "Evaluation 完整率"
        result: "Yes"
        evidence: "7 个 MEU 均有 evaluation_steps/pass_criteria 或等效的验收标准聚合描述"
      B4.1:
        name: "执行顺序完整性"
        result: "Yes"
        evidence: "执行计划包含基于依赖关系的拓扑排序，共 7 步"
      B4.2:
        name: "Prompt 覆盖率"
        result: "Yes"
        evidence: "7 个 MEU 均有对应执行 Prompt"
      B4.3:
        name: "Prompt 结构完整性"
        result: "Yes"
        evidence: "7 个执行 Prompt 均包含 9 段式结构（项目背景、任务描述、上下文信息、避坑专区、决策日志、分步执行指南、验收标准、约束条件、输出要求）"
      B4.4:
        name: "三种格式输出"
        result: "Yes"
        evidence: "输出包含 execution-plan.md、嵌入在文档中的 Prompt 列表、dependency-graph.mermaid"
      B4.5:
        name: "避坑专区完整性"
        result: "Yes"
        evidence: "7 个 MEU 的 Prompt 均包含'避坑专区'标题段落"
      B4.6:
        name: "决策日志完整性"
        result: "Yes"
        evidence: "7 个 MEU 的 Prompt 均包含'决策日志'标题段落"

    summary:
      passed:
        - B1.1
        - B1.2
        - B1.3
        - B2.1
        - B2.2
        - B2.3
        - B2.4
        - B2.5
        - B2.6
        - B2.7
        - B2.8
        - B2.9
        - B3.1
        - B3.2
        - B3.3
        - B3.4
        - B3.5
        - B4.1
        - B4.2
        - B4.3
        - B4.4
        - B4.5
        - B4.6
      failed:
        - B1.4
      recommendations:
        - "将节点'配置管理'改为'管理项目配置'，'日志系统'改为'实现日志系统'"

  # ============================================================
  # 评估体系 2：质量评估结果
  # ============================================================
  quality_evaluation:
    overall_result: "PASS"
    weighted_average: 4.2/5.0

    dimensions:
      executability:
        name: "可执行性"
        weight: 0.30
        weighted_score: 1.26
        benchmarks:
          Q1.1:
            name: "步骤具体性"
            score: 4
            evidence: "约 92% 的步骤包含具体 action，包含库版本和命令示例"
          Q1.2:
            name: "依赖可行性"
            score: 5
            evidence: "0 次依赖违反，拓扑排序正确"
          Q1.3:
            name: "产出明确性"
            score: 4
            evidence: "约 93% 的 MEU 有明确产出描述"
        average_score: 4.3/5.0

      research_thoroughness:
        name: "研究充分性"
        weight: 0.25
        weighted_score: 1.05
        benchmarks:
          Q2.1:
            name: "堵点识别率"
            score: 4
            evidence: "识别了 5 个关键堵点中的 4 个（80%），遗漏了跨平台兼容性"
          Q2.2:
            name: "方案可行性"
            score: 4
            evidence: "12 个方案中 10 个可行（83.3%）"
          Q2.3:
            name: "研究深度"
            score: 5
            evidence: "每个堵点有 >= 2 个方案（含 pros/cons + 代码片段），有交叉验证记录，有 decision_summary"
          Q2.4:
            name: "开放问题处理"
            score: 3
            evidence: "3 个开放问题中 2 个有处理策略（66.7%）"
        average_score: 4.0/5.0

      acceptance_quality:
        name: "验收标准质量"
        weight: 0.25
        weighted_score: 1.0
        benchmarks:
          Q3.1:
            name: "可测试性"
            score: 5
            evidence: "全部 18 个 Benchmark 均有 test_method 或 check 字段"
          Q3.2:
            name: "可量化性"
            score: 4
            evidence: "18 个 Benchmark 中 16 个有量化指标（88.9%）"
          Q3.3:
            name: "覆盖全面性"
            score: 4
            evidence: "每个 MEU 平均 3.2 个质量维度"
          Q3.4:
            name: "与研究一致性"
            score: 4
            evidence: "18 个 Benchmark 中 15 个与研究结论一致（83.3%）"
        average_score: 4.3/5.0

      plan_completeness:
        name: "计划完整性"
        weight: 0.20
        weighted_score: 0.89
        benchmarks:
          Q4.1:
            name: "MEU 覆盖率"
            score: 5
            evidence: "7 个 MEU 全部被覆盖"
          Q4.2:
            name: "上下文传递"
            score: 4
            evidence: "9 段式完整，避坑专区和决策日志有内容但不够详细"
          Q4.3:
            name: "风险识别"
            score: 4
            evidence: "识别了 4 条风险，均有缓解措施"
          Q4.4:
            name: "整体评估"
            score: 5
            evidence: "包含完整的总结、关键决策、预期产出和里程碑"
        average_score: 4.5/5.0

    summary:
      strengths:
        - "依赖关系设计合理，无违反"
        - "研究深度出色，所有方案都有优缺点分析，有交叉验证和决策摘要"
        - "验收标准可测试性强"
        - "风险识别全面"
        - "避坑专区和决策日志完整覆盖"
      weaknesses:
        - "个别节点命名不规范（B1.4）"
        - "开放问题处理不够充分（Q2.4）"
        - "避坑专区和决策日志内容可进一步丰富（Q4.2）"
      recommendations:
        - "修正不符合命名规范的节点名称"
        - "为剩余开放问题补充处理策略"
        - "丰富避坑专区和决策日志的内容，补充更多来自 research 的 decision_summary 和 pitfalls 信息"

  # ============================================================
  # 总体结论
  # ============================================================
  final_verdict:
    result: "PASS"
    compliance_pass: true
    quality_pass: true
    next_step: "可以进入执行阶段"
```

---

## 6. 评估流程与触发条件

### 评估流程

```
┌─────────────┐
│  用户输入想法  │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌──────────────────┐
│ Step 1: 拆解  │────▶│ 评估 B1.1-B1.4   │
└──────┬──────┘     └────────┬─────────┘
       │                     │
       │              PASS? ─┤
       │              │      │ No → 修复拆解 → 重新评估
       │              Yes    │
       ▼              ▼      │
┌─────────────┐     ┌────────┴─────────┐
│ Step 2: 研究  │────▶│ 评估 B2.1-B2.9   │
└──────┬──────┘     └────────┬─────────┘
       │                     │
       │              PASS? ─┤
       │              │      │ No → 补充研究 → 重新评估
       │              Yes    │
       ▼              ▼      │
┌─────────────┐     ┌────────┴─────────┐
│ Step 3: 评估  │────▶│ 评估 B3.1-B3.5   │
│  标准定义     │     └────────┬─────────┘
└──────┬──────┘              │
       │              PASS? ─┤
       │              │      │ No → 完善标准 → 重新评估
       │              Yes    │
       ▼              ▼      │
┌─────────────┐     ┌────────┴─────────┐
│ Step 4: 计划  │────▶│ 评估 B4.1-B4.6   │
│  生成        │     │ + 质量评估 Q1-Q4  │
└──────┬──────┘     └────────┬─────────┘
       │                     │
       │              PASS? ─┤
       │              │      │ No → 修复计划 → 重新评估
       │              Yes    │
       ▼              ▼      │
┌─────────────┐
│  输出最终计划  │◀───────────┘
└─────────────┘
```

### 触发条件

| 触发时机 | 评估范围 | 说明 |
|---------|---------|------|
| 拆解完成后 | B1.1 - B1.4 | 检查任务树结构合规性 |
| 研究完成后 | B2.1 - B2.9 | 检查研究过程合规性 |
| 评估标准完成后 | B3.1 - B3.5 | 检查 Benchmark 定义合规性 |
| 计划生成完成后 | B4.1 - B4.6 + Q1-Q4 | 检查计划合规性 + 整体质量 |
| 手动触发 | 全部 38 个 Benchmark | 用户可随时手动触发完整评估 |

### 评估失败处理策略

1. **首次失败**：自动生成修复建议，返回对应阶段进行修复
2. **二次失败**（同一 Benchmark）：标记为需要人工审核，提供详细的不合规原因
3. **三次失败**（同一 Benchmark）：终止流程，输出完整评估报告，建议重新开始对应阶段

### 评估注意事项

- LLM-as-judge 评估应使用与计划生成不同的模型实例，避免自我评估偏差
- 每次评估应保留完整的评估依据（evidence），便于追溯和审核
- 评估结果应保存为 YAML 格式，方便后续分析和对比
- 建议在评估 Prompt 中加入"思考链"（Chain-of-Thought）要求，提高评估准确性
- 对于存在争议的评估结果，建议使用多个 LLM 进行交叉验证
