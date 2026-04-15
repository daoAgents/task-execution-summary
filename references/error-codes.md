---
version: "2.0"
updated: "2026-04-15"
stage: production
skill_version: "v2.0+"
maintainer: "Task Execution Summary Generator Team"
---

# 错误码定义文档 (Error Codes Reference)

本文档定义任务执行总结报告生成器的错误码体系，包含15个错误码（E001-E005, E010-E012, E021-E022, E031-E032, E041, E051）的分类、定义、处理策略和降级机制。

---

## 目录

- [1. 概述](#1-概述)
  - [1.1 错误处理设计理念](#11-错误处理设计理念)
  - [1.2 错误码命名规则](#12-错误码命名规则)
  - [1.3 错误响应通用结构](#13-错误响应通用结构)
- [2. 错误分类体系总览](#2-错误分类体系总览)
- [3. 详细错误码定义](#3-详细错误码定义)
  - [E001: MissingRequiredParameter](#e001-missingrequiredparameter)
  - [E002: InvalidParameterType](#e002-invalidparametertype)
  - [E003: ParameterValueOutOfRange](#e003-parametervalueoutofrange)
  - [E004: ConflictingParameters](#e004-conflictingparameters)
  - [E005: InvalidChapterCombination](#e005-invalidchaptercombination)
  - [E010: InsufficientDataWarning](#e010-insufficientdatawarning)
  - [E011: ConversationHistoryUnavailable](#e011-conversationhistoryunavailable)
  - [E012: FileAccessDenied](#e012-fileaccessdenied)
  - [E021: GoalAnalysisFailed](#e021-goalanalysisfailed)
  - [E022: TimelineReconstructionFailed](#e022-timelinereconstructionfailed)
  - [E031: TemplateNotFound](#e031-templatenotfound)
  - [E032: ReportGenerationTimeout](#e032-reportgenerationtimeout)
  - [E041: InsufficientMemory](#e041-insufficientmemory)
  - [E051: ExecutionTimeout](#e051-executiontimeout)
- [4. 错误处理策略矩阵](#4-错误处理策略矩阵)
- [5. 降级策略详解](#5-降级策略详解)
- [6. 错误码快速查询表](#6-错误码快速查询表)

---

## 1. 概述

### 1.1 错误处理设计理念

核心原则：
- **分层防御**：输入验证 → 数据检查 → 分析容错 → 生成降级
- **优雅降级**：Warning级别继续执行，质量评分反映降级程度
- **透明告知**：明确错误码、恢复建议、完整日志
- **可观测性**：质量评分量化可信度，关键节点健康检查

### 1.2 错误码命名规则

格式: `E + 类别编号(1位) + 序号(2位)`，如 E001, E010

| 编号 | 类别 | 错误码范围 |
|------|------|-----------|
| 0 | 参数验证 | E001-E005 |
| 1 | 数据源 | E010-E012 |
| 2 | 分析引擎 | E021-E022 |
| 3 | 报告生成 | E031-E032 |
| 4 | 系统资源 | E041 |
| 5 | 超时 | E051 |

### 1.3 错误响应通用结构

```json
{
  "success": false,
  "error": {
    "code": "E001",
    "name": "MissingRequiredParameter",
    "message": "缺少必填参数: task_name",
    "category": "parameter_validation",
    "severity": "Error",
    "http_status": 400,
    "timestamp": "2026-04-15T14:30:00Z",
    "request_id": "req_abc123xyz",
    "context": {"missing_parameter": "task_name"},
    "recovery": {
      "mode": "terminate",
      "suggestions": ["请提供 task_name 参数"]
    }
  }
}
```

| 字段 | 说明 |
|------|------|
| success | 是否成功 |
| error.code | 错误码，如 "E001" |
| error.name | 错误名称 |
| error.severity | Critical/Error/Warning |
| error.http_status | HTTP 状态码 |
| error.recovery.mode | terminate/degrade/partial/estimate |

---

## 2. 错误分类体系总览

| 类别编码 | 类别名称 | 错误码范围 | 严重级别 | 处理策略 | 触发阶段 |
|---------|---------|-----------|---------|---------|---------|
| **E0xx** | 参数验证错误 | E001-E005 | Error | 验证失败立即返回 | 触发检测 → 信息收集 |
| **E1xx** | 数据源错误 | E010-E012 | Error/Warning | 重试或降级 | 信息收集阶段 |
| **E2xx** | 分析引擎错误 | E021-E022 | Error/Warning | 使用默认值或跳过 | 分析处理阶段 |
| **E3xx** | 报告生成错误 | E031-E032 | Error | 返回错误或部分结果 | 报告生成阶段 |
| **E4xx** | 系统资源错误 | E041 | Critical | 终止并告警 | 任意阶段 |
| **E5xx** | 超时错误 | E051 | Error | 终止或返回部分结果 | 任意阶段 |

**严重级别说明**：

| 严重级别 | 图标 | 定义 | 对执行的影响 |
|------|------|------|-------------|
| **Critical** | 🔴 | 系统级故障 | 立即终止，不生成输出 |
| **Error** | 🟠 | 功能性错误 | 终止当前步骤，可能返回部分结果 |
| **Warning** | 🟡 | 非致命问题 | 标记警告，继续执行 |

> 严重程度术语定义详见 [terminology.md#44-严重程度-severity](terminology.md#44-严重程度-severity)

---

## 3. 详细错误码定义

---

### E001: MissingRequiredParameter

| 属性 | 值 |
|------|---|
| **错误码** | E001 |
| **名称** | MissingRequiredParameter |
| **类别** | parameter_validation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 400 |
| **恢复模式** | terminate |

**触发条件**：缺少必填参数（如 `task_name`、对话历史引用、输出路径）

**错误消息模板**：
```
缺少必填参数: {parameter_name}
必需参数: {required_params}
当前已提供: {provided_params}
```

**恢复建议**：检查API文档、补充参数值、交互式引导

**预防措施**：SDK参数校验、清晰文档、智能推断

> 示例见 [examples-v2.md](examples-v2.md)

---

### E002: InvalidParameterType

| 属性 | 值 |
|------|---|
| **错误码** | E002 |
| **名称** | InvalidParameterType |
| **类别** | parameter_validation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 400 |
| **恢复模式** | terminate |

**触发条件**：参数类型不符（如 `detail_level` 应为字符串但传入数字）

**错误消息模板**：
```
参数类型错误: {parameter_name}
期望类型: {expected_type}
实际类型: {actual_type}
```

**恢复建议**：对照文档检查类型、使用正确数据类型、利用SDK类型提示

**预防措施**：强类型模型、OpenAPI文档、客户端类型检查

> 示例见 [examples-v2.md](examples-v2.md)

---

### E003: ParameterValueOutOfRange

| 属性 | 值 |
|------|---|
| **错误码** | E003 |
| **名称** | ParameterValueOutOfRange |
| **类别** | parameter_validation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 400 |
| **恢复模式** | terminate |

**触发条件**：参数值超出范围（数值超限、字符串过长、无效枚举值）

**错误消息模板**：
```
参数值超出范围: {parameter_name}
当前值: {current_value}
有效范围: {valid_range}
```

**恢复建议**：查看文档了解范围、调整参数值

**预防措施**：文档标注约束、enum类型限制、客户端预校验

> 示例见 [examples-v2.md](examples-v2.md)

---

### E004: ConflictingParameters

| 属性 | 值 |
|------|---|
| **错误码** | E004 |
| **名称** | ConflictingParameters |
| **类别** | parameter_validation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 400 |
| **恢复模式** | terminate |

**触发条件**：参数间逻辑冲突（如 `detail_level="summary"` 与 `chapters=[1..10]` 矛盾）

**错误消息模板**：
```
参数冲突: {parameter_1} 与 {parameter_2}
冲突原因: {conflict_explanation}
解决方案: {resolution_options}
```

**恢复建议**：理解参数关系、选择一致的组合

**预防措施**：参数兼容性表、SDK冲突检测

> 示例见 [examples-v2.md](examples-v2.md)

---

### E005: InvalidChapterCombination

| 属性 | 值 |
|------|---|
| **错误码** | E005 |
| **名称** | InvalidChapterCombination |
| **类别** | parameter_validation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 400 |
| **恢复模式** | terminate |

**触发条件**：章节组合违反依赖关系（缺少前置章节、仅选附录、空数组）

**错误消息模板**：
```
无效的章节组合: {chapter_list}
依赖关系: {dependencies}
建议组合: {recommended_sets}
```

**恢复建议**：了解章节依赖、补充前置章节、使用 `detail_level` 预设

**预防措施**：章节依赖图、实时校验、智能推荐

> 示例见 [examples-v2.md](examples-v2.md)

---

### E010: InsufficientDataWarning ⚠️ 重要

| 属性 | 值 |
|------|---|
| **错误码** | E010 |
| **名称** | InsufficientDataWarning |
| **类别** | data_source |
| **严重程度** | 🟡 Warning |
| **HTTP 状态码** | 206 |
| **恢复模式** | degrade |

**触发条件**：关键信息缺失但不阻止报告生成（对话过短、缺少决策记录、时间戳不完整）

**错误消息模板**：
```
⚠️ 数据不充分警告
缺失信息: {missing_data_types}
影响程度: {impact_level}
受影响章节: {affected_chapters}
质量评分扣减: {penalty}
```

**恢复建议**：接受降级结果并手动补充、补充信息后重新生成、使用交互模式

**预防措施**：详细对话记录、结构化命令标记关键事件

**quality_score 影响**：

| 缺失信息类型 | 扣分 | 影响章节 |
|-------------|------|---------|
| 决策记录 | -10~-20 | 第4章 |
| 时间信息 | -5~-15 | 第3、8章 |
| 问题记录 | -10~-20 | 第5章 |
| 资源信息 | -5~-10 | 第6章 |
| 协作信息 | 0~-10 | 第7章 |

> 示例见 [examples-v2.md](examples-v2.md)

---

### E011: ConversationHistoryUnavailable

| 属性 | 值 |
|------|---|
| **错误码** | E011 |
| **名称** | ConversationHistoryUnavailable |
| **类别** | data_source |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 503 |
| **恢复模式** | partial |

**触发条件**：无法访问对话历史（服务不可用、权限不足、记录过期）

**错误消息模板**：
```
对话历史不可用: {reason}
可选方案: 手动输入 / 稍后重试 / 本地恢复
```

**恢复建议**：检查权限、使用手动模式、等待重试

**预防措施**：定期备份、持久化存储、分级权限管理

> 示例见 [examples-v2.md](examples-v2.md)

---

### E012: FileAccessDenied

| 属性 | 值 |
|------|---|
| **错误码** | E012 |
| **名称** | FileAccessDenied |
| **类别** | data_source |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 403 |
| **恢复模式** | partial |

**触发条件**：文件读写失败（无权限、路径不存在、文件被锁定、磁盘满）

**错误消息模板**：
```
文件访问被拒绝: {file_path}
操作: {operation} | 原因: {reason}
备选路径: {alternative_paths}
```

**恢复建议**：更换路径、修复权限、检查磁盘空间

**预防措施**：路径可写性检查、默认安全路径、临时文件机制

> 示例见 [examples-v2.md](examples-v2.md)

---

### E021: GoalAnalysisFailed

| 属性 | 值 |
|------|---|
| **错误码** | E021 |
| **名称** | GoalAnalysisFailed |
| **类别** | analysis_engine |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 500 |
| **恢复模式** | estimate |

**触发条件**：目标达成度分析异常（无法提取目标、描述模糊、缺少验收标准）

**错误消息模板**：
```
目标达成度分析失败: {failure_reason}
影响章节: 第2章(目标定义)、第8章(达成度分析)
处理方式: 使用定性描述标注[分析受限]
```

**恢复建议**：接受定性结果、补充目标描述后重试、事后手动编辑

**预防措施**：引导SMART目标、结构化模板、记录目标变更

> 示例见 [examples-v2.md](examples-v2.md)

---

### E022: TimelineReconstructionFailed

| 属性 | 值 |
|------|---|
| **错误码** | E022 |
| **名称** | TimelineReconstructionFailed |
| **类别** | analysis_engine |
| **严重程度** | 🟡 Warning |
| **HTTP 状态码** | 206 |
| **恢复模式** | degrade |

**触发条件**：无法精确重建时间线（缺少时间戳、大量离线操作）

**错误消息模板**：
```
⚠️ 时间线重建精度受限: {reason}
时间精度: {granularity} (约±30%)
影响: 第3、8章时间数据为估算值
```

**恢复建议**：接受粗粒度时间线、手动校正、整合外部记录

**预防措施**：使用`/timer`记录、定期汇报、时间标记

> 示例见 [examples-v2.md](examples-v2.md)

---

### E031: TemplateNotFound

| 属性 | 值 |
|------|---|
| **错误码** | E031 |
| **名称** | TemplateNotFound |
| **类别** | report_generation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 404 |
| **恢复模式** | terminate |

**触发条件**：模板不存在（名称拼写错误、版本不存在、模板被删除）

**错误消息模板**：
```
报告模板不存在: {template_identifier}
可用模板: {available_templates}
```

**恢复建议**：使用默认模板、列出可用模板、创建新模板

**预防措施**：模板自动补全、版本管理、验证测试机制

> 示例见 [examples-v2.md](examples-v2.md)

---

### E032: ReportGenerationTimeout

| 属性 | 值 |
|------|---|
| **错误码** | E032 |
| **名称** | ReportGenerationTimeout |
| **类别** | report_generation |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 504 |
| **恢复模式** | partial |

**触发条件**：报告生成超时（任务复杂、系统负载高、外部服务阻塞）

**错误消息模板**：
```
报告生成超时: 已超过 {timeout_limit}
当前进度: {progress_percentage}%
已完成: {completed_chapters}
```

**恢复建议**：接收部分报告、降低详细程度、缩小范围

**预防措施**：预估生成时间、增量生成、进度显示

> 示例见 [examples-v2.md](examples-v2.md)

---

### E041: InsufficientMemory

| 属性 | 值 |
|------|---|
| **错误码** | E041 |
| **名称** | InsufficientMemory |
| **类别** | system_resource |
| **严重程度** | 🔴 Critical |
| **HTTP 状态码** | 507 |
| **恢复模式** | terminate |

**触发条件**：内存不足（超大对话、并发任务多、内存泄漏）

**错误消息模板**：
```
🔴 致命错误: 内存不足
已用: {used_memory} / {total_memory}
需求: {required_memory}
```

**恢复建议**：释放内存、等待空闲、使用summary模式

**预防措施**：内存监控、并发上限、分块处理、流式处理

> 示例见 [examples-v2.md](examples-v2.md)

---

### E051: ExecutionTimeout

| 属性 | 值 |
|------|---|
| **错误码** | E051 |
| **名称** | ExecutionTimeout |
| **类别** | timeout |
| **严重程度** | 🟠 Error |
| **HTTP 状态码** | 504 |
| **恢复模式** | partial |

**触发条件**：任务执行流程超过全局超时（默认10分钟）

**错误消息模板**：
```
执行超时: 已超过 {global_timeout}
已用时间: {elapsed_time}
当前阶段: {current_phase}
```

**恢复建议**：拆分任务、降低复杂度、申请延长超时

**预防措施**：复杂度评估、任务拆分建议、进度显示

> 示例见 [examples-v2.md](examples-v2.md)

---

## 4. 错误处理策略矩阵

| 严重级别 | 是否中断 | 用户通知 | 默认行为 | 典型错误码 |
|---------|---------|---------|---------|-----------|
| **🔴 Critical** | ✅ 立即终止 | 弹窗+日志+告警 | 返回错误，不生成输出 | E041 |
| **🟠 Error** | ✅ 终止操作 | 提示+日志+建议 | 返回错误，可能部分结果 | E001-E005, E011-E012, E021, E031-E032, E051 |
| **🟡 Warning** | ❌ 继续执行 | 报告内标注+日志 | 降级继续，报告标注警告 | E010, E022 |

**配置项**:
```yaml
error_handling_config:
  strict_mode: false
  allow_partial_output: true
  min_acceptable_quality_score: 60
  retry_policy:
    max_retries: 2
    retryable_errors: [E011, E032, E051]
```

---

## 5. [降级](terminology.md#229-降级-degradation)策略详解

### 5.1 降级执行流程

```
正常流程: 触发检测 → 收集信息 → 分析处理 → 生成报告 → 成功交付(100%)

降级流程: 触发检测 → 收集信息 → ⚠️ Warning(E010) → 生成报告 → 降级交付(85%,带警告)
                                          ↓
                                    标记受影响章节
                                    使用推断/默认值
                                    计算 penalty
```

> 术语定义详见 [terminology.md#229-降级-degradation](terminology.md#229-降级-degradation)

### 5.2 降级时的内容影响

| 受影响方面 | 正常模式 | 降级模式 | 影响程度 |
|-----------|---------|---------|---------|
| **目标达成度分析** | 量化评分 | 定性描述 | 🟡 中等 |
| **时间线精度** | 精确到分钟 | 精确到阶段(±30%) | 🟡 中等 |
| **决策记录完整性** | 含完整rationale | 仅记录结论 | 🟠 较高 |
| **问题分析深度** | 含根因分析 | 仅记录问题方案 | 🟠 较高 |
| **资源使用统计** | 详细清单 | 高层级概述 | 🟢 轻微 |
| **报告质量评分** | 90-100分 | 60-85分 | — |

### 5.3 降级信息报告呈现

**报告头部元信息**:
```markdown
> **质量评分**: 78/100 (B级) ⚠️
> **降级原因**: E010 - 数据不充分
> **受影响章节**: §4 关键决策分析, §8 多维度分析
```

**quality_score 计算**:

```
quality_score = base_score × coverage_factor × confidence_factor

- base_score: 基础分(0-100)
- coverage_factor: 信息覆盖率(0.0-1.0)
- confidence_factor: 置信度系数(0.5-1.0)
```

**质量等级**:

| 分数区间 | 等级 | 建议 |
|---------|------|------|
| 90-100 | A (优秀) | 可直接使用 |
| 80-89 | B (良好) | 快速审阅后使用 |
| 70-79 | C (合格) | 补充关键信息后使用 |
| 60-69 | D (勉强可用) | 大幅修订或重新生成 |
| <60 | F (不建议使用) | 补充数据后重新生成 |

---

## 6. 错误码快速查询表

| 错误码 | 名称 | 说明 | 严重级别 | 用户操作 |
|-------|------|------|---------|---------|
| **E001** | MissingRequiredParameter | 缺少必填参数 | 🟠 Error | 补上必填参数 |
| **E002** | InvalidParameterType | 参数类型不对 | 🟠 Error | 检查参数类型 |
| **E003** | ParameterValueOutOfRange | 参数值超范围 | 🟠 Error | 调整到允许范围 |
| **E004** | ConflictingParameters | 参数冲突 | 🟠 Error | 换兼容组合 |
| **E005** | InvalidChapterCombination | 章节组合无效 | 🟠 Error | 补充前置章节 |
| **E010** | InsufficientDataWarning | 数据不完整 | 🟡 Warning | 补充后重新生成 |
| **E011** | ConversationHistoryUnavailable | 对话记录不可用 | 🟠 Error | 检查权限或手动输入 |
| **E012** | FileAccessDenied | 文件访问被拒 | 🟠 Error | 换有权限路径 |
| **E021** | GoalAnalysisFailed | 目标分析失败 | 🟠 Error | 接受定性描述 |
| **E022** | TimelineReconstructionFailed | 时间线不准 | 🟡 Warning | 手动校正时间 |
| **E031** | TemplateNotFound | 模板不存在 | 🟠 Error | 使用默认模板 |
| **E032** | ReportGenerationTimeout | 生成超时 | 🟠 Error | 使用摘要模式重试 |
| **E041** | InsufficientMemory | 内存不足 | 🔴 Critical | 关闭其他程序 |
| **E051** | ExecutionTimeout | 执行超时 | 🟠 Error | 拆分任务 |

---

## 附录A：错误码分布图

```
E0xx 参数验证 (5个)          E1xx 数据源 (3个)           E2xx 分析引擎 (2个)
┌─────────────────┐          ┌───────────────┐          ┌───────────────┐
│ E001 缺少参数    │          │ E010 数据不足⚠️ │          │ E021 目标分析   │
│ E002 类型错误    │    →     │ E011 历史不可用 │    →     │ E022 时间线不准⚠️│
│ E003 值超范围    │          │ E012 文件权限   │          │               │
│ E004 参数冲突    │          │               │          │               │
│ E005 章节无效    │          │               │          │               │
└─────────────────┘          └───────────────┘          └───────────────┘
                                                                    ↓
                                                        E3xx 报告生成 (2个)    E4xx 资源 (1个)    E5xx 超时 (1个)
                                                        ┌───────────────┐    ┌───────────┐    ┌───────────┐
                                                        │ E031 模板不存在 │    │ E041 内存不足🔴│   │ E051 执行超时│
                                                        │ E032 生成超时   │    │           │    │           │
                                                        └───────────────┘    └───────────┘    └───────────┘
```

## 附录B：HTTP 状态码映射

| HTTP 状态码 | 含义 | 对应的错误码 |
|------------|------|------------|
| 400 Bad Request | 请求参数有问题 | E001-E005 |
| 403 Forbidden | 无权限访问 | E012 |
| 404 Not Found | 资源不存在 | E031 |
| 206 Partial Content | 部分成功 | E010, E022 |
| 500 Internal Server Error | 服务器内部错误 | E021 |
| 503 Service Unavailable | 服务暂不可用 | E011 |
| 504 Gateway Timeout | 网关超时 | E032, E051 |
| 507 Insufficient Storage | 存储空间不足 | E041 |

---

*文档版本: v2.0.0*  
*最后更新: 2026-04-15*  
*适用于 Task Execution Summary Generator v2.0*
