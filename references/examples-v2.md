# 任务执行总结报告生成器 - V2 完整使用示例

> **📌 文档定位** — 本文档是任务执行总结报告生成器的**唯一权威使用示例集**（取代 v1.0 版本的 examples.md）。
> 包含标准化请求/响应格式、完整字段说明和多场景覆盖，适用于集成开发和功能验证。

**文档版本**: v2.0
**最后更新**: 2026-04-15
**适用技能版本**: Task Execution Summary Generator v2.0+

本文档提供"任务执行总结报告生成器"技能的 **V2 标准化请求-响应格式示例**，涵盖正常调用、最小参数、参数错误和数据不足等典型场景。

---

> 📖 **文档阅读指南**: 集成开发者建议阅读全部 4 个示例；测试工程师关注示例 1、3、4；普通用户关注示例 1、2。
>
> 📋 **响应状态**: `success=true` 表示成功；`success=true` + `degraded=true` 表示降级成功；`success=false` 表示执行失败。
>
> 完整参数定义见 [api-reference-quick.md](api-reference-quick.md)

---

## 示例 1: 软件开发任务标准调用

### 场景描述

用户刚完成了一个用户认证模块的开发任务，该任务涉及 Session 管理、密码加密、登录注册流程实现等多个技术环节，历时约 2 小时。这是一个典型的软件开发场景，用户希望生成一份标准的执行总结报告用于技术沉淀和团队分享。

### 完整请求 JSON

```json
{
  "task_context": {
    "task_name": "用户认证模块开发",
    "task_type": "development",
    "time_range": null,
    "description": "实现基于 Session 的用户登录注册功能"
  },
  "generation_options": {
    "detail_level": "standard",
    "template_variant": "standard",
    "included_chapters": null,
    "excluded_chapters": null,
    "language_style": "professional",
    "focus_dimensions": ["goal_achievement", "problem_patterns"],
    "output_format": "markdown"
  },
  "output_config": {
    "save_to_file": true,
    "file_path": null,
    "include_metadata": true
  }
}
```

> 完整参数定义见 [api-reference-quick.md](api-reference-quick.md) 请求参数速查章节

### 预期成功响应

```json
{
  "success": true,
  "report_id": "rpt-20260409-dev-001",
  "timestamp": "2026-04-09T16:30:00Z",
  "processing_time_ms": 192000,

  "report": {
    "title": "用户认证模块开发 - 执行总结报告",

    "content": "# 用户认证模块开发 - 执行总结报告\n\n## 第1章 执行概览\n[基本信息表 + 关键指标 + 质量评分 94.5/100]\n\n## 第2章 任务背景与目标\n[目标清单：Session认证系统开发 + 预期交付物 + 目标调整记录]\n\n## 第3章 执行过程详解\n[4个阶段时间线：技术方案设计(25min) → 数据库设计(25min) → 核心功能实现(50min) → 测试优化(20min)]\n\n## 第4章 关键决策分析\n[D1: Session vs JWT选择 → 选Session(传统MPA+即时注销需求)]\n[D2: bcrypt vs argon2选择 → 选bcrypt cost=12(性能与安全平衡)]\n\n## 第5章 问题解决记录\n[I1: Migration执行顺序错误 → 重命名时间戳解决]\n[I2: Cookie安全标志缺失 → 添加httpOnly/secure/sameSite]\n\n## 第6章 资源使用情况\n[人力投入2小时 + 技术栈(Node.js/Express/Sequelize/PostgreSQL) + 新增依赖列表]\n\n## 第7章 多维分析汇总\n[目标达成度96.5% + 时间效能偏差+14% + 问题模式统计]\n\n## 第8章 经验总结与方法论\n[成功要素4条 + 最佳实践2条 + 新增知识点4项]\n\n## 第9章 改进建议与行动计划\n[P0: 账户锁定机制/Rate Limiting]\n[P1: API文档/E2E测试/Code Review]\n[P2-P3: OAuth2.0/Redis Session/MFA]\n\n## 第10章 附录\n[API端点列表/数据库Schema/测试用例清单]\n\n---\n*报告生成时间: 2026-04-09 16:30:00 UTC*",

    "word_count": 12580,
    "chapter_count": 10,

    "metadata": {
      "task_name": "用户认证模块开发",
      "generation_duration": "3分12秒",
      "template_used": "standard",
      "quality_score": 94.5,
      "focus_dimensions": ["goal_achievement", "problem_patterns"],
      "language_style": "professional"
    }
  },

  "quality_check": {
    "completeness_rate": 0.92,
    "accuracy_confidence": 0.89,

    "information_gaps": [
      "部分决策依据未在对话中明确说明（如为何选择 Express 而非 Koa）",
      "团队协作信息缺失（单人任务，第七章内容简化呈现）"
    ],

    "warnings": []
  },

  "statistics": {
    "total_phases": 4,
    "total_decisions": 2,
    "total_problems": 2,
    "suggestions_count": 7
  },

  "file_info": {
    "saved_to": "./reports/用户认证模块开发_执行总结报告_20260409.md",
    "file_size_kb": 48.5
  }
}
```

