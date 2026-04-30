# Idea-to-Action Plan Skill

将一个模糊的想法，通过 AI 驱动的递归拆解 → 多轮迭代研究 → Benchmark & Evaluation 定义，最终输出一份可直接交给 AI 执行的结构化计划。

## 这是什么？

这是一个 AI Skill（技能模块），定义了一套完整的"想法 → 可执行计划"转换方法论。当你有一个模糊的想法时，这个 skill 会指导 AI：

1. **递归拆解**：将想法拆解到最小可执行单元（MEU）
2. **多轮迭代研究**：对每个 MEU 通过 4 轮迭代识别堵点、交叉验证方案、记录决策和踩坑信息
3. **定义评估标准**：为每个 MEU 设计可量化的 Benchmark
4. **生成执行计划**：输出包含避坑专区和决策日志的 9 段式 AI 可执行计划
5. **质量评估**：两轮评估（23 项合规性 + 15 项质量）确保计划质量

## 文件结构

```
idea-to-action/
├── SKILL.md                              # 主指令文件（AI 入口，~320 行）
├── README.md                             # 本文件（人类阅读）
│
├── steps/                                # Step 核心指南（执行时按需加载）
│   ├── step1-decompose.md                # Step 1: 递归拆解（~250 行）
│   ├── step2-research.md                 # Step 2: 多轮迭代研究（~160 行）
│   ├── step3-evaluate.md                 # Step 3: 定义评估标准（~200 行）
│   ├── step4-plan.md                     # Step 4: 生成执行计划（~170 行）
│   └── step5-quality.md                  # Step 5: 质量评估（~200 行）
│
├── references/                           # 详细参考文档（进一步按需加载）
│   ├── step1-node-reference.md           # 节点字段完整参考 + 依赖详解
│   ├── step2-blocker-types.md            # 6 种堵点类型详解
│   ├── step2-output-format.md            # 研究输出格式详解 + 示例
│   ├── step2-validation.md               # 置信度定义 + 交叉验证方法
│   ├── step3-benchmark-types.md          # Benchmark 类型详解 + 模板
│   ├── step4-prompt-structure.md         # 9 段式 Prompt 详细结构
│   ├── step4-risk-management.md          # 风险与开放问题管理
│   └── step5-evaluation-detail.md        # 完整评估框架 + Prompt 模板
│
├── templates/                            # Prompt 模板
│   ├── decompose-prompt.md               # 拆解 Prompt
│   ├── research-prompt.md                # 研究 Prompt（4 轮迭代）
│   ├── evaluation-prompt.md              # 评估 Prompt
│   └── execution-prompt.md               # 执行 Prompt（9 段式）
│
└── examples/                             # 完整示例
    ├── filewatch-cli/                    # 文件监控 CLI 工具
    │   ├── task-tree.yaml
    │   ├── research.yaml
    │   └── execution-plan.md
    └── wechat-articles/                  # 微信公众号文章抓取
        ├── task-tree.yaml
        ├── research.yaml
        ├── benchmarks.yaml
        ├── execution-plan.md
        ├── prompts.yaml
        ├── dependency-graph.mermaid
        └── evaluation-report-v1.yaml
```

## 上下文管理策略

本 Skill 采用**三层渐进式披露**架构（参考 Anthropic Agent Skills 规范）：

| 层级 | 内容 | 加载时机 | Token 开销 |
|------|------|----------|-----------|
| **L1: 发现** | SKILL.md 的 `name` + `description` | 启动时 | ~100 tokens |
| **L2: 指令** | SKILL.md 正文 + 对应 Step 的核心指南 | 执行对应 Step 时 | ~500-800 tokens |
| **L3: 参考** | references/ 下的详细文档 | 需要深入了解时按需加载 | 按需 |

**实际效果**：执行 Step 2 时只需加载 ~650 行（SKILL.md + step2-research.md），而非原来的 ~1525 行（SKILL.md + research.md），**上下文节省 57%**。

## 如何使用

### 方式一：直接使用

将 `idea-to-action/` 目录放入你的 AI 工具的 skills 目录，然后用自然语言触发：

- "帮我拆解这个想法：..."
- "研究一下怎么做：..."
- "生成一个执行计划：..."

### 方式二：手动执行

按以下步骤依次使用 templates/ 中的 Prompt：

1. 用 `decompose-prompt.md` 拆解想法 → 得到任务树
2. 用 `research-prompt.md` 多轮迭代研究每个 MEU → 得到堵点、方案、决策摘要
3. 用 `evaluation-prompt.md` 定义 Benchmark → 得到验收标准
4. 用 `execution-prompt.md` 生成 9 段式执行计划 → 得到可执行 Prompt（含避坑专区 + 决策日志）

### 方式三：参考示例

查看 `examples/wechat-articles/` 目录，了解一个完整的执行案例（含评估报告）。

## 核心理念

- **所有 Benchmark 都必须达成**：没有"可选"标准
- **研究先行**：每个单元在执行前必须完成 4 轮迭代研究
- **避坑优先**：研究阶段的踩坑记录和决策日志必须传递到执行计划
- **可验证性**：每个 Benchmark 都可量化、可测试
- **上下文高效**：三层渐进式披露，按需加载，避免上下文爆炸

## 评估体系

本 skill 内置两套评估体系（详见 `steps/step5-quality.md`）：

1. **Prompt 执行合规性评估**（23 个 Benchmark）：检查最终输出是否准确执行了各阶段的 Prompt
2. **Plan 质量评估**（15 个 Benchmark，4 维度加权）：检查计划是否可执行、研究是否充分
