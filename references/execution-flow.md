---
version: "2.0"
updated: "2026-04-09"
stage: production
skill_version: "v2.0+"
maintainer: "Task Execution Summary Generator Team"
---

# 执行流程文档 (Execution Flow)

本文档描述"任务执行总结报告生成器"技能的完整执行流程，包括各阶段处理逻辑、数据流转、异常处理机制及性能特征。

---

## 目录

- [1. 流程概述](#1-流程概述)
- [2. 主执行流程](#2-主执行流程)
  - [Step 1: 参数解析与验证](#step-1-参数解析与验证)
  - [Step 2: 触发模式识别](#step-2-触发模式识别)
  - [Step 3: 信息收集阶段](#step-3-信息收集阶段)
  - [Step 4: 分析处理阶段](#step-4-分析处理阶段)
  - [Step 5: 报告生成阶段](#step-5-报告生成阶段)
  - [Step 6: 智能推荐生成](#step-6-智能推荐生成)
  - [Step 7: 质量检查与输出](#step-7-质量检查与输出)
- [3. 异常路径汇总](#3-异常路径汇总)
- [4. 性能基线](#4-性能基线)
- [5. 状态机说明](#5-状态机说明)
- [附录：错误码索引](#附录错误码索引)

---

## 1. 流程概述

### 1.1 设计原则

本技能围绕三大核心设计原则构建：

| 原则 | 核心要求 | 实现方式 |
|------|---------|---------|
| **确定性** | 相同输入产生可复现的输出结构 | InternalConfig统一配置、类型化数据模型、基于阈值的决策分支 |
| **可观测性** | 每个步骤产出可审查的中间结果 | 结构化日志记录、中间数据对象序列化、QualityMetrics质量指标追踪 |
| **容错性** | 非致命错误不阻断主流程 | Error/Warning分级处理、Graceful Degradation降级策略、每步骤独立错误捕获 |

### 1.2 整体架构

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  Step 1  │ → │  Step 2  │ → │  Step 3  │ → │  Step 4  │ → │  Step 5  │
│ 参数解析 │   │ 模式识别 │   │ 信息收集 │   │ 分析处理 │   │ 报告生成 │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └────┬─────┘
     │              │              │              │              │
     ↓              ↓              ↓              ↓              ↓
InternalConfig  CollectionScope  CollectedData  AnalysisReport  DraftReport
                                                                  │
┌──────────┐   ┌──────────┐                                       ↓
│  Step 6  │ → │  Step 7  │                                 FinalResponse
│ 推荐生成 │   │ 质检输出 │
└──────────┘   └──────────┘
```

**架构分层**：
- **执行流水线**：按顺序执行7个处理步骤 (Step 1~7)
- **数据流层**：定义步骤间的数据传递格式 (InternalConfig → CollectionScope → CollectedData → AnalysisReport → DraftReport → FinalResponse)
- **异常处理层**：错误码体系 E001-E032，分类处理策略

### 1.3 关键性能指标

**总耗时分布概览**（标准版报告，中等复杂度任务）：

| 步骤 | 耗时占比 | 预估耗时 | 主要瓶颈 |
|------|---------|---------|---------|
| Step 3 信息收集 | 40-50% | 30-120s | 对话历史解析 |
| Step 4 分析处理 | 35-40% | 60-180s | 五维分析计算 |
| Step 5 报告生成 | 15-20% | 30-120s | 内容生成 |
| Step 6 推荐生成 | 5-10% | 30-60s | 推荐算法 |
| Step 7 质检输出 | < 2% | < 10s | - |
| Step 1-2 | < 3% | < 3s | - |
| **总计** | 100% | **2-8 min** | - |

**场景化性能参考**：

| 任务场景 | 对话轮数 | 详细程度 | 预估总耗时 |
|---------|---------|---------|-----------|
| 简单任务总结 | < 20轮 | 摘要版 | 1-2 min |
| 标准任务复盘 | 20-50轮 | 标准版 | 3-5 min |
| 复杂项目总结 | > 50轮 | 详细版 | 6-10 min |

---

## 2. 主执行流程

### Step 1: 参数解析与验证

```
接收请求 → 参数完整性检查 → 类型与范围校验 → 应用默认值 → InternalConfig
              │                    │
              └─ ❌ E001-E005 ─────┘
```

**输入**：结构化JSON / 自然语言 / 快捷命令

**处理逻辑**：
1. **请求解析**：尝试JSON/YAML解析 → NLP提取参数 → 命令映射
2. **必填参数校验**：`task_scope`(必填), `detail_level`(默认standard), `include_sections`(默认全部10章), `output_format`(默认markdown), `language`(默认zh-CN)
3. **校验优先级**：存在性检查 → 类型匹配 → 值域范围 → 逻辑一致性 → 安全性检查
4. **默认值应用**：构建标准化内部配置对象

**输出**：`InternalConfig` 或 `ErrorResponse`

**InternalConfig 结构**：
```typescript
interface InternalConfig {
  taskScope: string;
  detailLevel: 'summary' | 'standard' | 'detailed';
  includeSections: ChapterName[];
  outputFormat: OutputFormat;
  language: string;
  timeRange: TimeRange;
  generationOptions: GenerationOptions;
  rawRequest: string;
  parsedAt: Date;
  validationWarnings?: string[];
}
```

**耗时预估**：< 1秒 | **失败概率**：< 5%

---

### Step 2: 触发模式识别

```
识别触发模式 → 确认/调整范围 → CollectionScope
  • 自动触发
  • 手动触发
  • 命令行触发
```

**输入**：`InternalConfig`

**处理逻辑**：

| 触发模式 | 判定条件 | 典型场景 |
|---------|---------|---------|
| **自动触发** | 检测到完成信号词 + 任务复杂度 > 中等 | "好了"、"完成了"、"可以了" |
| **手动触发** | 显式命令关键词 | "/summary"、"请生成总结" |
| **命令行触发** | 配置化参数调用 | API调用、脚本触发 |

**信号词检测库**：
- **明确完成信号**："完成了"、"好了"、"成功了"、"可以了"、"搞定了"、"OK"
- **隐含完成意图**："帮我总结一下"、"回顾一下"、"复盘"、"做得怎么样"

**歧义处理策略**：
- 时间范围不明 → 使用全量范围
- 任务边界模糊 → 使用最近的主任务
- 协作信息存疑 → 检测参与者数量，默认单人模式

**输出**：`CollectionScope`

```typescript
interface CollectionScope {
  timeWindow: { start: Date; end: Date; estimatedDuration: number };
  infoCategories: InfoCategory[];
  dataSources: DataSource[];
  triggerMode: 'auto' | 'manual' | 'command';
  confidence: number;
  clarificationsApplied: string[];
}
```

**耗时预估**：< 2秒

---

### Step 3: 信息收集阶段

> **核心阶段**，占总耗时40-50%

```
┌─────────────────────────────────────────────────────────────┐
│  Step 3: 信息收集                                            │
│  ├─ 3.1 数据源适配（对话历史/操作记录/文件变更）              │
│  ├─ 3.2 信息抽取（实体识别/关系抽取/事件检测）                │
│  ├─ 3.3 数据整合（清洗去重/时序对齐/关联建立）                │
│  └─ 3.4 质量检查（完整性/准确性/缺口检测）                    │
└─────────────────────────────────────────────────────────────┘
```

**输入**：`CollectionScope`

**6大类别结构化信息抽取**：

| 类别 | 抽取目标 | 关键信号词 | 输出结构 |
|------|---------|-----------|---------|
| 任务目标实体 | 目标关键词、成功标准 | "目标是"、"需要实现"、"期望" | ObjectiveEntity[] |
| 时间节点实体 | 时间表达、阶段划分 | "T+Xmin"、"第一阶段" | TimelineEntity[] |
| 决策实体 | 选择行为、依据 | "决定用"、"选择了"、"采用" | DecisionEntity[] |
| 问题实体 | 错误、异常、困难 | "报错"、"出错"、"为什么" | IssueEntity[] |
| 资源实体 | 工具、框架、服务 | 工具名、库名、框架名 | ResourceEntity[] |
| 协作实体 | 团队角色、分工 | 人名、角色词 | CollaborationEntity[] |

**质量检查阈值**：
- 综合覆盖率 ≥ 90%：✅ 优秀，直接进入下一步
- 综合覆盖率 70-90%：⚠️ 发出 E010 警告，标注缺失项，继续
- 综合覆盖率 < 70%：❌ 发出 E011 错误，提示用户选择（降级/补充/终止）

**输出**：`CollectedData` / `E010` / `E011` / `E012`

```typescript
interface CollectedData {
  metadata: {
    collectionTime: Date;
    dataSourceStats: Record<DataSource, number>;
    qualityScore: number;
    coverageByCategory: Record<InfoCategory, number>;
    warnings?: string[];
  };
  taskInfo: { objectives: ObjectiveEntity[]; timeline: TimelineEntity[] };
  executionInfo: { decisions: DecisionEntity[]; issues: IssueEntity[]; operations: OperationRecord[] };
  resourceInfo: { resources: ResourceEntity[]; dependencies: DependencyInfo[] };
  collaborationInfo?: { participants: Participant[]; communications: CommunicationEvent[] };
}
```

**耗时预估**：30-120秒

---

### Step 4: 分析处理阶段

```
┌─────────────────────────────────────────────┐
│  Step 4: 五维分析引擎                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │ 4.1 目标 │ │ 4.2 时间 │ │ 4.3 资源 │       │
│  │ 达成度   │ │ 效能    │ │ 利用率  │       │
│  └─────────┘ └─────────┘ └─────────┘       │
│       ↓          ↓          ↓              │
│  ┌─────────┐ ┌─────────┐                   │
│  │ 4.4 问题 │ │ 4.5 协作 │(条件)           │
│  │ 模式    │ │ 效果    │                   │
│  └─────────┘ └─────────┘                   │
│       ↓                                    │
│  AnalysisReport (五维分析结果)              │
└─────────────────────────────────────────────┘
```

**输入**：`CollectedData`

**五维分析**（详见 [terminology.md](terminology.md) 相关术语定义）：

| 维度 | 分析方法 | 核心指标 |
|------|---------|---------|
| **目标达成度** | 对比法 + 量化评估 | [达成率](terminology.md#26-达成率-achievement-rate) = 实际值/期望值 × 100% |
| **时间效能** | 偏差计算 + 瓶颈识别 | 总体[时效比](terminology.md#34-时效比-time-efficiency-ratio)、阶段均衡度、[瓶颈](terminology.md#33-瓶颈-bottleneck)集中度 |
| **资源利用率** | [利用率](terminology.md#44-利用率-utilization-rate)计算 + [浪费](terminology.md#52-浪费-waste)识别 | 必要性、充分性、适配性、性价比评分 |
| **问题模式** | 分类统计 + 有效性评估 | 技术/环境/流程/认知四层分类 |
| **协作效果** | 多维度评分 | 沟通效率、分工合理、协同效果 (条件启用) |

**达成等级标准**：
- 优秀 (Excellent)：≥ 95%
- 良好 (Good)：85% - 94%
- 合格 (Acceptable)：70% - 84%
- 待改进 (Needs Improvement)：< 70%

**输出**：`AnalysisReport` / `E021` / `E022`

```typescript
interface AnalysisReport {
  objectiveAchievement: { overallRate: number; grade: string; subGoals: SubGoalAnalysis[] };
  timeEfficiency: { totalTime: number; phases: PhaseAnalysis[]; metrics: EfficiencyMetrics; bottlenecks: Bottleneck[] };
  resourceUtilization: { resources: ResourceAssessment[]; wasteSummary: WasteSummary };
  issuePatterns: { totalIssues: number; distribution: IssueDistribution; topPatterns: Pattern[] };
  collaborationEffectiveness?: { overallScore: number; dimensions: DimensionScore[] };
  summary: { keyFindings: string[]; overallAssessment: string };
}
```

**耗时预估**：60-180秒

---

### Step 5: 报告生成阶段

```
AnalysisReport + InternalConfig → 5.1 选择模板 → 5.2 映射数据 → 5.3 填充内容 → 5.4 优化格式 → 5.5 润色语言 → DraftReport
```

**输入**：`AnalysisReport`, `InternalConfig`, `CollectedData`

**模板选择**：

| 详细程度 | 模板变体 | 包含章节 | 预计篇幅 |
|---------|---------|---------|---------|
| `summary` | 摘要版 | 第1章(完整) + 第10章(摘要) + 其他(标题+要点) | 500-800字 |
| `standard` | 标准版 | 全部10章（标准详细度） | 3000-5000字 |
| `detailed` | 详细版 | 全部10章（深度展开）+ 完整附录 | 8000-15000字 |

**章节定制处理**：基础章节（1,2,3,5,9,10章）建议始终保留；可选章节（4,6,7,8章）按需包含。

**输出**：`DraftReport` / `E031` / `E032`

```typescript
interface DraftReport {
  content: string;
  metadata: {
    generatedAt: Date;
    templateUsed: TemplateVariant;
    chapterCount: number;
    wordCount: number;
  };
  qualityIndicators: {
    structureCompleteness: number;
    dataMappingCoverage: number;
    formatCompliance: number;
  };
}
```

**耗时预估**：30-120秒

---

### Step 6: 智能推荐生成

```
DraftReport + AnalysisReport → 6.1 方法论提取 + 6.2 改进建议 + 6.3 风险预警 → Recommendations
```

**输入**：`DraftReport`, `AnalysisReport`, `CollectedData`

**方法论提取标准**：有效性证明、普适性、可描述性、优势明显

**改进建议生成原则**：基于证据、具体可行、优先级明确、量化预期、责任明确

**优先级分级**：

| 等级 | 分数区间 | 标记 | 响应要求 |
|------|---------|------|---------|
| P0-紧急 | 8.0-10.0 | 🔴 | 本周内启动 |
| P1-高 | 6.5-7.9 | 🟠 | 两周内启动 |
| P2-中 | 5.0-6.4 | 🟡 | 月内规划 |
| P3-低 | 3.0-4.9 | 🟢 | 季度规划 |
| P4-可选 | <3.0 | ⚪ | 长期储备 |

**输出**：`Recommendations`

```typescript
interface Recommendations {
  methodologies: Methodology[];
  improvementSuggestions: Suggestion[];
  riskWarnings: RiskWarning[];
  summary: {
    methodologyCount: number;
    suggestionCount: number;
    highPriorityCount: number;
    riskCount: number;
    criticalRiskCount: number;
  };
}
```

**耗时预估**：30-60秒

---

### Step 7: 质量检查与输出

```
DraftReport + Recommendations → 7.1 结构完整性验证 → 7.2 内容准确性抽检 → 7.3 组装最终响应 → Final Response
```

**输入**：`DraftReport`, `Recommendations`, `InternalConfig`

**结构完整性检查清单**：
- YAML Frontmatter 完整
- 10章齐全（或按定制）
- 每章包含规定小节
- 表格格式正确
- 代码块有语言标注
- 标题层级正确

**内容准确性抽检**：数字一致性（100%关键数字）、逻辑自洽性（抽样20%）、引用有效性（抽样30%）、时间线连贯性（100%）、建议可操作性（100%）

**输出**：`SuccessResponse`

```typescript
interface SuccessResponse {
  status: 'success';
  report: { content: string; fileName: string; metadata: ReportMetadata };
  qualityMetrics: {
    overallScore: number;
    informationCoverage: number;
    factualAccuracy: number;
    structuralIntegrity: number;
    suggestionQuality: number;
  };
  processingInfo: {
    totalDuration: number;
    stepDurations: Record<StepName, number>;
    warningsIssued: ErrorCode[];
    degradationsApplied: string[];
  };
  recommendations: Recommendations;
}
```

**文件命名规范**：`task-summary-[任务名称简写]-YYYYMMDD.md`

**耗时预估**：< 10秒

---

## 3. 异常路径汇总

### 完整异常路径图

```
                    正常路径
Step1 → Step2 → Step3 → Step4 → Step5 → Step6 → Step7 → ✅ Success
  │        │       │       │       │       │
  ↓E001   ↓       ↓E010   ↓E021   ↓E031   │
  ❌Error  │       ⚠️Warn  ⚠️Warn  ❌Error  │
          │       ↓       ↓       ↓       │
          │   [降级继续] [跳过] [回退]    │
          └───────────────────────────────┘
                    (带警告的成功)
```

### 错误码详细说明

#### 第一类：参数验证错误 (E001-E005)

| 错误码 | 错误名称 | 触发条件 | 严重级别 | 处理方式 | 恢复可能 |
|-------|---------|---------|---------|---------|---------|
| E001 | 缺少必填参数 | 必填参数未提供 | 🔴 致命 | 返回错误响应，提示补充 | 需用户修正 |
| E002 | 参数类型错误 | 参数值类型不符 | 🔴 致命 | 返回错误响应，给出格式示例 | 需用户修正 |
| E003 | 参数值越界 | 参数值超出范围 | 🟡 警告 | 修正到最近合法值，发出警告 | 自动恢复 |
| E004 | 参数冲突 | 参数间逻辑矛盾 | 🔴 致命 | 返回错误响应，指出冲突参数 | 需用户选择 |
| E005 | 安全策略违规 | 参数含非法内容 | 🔴 致命 | 返回错误响应，拒绝处理 | 需用户修正 |

**影响范围**：仅在 Step 1 发生，阻断后续所有步骤

#### 第二类：数据质量错误 (E010-E012)

| 错误码 | 错误名称 | 触发条件 | 严重级别 | 处理方式 | 恢复可能 |
|-------|---------|---------|---------|---------|---------|
| E010 | 信息覆盖不足 | 覆盖率 70-90% | 🟡 警告 | 标注缺失项，降级继续 | 可降级运行 |
| E011 | 信息严重缺失 | 覆盖率 < 70% | 🔴 错误 | 提供三选一：降级/补充/终止 | 用户选择 |
| E012 | 数据源不可用 | 关键数据源无法访问 | 🔴 错误 | 尝试备用源，全部不可用时终止 | 部分可恢复 |

**降级策略**：缺失信息标注"[信息不足]"，相关分析维度降低可信度，qualityMetrics降低分数。

#### 第三类：分析引擎错误 (E021-E022)

| 错误码 | 错误名称 | 触发条件 | 严重级别 | 处理方式 | 恢复可能 |
|-------|---------|---------|---------|---------|---------|
| E021 | 部分分析失败 | 某一维度数据不足 | 🟡 警告 | 跳过该维度，其他维度正常输出 | 自动恢复 |
| E022 | 核心分析引擎错误 | 分析引擎异常 | 🔴 错误 | 回退到简化分析模式 | 部分可恢复 |

#### 第四类：报告生成错误 (E031-E032)

| 错误码 | 错误名称 | 触发条件 | 严重级别 | 处理方式 | 恢复可能 |
|-------|---------|---------|---------|---------|---------|
| E031 | 模板渲染失败 | 模板引擎异常 | 🔴 错误 | 回退到备用模板 | 通常可恢复 |
| E032 | 内容生成失败 | 内容生成器异常 | 🔴 错误 | 使用已有数据直接组装 | 部分可恢复 |

---

## 4. 性能基线

### 各阶段性能明细

| 步骤 | 预估耗时 | 占总时长% | 主要瓶颈 | 优化空间 |
|------|---------|----------|---------|---------|
| Step 1: 参数解析 | < 1s | < 1% | 无 | 已最优 |
| Step 2: 模式识别 | < 2s | < 2% | 无 | 已最优 |
| Step 3: 信息收集 | 30-120s | 40-50% | 对话历史解析 | 可并行化 |
| Step 4: 分析处理 | 60-180s | 35-40% | 五维分析计算 | 可缓存 |
| Step 5: 报告生成 | 30-120s | 15-20% | 内容生成 | 可预编译模板 |
| Step 6: 推荐生成 | 30-60s | 5-10% | 推荐算法 | 可增量更新 |
| Step 7: 质检输出 | < 10s | < 2% | 无 | 已最优 |
| **总计** | **2-8 min** | **100%** | Step 3 + Step 4 | **约 30-40%** |

### 性能优化策略

**短期优化**：
1. **Step 3 并行化**：多数据源并行处理，预期减少30-40%耗时
2. **Step 5 模板预编译**：首次渲染8s → 后续3s，降低62%
3. **增量处理**：重复生成场景减少60-80% Step 3耗时

**中期优化**：
4. **Step 4 五维分析并行化**：预期减少60-70%耗时
5. **分析结果缓存**：相似任务类型缓存复用
6. **异步流水线**：支持流式输出

**长期优化**：
7. **智能采样策略**：>50轮对话场景减少40-50% Step 3耗时

### 生产环境监控指标

| 指标名称 | 目标值 | 告警阈值 |
|---------|--------|---------|
| P50 延迟 | < 3 分钟 | > 4 分钟 |
| P95 延迟 | < 5 分钟 | > 8 分钟 |
| P99 延迟 | < 10 分钟 | > 15 分钟 |
| 错误率 | < 2% | > 5% |
| 降级率 | < 15% | > 30% |
| 质量评分均值 | > 85 | < 75 |

### 超时与资源限制策略

| 步骤 | 默认超时 | 最大超时 | 超时处理 |
|------|---------|---------|---------|
| Step 1 | 5s | 10s | 返回 E001 |
| Step 2 | 5s | 10s | 使用默认模式 |
| Step 3 | 180s | 300s | 保存已收集数据，发出 E051 |
| Step 4 | 180s | 300s | 返回已完成的分析维度 |
| Step 5 | 120s | 180s | [降级](terminology.md#229-降级-degradation)到简化模板 |
| Step 6 | 60s | 90s | 跳过推荐 |
| Step 7 | 30s | 60s | 跳过检查，直接输出 |
| **全流程** | **480s** | **720s** | **返回已完成部分** |

---

## 5. 状态机说明

### 执行状态定义

```
IDLE → PARSING → COLLECTING → ANALYZING → GENERATING → RECOMMENDING → FINALIZING → COMPLETE
  ↑         ↓           ↓           ↓            ↓              ↓              ↓
  └─────────┴───────────┴───────────┴────────────┴──────────────┴──────────────┘
                           (任何错误状态均可回到 IDLE 或 ERROR)
```

### 状态详解

| 状态 | 英文名称 | 触发条件 | 允许的下一状态 |
|------|---------|---------|-------------|
| 空闲 | IDLE | 技能初始化 / 上次执行完成 | PARSING |
| 解析中 | PARSING | 开始 Step 1 | COLLECTING / ERROR |
| 收集中 | COLLECTING | 开始 Step 3 | ANALYZING / ERROR |
| 分析中 | ANALYZING | 开始 Step 4 | GENERATING / ERROR |
| 生成中 | GENERATING | 开始 Step 5 | RECOMMENDING / ERROR |
| 推荐中 | RECOMMENDING | 开始 Step 6 | FINALIZING / ERROR |
| 完善中 | FINALIZING | 开始 Step 7 | COMPLETE / ERROR |
| 完成 | COMPLETE | 所有步骤成功 | IDLE |
| 错误 | ERROR | 任何步骤遇到致命错误 | IDLE |

### 状态转换规则

**合法转换**：
- IDLE ──→ PARSING (新请求到达)
- PARSING ──→ COLLECTING (参数验证通过) / ERROR (验证失败)
- COLLECTING ──→ ANALYZING (信息收集完成) / ERROR (数据严重缺失)
- ANALYZING ──→ GENERATING (分析完成) / ERROR (分析引擎故障)
- GENERATING ──→ RECOMMENDING (报告草稿完成) / ERROR (生成完全失败)
- RECOMMENDING ──→ FINALIZING (推荐生成完成)
- FINALIZING ──→ COMPLETE (质检通过) / ERROR (致命问题)
- COMPLETE / ERROR ──→ IDLE (重置)

**非法转换**：
- ✗ 任意状态 ──→ COMPLETE (必须走完所有步骤)
- ✗ ERROR ──→ PARSING (必须先回到 IDLE)
- ✗ COLLECTING ──→ GENERATING (不能跳过分析)

---

## 附录：错误码索引

| 错误码 | 所属类别 | 错误名称 | 严重级别 | 发生步骤 | 可恢复性 |
|-------|---------|---------|---------|---------|---------|
| E001 | 参数验证 | 缺少必填参数 | 🔴 致命 | Step 1 | 需用户修正 |
| E002 | 参数验证 | 参数类型错误 | 🔴 致命 | Step 1 | 需用户修正 |
| E003 | 参数验证 | 参数值越界 | 🟡 警告 | Step 1 | 自动修正 |
| E004 | 参数验证 | 参数冲突 | 🔴 致命 | Step 1 | 需用户选择 |
| E005 | 参数验证 | 安全策略违规 | 🔴 致命 | Step 1 | 需用户修正 |
| E010 | 数据质量 | 信息覆盖不足 | 🟡 警告 | Step 3 | 可降级运行 |
| E011 | 数据质量 | 信息严重缺失 | 🔴 错误 | Step 3 | 用户选择 |
| E012 | 数据质量 | 数据源不可用 | 🔴 错误 | Step 3 | 部分可恢复 |
| E021 | 分析引擎 | 部分分析失败 | 🟡 警告 | Step 4 | 自动跳过 |
| E022 | 分析引擎 | 核心分析引擎错误 | 🔴 错误 | Step 4 | 可降级运行 |
| E031 | 报告生成 | 模板渲染失败 | 🔴 错误 | Step 5 | 可回退模板 |
| E032 | 报告生成 | 内容生成失败 | 🔴 错误 | Step 5 | 部分可恢复 |

---

*文档版本：v2.0* | *最后更新：2026-04-09* | *适用于 Task Execution Summary Generator v2.0*