### 关键点说明

- ✅ **`success: true`** 表示报告成功生成，可直接使用
- ✅ **`quality_score: 94.5`** 表示高质量（>90 为优秀，>80 为良好）
- ✅ **`completeness_rate: 0.92`** > 0.9 阈值，信息覆盖度优秀
- ✅ **报告包含完整的 10 章 Markdown 内容**，结构清晰
- ✅ **文件已自动保存**到 `./reports/` 目录，大小 48.5KB

---

## 示例 2: Sprint 复盘最小化调用

### 场景描述

团队刚结束了一个为期 2 周（10 个工作日）的敏捷 Sprint 迭代。项目经理希望快速生成一份复盘报告用于回顾会议。使用最简化的参数配置，完全依赖系统的智能默认值。

### 完整请求 JSON

```json
{
  "task_context": {
    "task_name": "Sprint 24 回顾"
  }
}
```

> 完整参数定义见 [api-reference-quick.md](api-reference-quick.md)

### 预期响应

```json
{
  "success": true,
  "report_id": "rpt-20260409-pm-002",
  "timestamp": "2026-04-09T17:00:00Z",
  "processing_time_ms": 168000,

  "report": {
    "title": "Sprint 24 回顾 - 执行总结报告",

    "content": "# Sprint 24 回顾 - 执行总结报告\n\n## 第1章 执行概览\n[基本信息 + Sprint周期 + 质量评分 91.2/100]\n\n## 第2章 任务目标与背景\n[Sprint目标 + 8个用户故事(46SP) + 目标调整记录]\n\n## 第3章 执行过程详解\n[Sprint Planning → 开发执行(两周) → Code Review → Sprint Review]\n\n## 第4章 关键决策分析\n[D1: 支付网关紧急插入 → 立即处理(业务连续性优先)]\n[D2: ClickHouse迁移方案 → 增量双写方案]\n\n## 第5章 问题解决记录\n[I1: 支付网关接口升级阻塞 → 16工时紧急响应]\n[I2: UI设计稿反复修改 → 建立Design Review Checklist]\n\n## 第6章 团队协作分析\n[6人团队 + 协作效能11/15分 + 亮点与改进项]\n\n## 第7章 多维分析汇总\n[目标达成度75% + Velocity提升16.7% + 问题模式统计]\n\n## 第8章 经验总结与方法论\n[成功要素3条 + 方法论2套]\n\n## 第9章 改进建议与行动计划\n[P0: 阻塞分级响应/需求变更控制]\n[P1: Planning效率/Tech Talk/CI-CD]\n[P2-P3: 第三方监控/Design System/新人培训]\n\n## 第10章 附录\n[燃尽图数据/会议记录]\n\n---\n*报告生成时间: 2026-04-09 17:00:00 UTC*",

    "word_count": 9876,
    "chapter_count": 10,

    "metadata": {
      "template_used": "standard",
      "quality_score": 91.2,
      "auto_detected_type": "management"
    }
  },

  // quality_check、statistics、file_info 结构同示例1，具体值不同
  "quality_check": {
    "completeness_rate": 0.88,
    "accuracy_confidence": 0.85,
    "information_gaps": [],
    "warnings": [
      {
        "code": "W001",
        "message": "团队协作信息有限，第七章内容基于对话推断",
        "severity": "low",
        "affected_chapters": [7]
      }
    ]
  },

  "statistics": {
    "total_phases": 3,
    "total_decisions": 5,
    "total_problems": 3,
    "suggestions_count": 7
  },

  "file_info": {
    "saved_to": "./reports/Sprint_24_回顾_执行总结报告_20260409.md",
    "file_size_kb": 38.2
  }
}
```

