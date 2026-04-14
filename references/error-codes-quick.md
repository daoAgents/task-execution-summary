---
version: "1.0-quick"
updated: "2026-04-15"
stage: production
skill_version: "v1.0+"
---

# 错误码速查表

> **完整错误码详解请查看 [error-codes.md](error-codes.md)**

---

## 错误码分类总览

| 类别编码 | 类别名称 | 错误码范围 | 严重级别 | 处理策略 |
|---------|---------|-----------|---------|---------|
| **E0xx** | 参数验证错误 | E001-E005 | Error | 验证失败立即返回 |
| **E1xx** | 数据源错误 | E010-E012 | Error/Warning | 重试或降级 |
| **E2xx** | 分析引擎错误 | E021-E022 | Error/Warning | 使用默认值或跳过 |
| **E3xx** | 报告生成错误 | E031-E032 | Error | 回退简化模板 |
| **E4xx** | 系统资源错误 | E041 | Critical | 终止并告警 |
| **E5xx** | 超时错误 | E051 | Error | 终止或返回部分结果 |

**严重级别说明**：🔴 Critical（系统故障）| 🟠 Error（功能错误）| 🟡 Warning（非致命问题）

---

## 参数验证错误 (E0xx)

| 错误码 | 名称 | 描述 | 快速修复 |
|--------|------|------|---------|
| **E001** | MissingRequiredParameter | 缺少必填参数 | 补充 `task_name` 等必填参数 |
| **E002** | InvalidParameterType | 参数类型错误 | 检查参数类型（如字符串vs数字） |
| **E003** | ParameterValueOutOfRange | 参数值越界 | 调整值到有效范围（如章节1-10） |
| **E004** | ConflictingParameters | 参数冲突 | 检查参数间逻辑一致性 |
| **E005** | InvalidChapterCombination | 无效章节组合 | 确保章节依赖关系满足 |

---

## 数据源错误 (E1xx)

| 错误码 | 名称 | 描述 | 快速修复 |
|--------|------|------|---------|
| **E010** ⚠️ | InsufficientDataWarning | 数据不充分警告 | 接受降级结果或补充信息后重试 |
| **E011** | ConversationHistoryUnavailable | 对话历史不可用 | 使用手动输入模式提供信息 |
| **E012** | FileAccessDenied | 文件访问被拒绝 | 更换输出路径或检查权限 |

---

## 分析引擎错误 (E2xx)

| 错误码 | 名称 | 描述 | 快速修复 |
|--------|------|------|---------|
| **E021** | GoalAnalysisFailed | 目标达成度分析失败 | 提供更具体的 SMART 目标描述 |
| **E022** ⚠️ | TimelineReconstructionFailed | 时间线重建精度受限 | 接受粗粒度时间或手动校正 |

---

## 报告生成错误 (E3xx)

| 错误码 | 名称 | 描述 | 快速修复 |
|--------|------|------|---------|
| **E031** | TemplateNotFound | 报告模板不存在 | 使用默认模板或检查模板名称 |
| **E032** | ReportGenerationTimeout | 报告生成超时 | 降低详细程度或获取部分报告 |

---

## 系统资源错误 (E4xx)

| 错误码 | 名称 | 描述 | 快速修复 |
|--------|------|------|---------|
| **E041** 🔴 | InsufficientMemory | 内存不足 | 关闭其他应用或使用 summary 模式 |

---

## 超时错误 (E5xx)

| 错误码 | 名称 | 描述 | 快速修复 |
|--------|------|------|---------|
| **E051** | ExecutionTimeout | 执行超时 | 简化任务范围或使用异步接口 |

---

## 错误响应结构速查

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
    "timestamp": "2026-04-09T14:30:00Z",
    "request_id": "req_abc123xyz",
    "context": { /* 错误上下文 */ },
    "recovery": {
      "mode": "terminate",
      "suggestions": ["修复建议1", "修复建议2"]
    }
  }
}
```

---

## 恢复模式说明

| 模式 | 说明 | 适用错误 |
|------|------|---------|
| **terminate** | 终止执行，返回错误 | E001-E005, E031, E041 |
| **degrade** | 降级继续，标注影响 | E010, E022 |
| **partial** | 返回部分结果 | E011, E012, E032, E051 |
| **estimate** | 使用估算值 | E021 |

---

## 质量评分影响速查

| 缺失信息类型 | 分数扣减 | 影响章节 |
|-------------|---------|---------|
| 决策记录缺失 | -10 至 -20 | 第4章 |
| 时间信息不全 | -5 至 -15 | 第3、8章 |
| 问题记录模糊 | -10 至 -20 | 第5章 |
| 资源信息缺失 | -5 至 -10 | 第6章 |
| 协作信息缺失 | -0 至 -10 | 第7章 |

---

## 常见错误场景对照

| 场景 | 可能错误码 | 推荐处理 |
|------|-----------|---------|
| 忘记指定任务名称 | E001 | 补充 `task_context.task_name` |
| 章节编号写错 | E003 | 修正为 1-10 范围内的数字 |
| 对话历史太短 | E010 | 接受降级或补充信息 |
| 输出到系统目录 | E012 | 更换为用户目录路径 |
| 任务过于复杂 | E032/E051 | 使用 `detail_level: "summary"` |

---

*本文档为快速参考版，完整信息请参阅 [error-codes.md](error-codes.md)*