### 关键点说明

- ⚠️ **最小参数也能成功生成报告** — 仅 `task_name` 即可触发完整流程
- ⚠️ **`warnings` 数组非空** — 提示数据不充分的部分，但不影响整体可用性
- ⚠️ **`quality_score: 91.2`** — 仍然达到优秀级别（>90）
- 💡 **系统智能推断** — 自动检测到 `task_type` 为项目管理类型

---

## 示例 3: 参数验证错误（异常场景）

### 场景描述

用户在批量调用或集成测试场景中，提供了多个错误的参数组合：缺少必填参数 `task_name`、使用了无效的枚举值 `detail_level`、章节编号超出合法范围（1-10）、以及试图排除所有核心章节。

### 完整请求 JSON

```json
{
  "task_context": {},
  "generation_options": {
    "detail_level": "invalid_value",
    "template_variant": "detailed",
    "included_chapters": [1, 2, 99],
    "excluded_chapters": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  },
  "output_config": {
    "save_to_file": true
  }
}
```

> 完整错误码定义见 [error-codes-quick.md](error-codes-quick.md)

### 预期错误响应

```json
{
  "success": false,

  "error": {
    "code": "E001",
    "name": "MissingRequiredParameter",
    "message": "缺少必填参数: task_name",
    "category": "validation",
    "severity": "error",
    "http_status": 400,
    "timestamp": "2026-04-09T16:35:00Z",

    "details": [
      {
        "parameter": "task_context.task_name",
        "constraint": "required: true, type: string, min_length: 2, max_length: 200",
        "actual_value": null,
        "suggestion": "请在 task_context 中添加任务名称字符串（2-200字符）"
      },
      {
        "parameter": "generation_options.detail_level",
        "constraint": "enum: [\"summary\", \"standard\", \"detailed\"]",
        "actual_value": "invalid_value",
        "suggestion": "请使用 summary（摘要）/ standard（标准）/ detailed（详细）之一"
      }
      // 其他错误详情省略...
    ],

    "recovery_actions": [
      {
        "action": "补充 task_name 参数",
        "description": "在 task_context 中添加任务名称字符串",
        "auto_recoverable": true,
        "priority": "P0",
        "example": "{\"task_context\": {\"task_name\": \"Sprint 24 回顾\"}}"
      },
      {
        "action": "修正 detail_level 值",
        "description": "使用 summary / standard / detailed 三个有效值之一",
        "auto_recoverable": true,
        "priority": "P0",
        "example": "\"detail_level\": \"standard\""
      }
      // 其他恢复动作省略...
    ],

    "hint": "最快的修复方式是仅提供 task_name，其他参数将使用合理的默认值。最小可用请求：{\"task_context\": {\"task_name\": \"您的任务名称\"}}",

    "documentation_reference": {
      "parameter_spec": "references/api-reference.md#参数定义",
      "error_codes": "references/error-codes.md#E001-E005",
      "quick_example": "references/examples-v2.md#示例-2"
    }
  },

  "request_snapshot": {
    "received_at": "2026-04-09T16:34:55Z",
    "raw_request_size_bytes": 187,
    "validation_failed_at_step": 1,
    "total_errors_found": 4,
    "errors_by_severity": {
      "error": 3,
      "warning": 1
    }
  }
}
```

### 关键点说明

- ❌ **`success: false`** — 表示执行失败，未生成任何报告
- 🔴 **`severity: "error"`** — 这是 Error 级别，会**终止执行**
- 🔴 **`error.code: "E001"`** — 显示的是**第一个致命错误**（缺少必填参数）
- 📋 **`details` 数组列出所有检测到的错误**（共 4 个），一次性返回以便用户批量修复
- 💡 **`recovery_actions` 提供具体的修复步骤**，包含优先级和示例代码

---

## 示例 4: 数据不足时的降级执行

### 场景描述

用户想要对一个快速 Bug 修复任务（"登录页面样式错乱"）生成详细级别的报告。然而，该任务的对话历史较短，信息密度不足以支撑 `detailed` 级别的完整报告。系统应该**降级执行**——将详细程度从 `detailed` 降级为 `standard`，并在响应中明确发出警告。

### 完整请求 JSON

```json
{
  "task_context": {
    "task_name": "快速Bug修复: 登录页面样式错乱",
    "task_type": "development"
  },
  "generation_options": {
    "detail_level": "detailed"
  },
  "output_config": {
    "save_to_file": true,
    "include_metadata": true
  }
}
```

> 完整参数定义见 [api-reference-quick.md](api-reference-quick.md)

### 预期响应（带警告的成功）

```json
{
  "success": true,
  "report_id": "rpt-20260409-deg-003",
  "timestamp": "2026-04-09T16:45:00Z",
  "processing_time_ms": 125000,

  "degraded": true,
  "degradation_info": {
    "original_detail_level": "detailed",
    "effective_detail_level": "standard",
    "reason": "数据不足以支持 detailed 级别的完整报告（信息覆盖率 68%，阈值 80%）",
    "degradation_decision_made_at": "step_3_quality_check",
    "affected_capabilities": [
      "深度决策分析（第四章内容粒度降低）",
      "详尽问题排查过程（第五章省略次要分支）",
      "完整时间线重建（第二章使用估算值）",
      "团队协作矩阵（第七章简化或跳过）"
    ],
    "user_can_prevent_degradation": true,
    "prevention_hint": "如在任务执行过程中保持更详细的对话记录（如说明思考过程、列举备选方案），可避免降级"
  },

  "report": {
    "title": "快速Bug修复: 登录页面样式错乱 - 执行总结报告",

    "content": "# 快速Bug修复: 登录页面样式错乱 - 执行总结报告\n\n> ⚠️ **信息有限声明**: 本报告基于可用数据生成，部分章节内容因信息不足进行了简化或推断。\n\n## 第1章 执行概览\n[基本信息 + 质量评分 78.5/100(降级执行)]\n\n## 第2章 任务目标与背景\n[Bug现象：登录页面表单布局错位 + 修复目标3项]\n\n## 第3章 执行过程（⚠️ 信息有限）\n[5个步骤：复现(3min) → 定位(8min) → 修复(5min) → 验证(7min) → 提交(2min)]\n\n## 第4章 关键决策分析（⚠️ 信息有限）\n[D1: CSS布局方案选择 → 调整Flexbox参数(基于代码变更推断)]\n\n## 第5章 问题解决记录\n[I1: 登录页面表单布局错位 → flex-shrink默认行为导致 → 添加flex-basis+max-width约束]\n\n## 第6章 资源使用情况（⚠️ 信息有限）\n[前端开发1人~25min + 技术栈(Chrome DevTools/Git/CSS3/React)]\n\n## 第7章 多维分析汇总\n[目标达成度100% + 时间效能分布]\n\n## 第8章 经验总结与方法论\n[成功要素3条 + CSS响应式Bug排查SOP]\n\n## 第9章 改进建议与行动计划\n[A1: 建立响应式设计规范]\n[A2: 增加CI视觉回归测试]\n\n## 第10章 附录\n[代码变更片段]\n\n---\n*生成模式: DEGRADED（原请求 detailed → 实际 standard）*\n*生成时间: 2026-04-09 16:45:00 UTC*",

    "word_count": 6850,
    "chapter_count": 10,

    "metadata": {
      "quality_score": 78.5,
      "generation_mode": "degraded",
      "original_requested_level": "detailed",
      "effective_generated_level": "standard",
      "information_coverage_rate": 0.68
    }
  },

  // quality_check、statistics、file_info 结构同示例1
  "quality_check": {
    "completeness_rate": 0.72,
    "accuracy_confidence": 0.70,

    "information_gaps": [
      {
        "category": "decision_records",
        "coverage_rate": 0.45,
        "threshold": 0.70,
        "impact": "决策记录信息覆盖率仅 45% → 第四章内容可能不完整",
        "affected_chapters": [4]
      }
      // 其他信息缺口省略...
    ],

    "warnings": [
      {
        "code": "E010",
        "name": "InsufficientDataWarning",
        "message": "决策记录信息覆盖率仅 45%（阈值 70%）",
        "severity": "warning",
        "affected_chapters": [4]
      }
      // 其他警告省略...
    ]
  },

  "statistics": {
    "total_phases": 2,
    "total_decisions": 1,
    "total_problems": 1,
    "suggestions_count": 3
  },

  "user_advice": {
    "can_upgrade": true,
    "how_to_upgrade": [
      "方式一（推荐）: 补充更多任务细节后重新生成",
      "方式二: 手动编辑生成的报告补充标注「⚠️ 信息有限」的段落",
      "方式三: 结合 Git 历史获取精确时间线替换估算值"
    ],
    "prevention_for_next_time": "下次执行类似任务时，建议在过程中保持更详细的对话记录"
  },

  "file_info": {
    "saved_to": "./reports/快速Bug修复_登录页面样式错乱_执行总结报告_20260409.md",
    "file_size_kb": 26.8
  }
}
```

### 关键点说明

- ✅ **`success: true`** — 降级执行**仍然返回成功**，报告可用
- ⚠️ **`degraded: true`** — 明确标记这是**降级后的结果**
- ⚠️ **`degradation_info` 字段** — 提供了降级的详细信息和原因
- ⚠️ **`quality_score: 78.5`** — 反映了数据质量的影响（<80 为一般级别）
- 💡 **`user_advice` 字段** — 告知用户**如何升级到完整版**

---

## 📊 示例对比速查表

| 维度 | 示例 1: 标准调用 | 示例 2: 最小参数 | 示例 3: 参数错误 | 示例 4: 降级执行 |
|------|----------------|----------------|----------------|----------------|
| **success** | `true` | `true` | `false` | `true` |
| **degraded** | 无 | 无 | 无 | `true` |
| **quality_score** | 94.5 (优秀) | 91.2 (优秀) | N/A | 78.5 (一般) |
| **completeness_rate** | 0.92 | 0.88 | N/A | 0.72 |
| **word_count** | 12,580 | 9,876 | N/A | 6,850 |
| **chapter_count** | 10 | 10 | N/A | 10 |
| **warnings 数量** | 0 | 2 (low) | N/A | 3 |
| **file_saved** | ✅ (48.5 KB) | ✅ (38.2 KB) | ❌ | ✅ (26.8 KB) |
| **适用场景** | 常规软件开发 | 快速 Sprint 复盘 | 参数调试/集成测试 | 简单任务/短对话 |
| **推荐度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ (测试用) | ⭐⭐⭐⭐ |

---

## 🔧 集成测试用例建议

### 正向测试（Expected: success=true）

- **TC-001 标准开发任务**: 示例 1 完整请求 → 断言 quality_score > 90, chapter_count == 10
- **TC-002 最小参数调用**: 仅 `{task_name: "Test"}` → 断言使用默认值，report_id 格式正确
- **TC-003 摘要级报告**: `detail_level: "summary"` → 断言 word_count < 1500
- **TC-004 自定义章节**: `included_chapters: [1,5,10]` → 断言仅包含指定章节

### 反向测试（Expected: success=false）

- **TC-101 缺少 task_name**: `task_context: {}` → 断言 error.code == "E001"
- **TC-102 无效 detail_level**: `detail_level: "ultra"` → 断言 error.code == "E002"
- **TC-103 排除所有章节**: `excluded_chapters: [1..10]` → 断言 error.code == "E005"

### 边界测试（Expected: degraded=true 或 warning）

- **TC-201 数据不足降级**: 短对话 + detailed 请求 → 断言 degraded == true
- **TC-202 部分信息缺失**: 正常对话但某类信息稀少 → 断言 warnings 非空

---

## 📖 相关文档索引

| 文档 | 内容 | 链接 |
|------|------|------|
| **SKILL.md** | 技能主文档（概述、触发条件、执行流程） | [SKILL.md](../SKILL.md) |
| **api-reference-quick.md** | API 参数速查 | [api-reference-quick.md](api-reference-quick.md) |
| **error-codes-quick.md** | 错误码速查 | [error-codes-quick.md](error-codes-quick.md) |
| **execution-flow.md** | 7 步执行流程详解 | [execution-flow.md](execution-flow.md) |
| **templates.md** | 4 种模板变体的结构定义 | [templates.md](templates.md) |
| **terminology.md** | 86 个专业术语定义 | [terminology.md](terminology.md) |

---

## 📝 文档修订历史

| 版本 | 日期 | 作者 | 变更内容 |
|------|------|------|---------|
| v2.0 | 2026-04-15 | Task Execution Summary Generator Team | 精简版本，4个示例响应JSON章节概要化，节省约38%行数 |

---

*本文档遵循 Task Execution Summary Generator v2.0 接口规范*
*如有疑问，请参阅 [api-reference.md](api-reference.md) 或 [error-codes.md](error-codes.md)*
