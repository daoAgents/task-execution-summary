# 任务执行总结报告生成器 - API 接口参考文档

> **文档版本**: v2.0
>
> **最后更新**: 2026-04-09
>
> **适用技能版本**: Task Execution Summary Generator v2.0
>
> **维护者**: Task Execution Summary Generator Team

---

## 目录

- [1. 概述](#1-概述)
  - [1.1 文档目的与适用范围](#11-文档目的与适用范围)
  - [1.2 接口版本信息](#12-接口版本信息)
  - [1.3 基础 URL 与调用方式](#13-基础-url-与调用方式)
  - [1.4 认证方式](#14-认证方式)
- [2. 输入参数完整定义](#2-输入参数完整定义)
  - [2.1 task_context 对象（必填）](#21-task_context-对象必填)
  - [2.2 generation_options 对象（可选）](#22-generation_options-对象可选)
  - [2.3 output_config 对象（可选）](#23-output_config-对象可选)
- [3. 输出响应格式定义](#3-输出响应格式定义)
  - [3.1 成功响应（HTTP 200）](#31-成功响应http-200)
  - [3.2 错误响应（HTTP 4xx/5xx）](#32-错误响应http-4xx5xx)
- [4. 参数验证规则表](#4-参数验证规则表)
- [5. 调用示例](#5-调用示例)
  - [5.1 示例一：最小调用（仅 task_name）](#51-示例一最小调用仅-task_name)
  - [5.2 示例二：标准调用（常用配置组合）](#52-示例二标准调用常用配置组合)
  - [5.3 示例三：完全配置调用（所有参数都指定）](#53-示例三完全配置调用所有参数都指定)
- [6. 版本历史与变更记录](#6-版本历史与变更记录)

---

## 1. 概述

### 1.1 文档目的与适用范围

**文档目的**

本文档为"任务执行总结报告生成器"技能提供完整的 API 接口规范说明，旨在帮助开发者、集成商和高级用户：

- 理解接口的完整功能能力和参数体系
- 准确构建符合规范的请求报文
- 正确解析和处理响应数据
- 进行系统集成和自动化工作流开发
- 实现定制化的报告生成解决方案

**适用范围**

本接口适用于以下场景：

| 场景类型 | 典型用例 | 说明 |
|---------|---------|------|
| 软件开发 | 功能开发完成、Bug修复、技术重构 | 技术方案沉淀、问题排查记录 |
| 项目管理 | Sprint结束、里程碑达成、项目收尾 | 进度复盘、资源评估 |
| 运维排查 | 故障处理、性能优化、安全加固 | 排查流程标准化、预防措施 |
| 技术研究 | 技术选型、POC验证、架构设计 | 决策依据、技术对比 |
| 学习成长 | 课程学习、技能培训、认证备考 | 知识体系构建、学习方法论 |

**核心能力概述**

本接口基于四大核心引擎协同工作：

1. **信息收集引擎 (Information Collection Engine)**：从对话历史和相关文件中全面提取任务执行的关键信息
2. **分析处理引擎 (Analysis Processing Engine)**：对收集到的信息进行深度分析和多维度评估
3. **报告生成引擎 (Report Generation Engine)**：按照规范模板将分析结果转化为结构化 Markdown 报告
4. **智能推荐引擎 (Intelligent Recommendation Engine)**：生成针对性的改进建议和可复用的方法论

### 1.2 接口版本信息

```
API 版本: v2.0
协议: RESTful API / JSON-RPC
数据格式: JSON (请求) / Markdown 或 JSON (响应)
编码: UTF-8
兼容性: 向后兼容
```

**版本策略**：

- 主版本号（Major）：不兼容的 API 变更
- 次版本号（Minor）：向后兼容的功能新增
- 修订号（Patch）：向后兼容的问题修复

### 1.3 基础 URL 与调用方式

**基础 URL 结构**

```
生产环境: https://api.task-execution-summary.com/v1
测试环境: https://staging-api.task-execution-summary.com/v1
本地开发: http://localhost:8080/v1
```

**主要端点 (Endpoint)**

| 端点路径 | HTTP 方法 | 功能描述 | 认证要求 |
|---------|-----------|---------|---------|
| `/generate` | POST | 生成任务执行总结报告 | 可选 |
| `/generate/async` | POST | 异步生成报告（适用于复杂任务） | 可选 |
| `/status/{report_id}` | GET | 查询异步任务状态 | 可选 |
| `/templates` | GET | 获取可用模板列表 | 无需认证 |
| `/validate` | POST | 验证请求参数合法性 | 无需认证 |

**调用方式示例**

```bash
# 同步调用
curl -X POST https://api.task-execution-summary.com/v1/generate \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "task_context": {
      "task_name": "用户认证模块开发"
    }
  }'

# 异步调用（推荐用于大型任务）
curl -X POST https://api.task-execution-summary.com/v1/generate/async \
  -H "Content-Type: application/json" \
  -d '{
    "task_context": {
      "task_name": "Sprint 24 回顾",
      "task_type": "management"
    },
    "generation_options": {
      "detail_level": "detailed"
    }
  }'
```

### 1.4 认证方式

**认证机制**

本接口支持以下认证方式：

| 认证方式 | 适用场景 | 安全级别 | 说明 |
|---------|---------|---------|------|
| API Key | 服务端集成、自动化脚本 | 中等 | 适合内部系统和可信环境 |
| OAuth 2.0 / JWT | 企业级应用、多租户系统 | 高 | 支持用户身份验证和权限控制 |
| 无认证 | 本地开发、公开演示 | 低 | 仅限测试和非敏感场景 |

**API Key 认证**

```http
Authorization: Bearer your_api_key_here
```

或在查询参数中：

```
?api_key=your_api_key_here
```

**获取 API Key**

1. 注册开发者账号
2. 在控制台创建应用
3. 生成 API Key（支持设置过期时间和权限范围）
4. 在请求头中携带 API Key

**速率限制 (Rate Limiting)**

| 计费层级 | 限制 | 说明 |
|---------|------|------|
| 免费版 | 100 次/小时 | 适合个人学习和测试 |
| 专业版 | 1000 次/小时 | 适合团队日常使用 |
| 企业版 | 自定义 | 适合大规模集成 |

超限返回 `429 Too Many Requests`，响应头包含：

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000
```

---

## 2. 输入参数完整定义

### 2.1 task_context 对象（必填）

task_context 对象是请求的核心，包含任务的基本信息和上下文数据。

#### 2.1.1 task_name

- **类型**: string
- **必填**: 是 ✅
- **默认值**: 无
- **描述**: 任务名称或标题，用于标识任务并生成报告标题
- **约束条件**:
  - 最小长度: 2 字符
  - 最大长度: 200 字符
  - 不允许纯空格或特殊控制字符
  - 支持中文、英文、数字及常见标点符号
- **示例值**:
  ```json
  "用户认证模块开发"
  ```
  ```json
  "Sprint 24 回顾"
  ```
  ```json
  "Docker容器化学习"
  ```
- **使用场景**: 必须提供，用于生成报告标题和标识任务
- **最佳实践**: 使用简洁明确的名称，避免过于笼统（如"任务1"）

#### 2.1.2 task_type

- **类型**: enum (枚举字符串)
- **必填**: 否
- **默认值**: `"auto-detect"` （自动检测）
- **可选值**:

  | 值 | 说明 | 影响范围 |
  |---|------|---------|
  | `development` | 软件开发类任务 | 强化技术决策、代码质量、问题解决维度分析权重 |
  | `management` | 项目管理类任务 | 强化进度管理、资源分配、团队协作维度分析权重 |
  | `operations` | 运维排查类任务 | 强化问题诊断、根因分析、预防措施维度分析权重 |
  | `research` | 技术研究类任务 | 强化技术对比、方案论证、创新发现维度分析权重 |
  | `learning` | 学习成长类任务 | 强化知识掌握、学习方法、能力提升维度分析权重 |
  | `auto-detect` | 自动从上下文推断（默认） | 智能识别任务特征并匹配最合适的分析模型 |

- **描述**: 任务类型分类，影响分析维度权重和报告模板选择
- **何时指定**: 当自动检测可能不准确时建议手动指定；对于混合型任务（如既包含开发又包含管理），选择主导类型
- **示例值**:
  ```json
  "development"
  ```

#### 2.1.3 time_range 对象（可选）

time_range 对象用于指定任务的执行时间范围，帮助系统准确计算时间效能指标。

##### start_time

- **类型**: datetime (ISO 8601 格式)
- **必填**: 否（但建议提供以提高时间分析的准确性）
- **默认值**: 从对话历史中自动提取第一个有效时间戳
- **描述**: 任务开始的精确时间点
- **格式要求**: `YYYY-MM-DDTHH:mm:ssZ` 或 `YYYY-MM-DDTHH:mm:ss+HH:mm`
- **示例值**:
  ```json
  "2026-04-01T09:00:00+08:00"
  ```
  ```json
  "2026-04-01T09:00:00Z"
  ```

##### end_time

- **类型**: datetime (ISO 8601 格式)
- **必填**: 否（但建议提供）
- **默认值**: 从对话历史中自动提取最后一个有效时间戳或当前时间
- **描述**: 任务结束的精确时间点
- **约束条件**: 必须晚于 start_time（如果两者都提供）
- **示例值**:
  ```json
  "2026-04-01T17:30:00+08:00"
  ```

**完整 time_range 示例**:

```json
{
  "start_time": "2026-04-01T09:00:00+08:00",
  "end_time": "2026-04-01T17:30:00+08:00"
}
```

#### 2.1.4 description

- **类型**: string
- **必填**: 否
- **默认值**: 从对话历史中自动提取摘要
- **描述**: 任务的简要描述，帮助理解任务背景和目标
- **约束条件**:
  - 最小长度: 10 字符
  - 最大长度: 2000 字符
- **示例值**:
  ```json
  "开发一个基于JWT的用户认证模块，支持登录、注册、密码重置和Token刷新功能，集成OAuth2.0第三方登录"
  ```

#### 2.1.5 participants 数组（可选）

- **类型**: object 数组
- **必填**: 否
- **默认值**: 空（表示单人任务）
- **描述**: 参与任务的人员列表及其角色信息
- **数组元素结构**:

  ```json
  {
    "name": "string",           // 参与者姓名或代号
    "role": "string",           // 角色（如：开发者、测试、产品经理、运维）
    "responsibility": "string"  // 主要职责描述（可选）
  }
  ```

- **约束条件**: 最多支持 50 个参与者
- **示例值**:

  ```json
  [
    {"name": "张伟", "role": "Tech Lead", "responsibility": "架构设计与核心开发"},
    {"name": "李娜", "role": "前端开发", "responsibility": "UI实现与交互"},
    {"name": "王强", "role": "QA工程师", "responsibility": "测试用例设计与执行"}
  ]
  ```

#### 2.1.6 context_data 对象（可选）

- **类型**: object
- **必填**: 否
- **默认值**: 空
- **描述**: 额外的上下文数据，可用于补充系统无法自动提取的信息
- **支持的子字段**:

  | 字段名 | 类型 | 说明 | 示例 |
  |-------|------|------|------|
  | `objectives` | array | 任务目标列表 | `["完成用户登录功能", "通过安全审计"]` |
  | `constraints` | array | 约束条件列表 | `["工期2周", "预算有限"]` |
  | `tools_used` | array | 使用的工具列表 | `["VS Code", "Git", "Postman"]` |
  | `technologies` | array | 技术栈列表 | `["React", "Node.js", "MongoDB"]` |
  | `external_references` | array | 外部参考资料 | `["官方文档URL", "技术博客链接"]` |
  | `custom_metadata` | object | 自定义元数据 | `{"project_id": "PROJ-123", "sprint_id": 24}` |

- **示例值**:

  ```json
  {
    "objectives": [
      "实现用户注册和登录功能",
      "集成微信和GitHub OAuth登录",
      "通过OWASP安全扫描"
    ],
    "tools_used": ["VS Code", "Git", "Postman", "Chrome DevTools"],
    "technologies": ["TypeScript", "Express.js", "JWT", "bcrypt"],
    "custom_metadata": {
      "project_id": "AUTH-MODULE-001",
      "jira_ticket": "AUTH-42"
    }
  }
  ```

**完整 task_context 示例**:

```json
{
  "task_name": "用户认证模块开发",
  "task_type": "development",
  "time_range": {
    "start_time": "2026-04-01T09:00:00+08:00",
    "end_time": "2026-04-03T17:30:00+08:00"
  },
  "description": "开发基于JWT的用户认证模块，支持多种登录方式和安全特性",
  "participants": [
    {"name": "张伟", "role": "后端开发"},
    {"name": "李娜", "role": "前端开发"}
  ],
  "context_data": {
    "objectives": [
      "实现用户注册、登录、登出功能",
      "支持本地密码和OAuth2.0第三方登录",
      "Token刷新和安全存储机制"
    ],
    "technologies": ["TypeScript", "Express.js", "JWT", "Passport.js"]
  }
}
```

---

### 2.2 generation_options 对象（可选）

generation_options 对象控制报告生成的详细程度、模板选择和内容定制。

#### 2.2.1 detail_level

- **类型**: enum
- **必填**: 否
- **默认值**: `"standard"`
- **可选值**:

  | 值 | 预计篇幅 | 包含内容 | 适用场景 |
  |---|---------|---------|---------|
  | `summary` | 2-3页（500-800字） | 仅核心章节（第1章完整 + 第10章摘要 + 其他章节仅标题和数据点） | 快速汇报、周报、管理层简报、日常站会纪要 |
  | `standard` | 8-15页（3000-5000字） | 完整10章结构，标准详细程度的分析，5-8条建议【默认推荐】 | 常规任务复盘、项目文档归档、知识分享、月度/季度总结 |
  | `detailed` | 20-30页（8000-15000字） | 所有10章完整且深入，细粒度原始数据和趋势图表，10-15条建议，完整附录 | 复杂项目深度复盘、审计需求、培训材料、重大故障事后分析 |

- **各选项详细说明**:

  **summary（快速摘要版）**:
  - ✅ 第一章：执行概览（完整）
  - ✅ 第十章：改进建议摘要（仅高优先级）
  - ⚠️ 其他章节：仅包含标题和关键数据点
  - 特点：精简高效、重点突出、一目了然

  **standard（标准版）** 【默认】:
  - ✅ 所有10个章节（完整）
  - ✅ 标准详细程度的分析
  - ✅ 标准数量的建议（5-8条）
  - 特点：结构完整、内容翔实、分析深入

  **detailed（详细深度版）**:
  - ✅ 所有10个章节（完整且深入）
  - ✅ 深度分析（包含更多图表和细分数据）
  - ✅ 详尽的问题解决过程
  - ✅ 更多方法论和建议（10-15条）
  - ✅ 完整的附录（代码清单、执行日志、监控数据、通信记录、参考资料索引）
  - 特点：极度详尽、数据丰富、可追溯性强

- **示例值**:
  ```json
  "standard"
  ```

#### 2.2.2 template_variant

- **类型**: enum
- **必填**: 否
- **默认值**: `"standard"`
- **可选值**:

  | 值 | 说明 | 与 detail_level 的关系 |
  |---|------|---------------------|
  | `summary` | 快速摘要版模板 | 当 detail_level 为 standard 时可强制使用此模板 |
  | `standard` | 标准通用模板（默认） | 适用于大多数任务类型 |
  | `detailed` | 详细深度版模板 | 当 detail_level 为 summary 时可强制使用此模板 |
  | `learning` | 学习专用模板 | 强调知识掌握、学习方法论、成长路径，调整章节角度 |

- **与 detail_level 的关系**: 当两者冲突时，template_variant 优先。例如：
  - `detail_level: "summary"` + `template_variant: "detailed"` → 使用 detailed 模板但简化内容
  - `detail_level: "detailed"` + `template_variant: "learning"` → 使用 learning 模板且内容详尽
- **何时使用 learning 模板**: 当任务是学习项目、课程总结、技能认证备考回顾时，learning 模板会：
  - 将第七章从"团队协作分析"替换为"学习支持系统"
  - 强化第九章（知识体系与方法论沉淀）和第十章（后续学习路线图）
  - 增加学习效率评估、技能等级自评等学习特有维度
- **示例值**:
  ```json
  "learning"
  ```

#### 2.2.3 included_chapters / excluded_chapters

- **类型**: integer 数组
- **必填**: 否
- **默认值**: 空（表示全部包含/不排除）
- **有效范围**: 1-10（对应报告的10个章节）
- **约束条件**:
  - 不能同时排除所有章节
  - 至少保留第1章（执行概览）、第9章（经验总结）、第10章（改进建议）
  - included_chapters 和 excluded_chapters 不能同时使用（互斥）
- **章节编号对照表**:

  | 编号 | 章节名称 | 建议保留程度 | 说明 |
  |-----|---------|------------|------|
  | 1 | 执行概览 | ★★★★★ 必须 | 提供全局视图和关键数据 |
  | 2 | 任务背景与目标 | ★★★★☆ 推荐 | 为后续分析提供基准 |
  | 3 | 执行过程详解 | ★★★★☆ 推荐 | 记录完整执行过程 |
  | 4 | 关键决策分析 | ★★★☆☆ 可选 | 重要决策的 rationale |
  | 5 | 问题与解决方案 | ★★★★★ 必须 | 核心价值章节 |
  | 6 | 资源使用情况 | ★★★☆☆ 可选 | 资源利用效率评估 |
  | 7 | 团队协作分析 | ★★☆☆☆ 可选 | 仅多人任务时建议包含 |
  | 8 | 多维度分析 | ★★★★☆ 推荐 | 深度数据分析 |
  | 9 | 经验总结与方法论 | ★★★★★ 必须 | 核心价值章节 |
  | 10 | 改进建议与行动计划 | ★★★★★ 必须 | 核心价值章节 |

- **使用示例**:

  ```json
  // 仅包含第1、5、9、10章（精简模式）
  "included_chapters": [1, 5, 9, 10]
  ```

  ```json
  // 排除第7章（无团队协作）和附录
  "excluded_chapters": [7]
  ```

#### 2.2.4 language_style

- **类型**: enum
- **必填**: 否
- **默认值**: `"professional"`
- **可选值**:

  | 值 | 语言风格特点 | 适用场景 |
  |---|-------------|---------|
  | `professional` | 专业、客观、准确，使用书面语，技术术语规范 | 正式报告、项目归档、对外分享 |
  | `casual` | 轻松、亲切、易懂，适当使用口语化表达 | 团队内部交流、个人笔记、非正式场合 |
  | `academic` | 严谨、学术化，引用规范，逻辑严密 | 研究报告、论文支撑材料、学术交流 |

- **示例值**:
  ```json
  "professional"
  ```

#### 2.2.5 focus_dimensions

- **类型**: enum 数组
- **必填**: 否
- **默认值**: 空（表示全部维度均衡分析）
- **可选值**:

  | 值 | 维度名称 | 分析重点 |
  |---|---------|---------|
  | `goal_achievement` | 目标达成度 | 目标完成率、成果质量、偏差原因分析 |
  | `time_efficiency` | 时间管理效能 | 各阶段耗时、瓶颈识别、时效比计算 |
  | `resource_utilization` | 资源利用效率 | 工具使用率、依赖合理性、成本效益 |
  | `problem_patterns` | 问题解决模式 | 问题分类统计、高频模式、根因归类 |
  | `collaboration` | 协作效果 | 沟通效率、分工合理性、协同顺畅度 |

- **用途**: 仅对指定的维度进行深度分析（包含详细图表、趋势分析和具体建议），其他维度进行简化处理（仅包含汇总级数据）。当任务有明显侧重点时使用此参数可提高报告针对性。
- **约束条件**: 最多指定5个维度（即全部），最少1个
- **示例值**:

  ```json
  // 重点关注目标达成和时间效率
  ["goal_achievement", "time_efficiency"]
  ```

  ```json
  // 全面深度分析（等同于不指定）
  ["goal_achievement", "time_efficiency", "resource_utilization", "problem_patterns", "collaboration"]
  ```

#### 2.2.6 output_format

- **类型**: enum
- **必填**: 否
- **默认值**: `"markdown"`
- **可选值**:

  | 值 | 输出格式 | 文件扩展名 | 说明 |
  |---|---------|-----------|------|
  | `markdown` | Markdown 格式 | `.md` | 默认格式，结构清晰，广泛支持，可直接渲染为HTML/PDF |
  | `json` | JSON 格式 | `.json` | 结构化数据，便于程序解析和二次处理 |
  | `html` | HTML 格式 | `.html` | 可直接在浏览器中查看，样式美观 |

- **各格式特点**:

  **markdown**:
  - ✅ 结构化标记，层次清晰
  - ✅ 广泛支持的开放格式
  - ✅ 可直接渲染为 HTML/PDF
  - ✅ 适合 Git 版本控制
  - ✅ 支持代码高亮和表格

  **json**:
  - ✅ 便于程序化处理和数据分析
  - ✅ 可轻松转换为其他格式
  - ✅ 支持字段选择性提取
  - ⚠️ 不适合直接人工阅读
  - ⚠️ 文件体积相对较大

  **html**:
  - ✅ 可直接在浏览器中打开查看
  - ✅ 内置 CSS 样式，视觉效果好
  - ✅ 适合分享和非技术人员阅读
  - ⚠️ 不利于版本控制
  - ⚠️ 编辑需要 HTML 知识

- **示例值**:
  ```json
  "markdown"
  ```

**完整 generation_options 示例**:

```json
{
  "detail_level": "standard",
  "template_variant": "standard",
  "included_chapters": [1, 2, 3, 4, 5, 6, 8, 9, 10],
  "language_style": "professional",
  "focus_dimensions": ["goal_achievement", "time_efficiency", "problem_patterns"],
  "output_format": "markdown"
}
```

---

### 2.3 output_config 对象（可选）

output_config 对象控制报告输出的存储、命名和附加选项。

#### 2.3.1 save_to_file

- **类型**: boolean
- **必填**: 否
- **默认值**: `true`
- **描述**: 是否将生成的报告保存到文件系统
- **说明**: 设为 `false` 时，报告仅在响应中返回，不写入磁盘
- **示例值**:
  ```json
  true
  ```

#### 2.3.2 file_path

- **类型**: string
- **必填**: 否（当 save_to_file 为 true 时建议指定）
- **默认值**: 自动生成路径 `{working_dir}/task-summary-{task_name_sanitized}-{timestamp}.md`
- **描述**: 报告保存的完整文件路径
- **约束条件**:
  - 必须是有效的文件路径（绝对路径或相对于工作目录的相对路径）
  - 父目录必须存在或可创建
  - 文件扩展名应与 output_format 匹配（`.md` / `.json` / `.html`）
- **示例值**:
  ```json
  "./reports/task-summary-auth-module-20260403.md"
  ```
  ```json
  "D:/projects/docs/sprint24-retro.md"
  ```

**文件命名规范**（当使用自动生成时）:

```
task-summary-[任务名称简写]-YYYYMMDD-HHmmss.[ext]

示例：
task-summary-payment-refactor-20260409.md
task-summary-sprint24-review-20260409-143022.json
```

#### 2.3.3 include_metadata

- **类型**: boolean
- **必填**: 否
- **默认值**: `true`
- **描述**: 是否在报告中包含 YAML Frontmatter 元数据块
- **说明**: 元数据包括任务名称、生成日期、报告版本、生成器信息等，便于文档管理和检索
- **Frontmatter 示例**:

  ```yaml
  ---
  name: task-execution-summary
  version: 1.0
  template_type: standard
  generated_by: Task Execution Summary Generator v2.0
  task_name: 用户认证模块开发
  generated_at: 2026-04-03T17:30:00+08:00
  ---
  ```

- **示例值**:
  ```json
  true
  ```

#### 2.3.4 append_to_existing

- **类型**: boolean
- **必填**: 否
- **默认值**: `false`
- **描述**: 是否追加到已有文件（而非覆盖）
- **约束条件**: 仅当 file_path 指向的文件已存在时生效
- **使用场景**: 需要在同一文件中累积多次任务总结（如日志形式）
- **示例值**:
  ```json
  false
  ```

#### 2.3.5 encoding

- **类型**: enum
- **必填**: 否
- **默认值**: `"utf-8"`
- **可选值**: `utf-8` | `gbk` | `gb2312` | `ascii`
- **描述**: 文件编码格式
- **说明**: 通常使用 UTF-8 即可；如需与旧系统兼容，可选择其他编码
- **示例值**:
  ```json
  "utf-8"
  ```

#### 2.3.6 custom_header / custom_footer

- **类型**: string
- **必填**: 否
- **默认值**: 空（使用系统默认的头部/尾部）
- **描述**: 自定义报告的头部或尾部内容（支持 Markdown 格式）
- **用途**: 添加公司声明、保密提示、联系方式等自定义内容
- **约束条件**: 最大长度 5000 字符
- **示例值**:

  ```json
  {
    "custom_header": "> **机密文件** - 本报告仅供内部使用，请勿外传\n>\n> **项目**: 电商平台重构 V2.0\n> **部门**: 技术研发部",
    "custom_footer": "---\n\n*报告生成器: Task Execution Summary Generator v2.0*\n*如有疑问请联系: tech-docs@company.com*"
  }
  ```

**完整 output_config 示例**:

```json
{
  "save_to_file": true,
  "file_path": "./reports/task-summary-sprint24-20260409.md",
  "include_metadata": true,
  "append_to_existing": false,
  "encoding": "utf-8",
  "custom_header": "> **Sprint 24 回顾报告**\n> **团队**: 平台研发组\n> **周期**: 2026-03-25 ~ 2026-04-07",
  "custom_footer": null
}
```

---

## 3. 输出响应格式定义

### 3.1 成功响应（HTTP 200）

当报告成功生成时，返回 HTTP 200 状态码和以下 JSON 结构：

```json
{
  "success": true,
  "report_id": "rpt_20260409_abc123def456",
  "timestamp": "2026-04-09T14:30:22+08:00",
  "processing_time_ms": 3250,
  "report": {
    "title": "任务执行总结报告：用户认证模块开发",
    "content": "# 任务执行总结报告\n\n> **报告元信息**\n> ...\n\n## 第一章：执行概览\n...",
    "word_count": 4580,
    "chapter_count": 10,
    "metadata": {
      "task_name": "用户认证模块开发",
      "task_type": "development",
      "generated_at": "2026-04-09T14:30:22+08:00",
      "generator_version": "v2.0"
      "template_used": "standard",
      "detail_level": "standard",
      "language_style": "professional",
      "file_path": "./reports/task-summary-auth-module-20260409.md",
      "file_size_bytes": 24568
    }
  },
  "quality_check": {
    "completeness_rate": 0.95,
    "accuracy_confidence": 0.97,
    "information_gaps": [
      {
        "section": "第三章：执行过程详解",
        "issue": "缺少T+120min至T+150min之间的详细操作记录",
        "severity": "low",
        "suggestion": "如需补充，可手动添加该时间段的具体操作步骤"
      }
    ],
    "warnings": [
      {
        "code": "W001",
        "message": "第七章（团队协作分析）因检测到单人任务而简化处理",
        "severity": "info"
      }
    ],
    "overall_quality_score": 92
  },
  "statistics": {
    "total_phases": 6,
    "total_decisions": 8,
    "total_problems": 12,
    "suggestions_count": 7,
    "methodologies_extracted": 3,
    "key_metrics": {
      "goal_achievement_rate": 0.93,
      "time_efficiency_ratio": 1.05,
      "resource_utilization_rate": 0.78,
      "problem_resolution_rate": 1.0
    }
  },
  "file_info": {
    "saved": true,
    "path": "./reports/task-summary-auth-module-20260409.md",
    "size_bytes": 24568,
    "checksum_md5": "d41d8cd98f00b204e9800998ecf8427e"
  }
}
```

**字段详细说明**：

#### 顶层字段

| 字段名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| `success` | boolean | 是 | 固定为 `true`，表示请求成功 |
| `report_id` | string | 是 | 报告唯一标识符，格式：`rpt_{YYYYMMDD}_{随机16位hex}`，可用于后续查询和管理 |
| `timestamp` | datetime | 是 | 报告生成完成的 ISO 8601 时间戳 |
| `processing_time_ms` | integer | 是 | 报告生成耗时（毫秒），用于性能监控和用户体验反馈 |

#### report 对象

| 字段名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| `title` | string | 是 | 报告完整标题，通常格式为"任务执行总结报告：{task_name}" |
| `content` | string | 是 | 报告完整内容（Markdown/JSON/HTML 格式，取决于 output_format 参数） |
| `word_count` | integer | 是 | 报告正文字数统计（不含元数据和空格） |
| `chapter_count` | integer | 是 | 实际生成的章节数量（可能因 excluded_chapters 参数而少于10） |
| `metadata` | object | 是 | 报告元数据对象（详见下表） |

**metadata 子字段**：

| 字段名 | 类型 | 说明 |
|-------|------|------|
| `task_name` | string | 任务名称 |
| `task_type` | string | 任务类型（实际使用的值） |
| `generated_at` | datetime | 生成时间戳 |
| `generator_version` | string | 生成器版本号 |
| `template_used` | string | 实际使用的模板变体 |
| `detail_level` | string | 实际的详细程度 |
| `language_style` | string | 实际的语言风格 |
| `file_path` | string | 保存的文件路径（如果保存到文件） |
| `file_size_bytes` | integer | 文件大小（字节） |

#### quality_check 对象

| 字段名 | 类型 | 说明 |
|-------|------|------|
| `completeness_rate` | float | 信息完整率（0-1），衡量报告涵盖的任务关键信息占应涵盖信息的比例。>0.90 为合格 |
| `accuracy_confidence` | float | 准确性置信度（0-1），基于事实可追溯性和数据一致性评估。>0.95 为优秀 |
| `information_gaps` | array | 信息缺口列表，每个元素包含 section（缺失章节）、issue（问题描述）、severity（[严重程度](terminology.md#44-严重程度-severity)）、suggestion（补充建议） |
| `warnings` | array | 警告信息列表，每个元素包含 code（警告码）、message（警告消息）、severity（严重程度：info/warning/error） |
| `overall_quality_score` | integer | 综合质量评分（0-100），基于完整性、准确性、结构规范性等多维度加权计算 |

#### statistics 对象

| 字段名 | 类型 | 说明 |
|-------|------|------|
| `total_phases` | integer | 识别出的任务阶段数量 |
| `total_decisions` | integer | 记录的关键决策数量 |
| `total_problems` | integer | 记录的问题总数 |
| `suggestions_count` | integer | 生成的改进建议数量 |
| `methodologies_extracted` | integer | 提炼的方法论数量 |
| `key_metrics` | object | 核心指标对象（详见下表） |

**key_metrics 子字段**：

| 字段名 | 类型 | 说明 | 计算公式 |
|-------|------|------|---------|
| `goal_achievement_rate` | float | [目标达成率](terminology.md#26-达成率-achievement-rate) | 实际完成目标数 / 总目标数 |
| `time_efficiency_ratio` | float | [时间效能比](terminology.md#34-时效比-time-efficiency-ratio) | 计划总时长 / 实际总时长（>1 表示提前，<1 表示超时） |
| `resource_utilization_rate` | float | [资源利用率](terminology.md#44-利用率-utilization-rate) | 有效使用资源数 / 总引入资源数 |
| `problem_resolution_rate` | float | 问题解决率 | 已解决问题数 / 总问题数 |

#### file_info 对象（仅在 save_to_file=true 时存在）

| 字段名 | 类型 | 说明 |
|-------|------|------|
| `saved` | boolean | 是否成功保存到文件 |
| `path` | string | 保存的完整文件路径 |
| `size_bytes` | integer | 文件大小（字节） |
| `checksum_md5` | string | 文件 MD5 校验和，用于完整性验证 |

---

### 3.2 错误响应（HTTP 4xx/5xx）

当请求失败时，返回相应的 HTTP 错误状态码和统一的错误响应格式：

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数验证失败",
    "category": "client_error",
    "severity": "error",
    "timestamp": "2026-04-09T14:28:15+08:00",
    "request_id": "req_20260409_xyz789",
    "details": [
      {
        "field": "task_context.task_name",
        "issue": "任务名称不能为空",
        "constraint": "minLength: 2",
        "actual_value": "",
        "suggestion": "请提供一个2-200字符的任务名称"
      },
      {
        "field": "task_context.time_range.end_time",
        "issue": "结束时间不能早于开始时间",
        "constraint": "end_time > start_time",
        "actual_value": "2026-04-01T08:00:00+08:00",
        "suggestion": "请确保结束时间晚于开始时间 2026-04-01T09:00:00+08:00"
      }
    ],
    "recovery_actions": [
      "检查并修正 task_name 字段，确保不为空且长度符合要求",
      "检查 time_range 的时间逻辑，确保 end_time 晚于 start_time",
      "参考本文档第2节的参数定义确认正确的参数格式"
    ],
    "documentation_url": "https://docs.task-execution-summary.com/api/errors/VALIDATION_ERROR",
    "support_contact": "support@task-execution-summary.com"
  },
  "http_status_code": 400
}
```

**错误响应字段说明**：

| 字段名 | 类型 | 说明 |
|-------|------|------|
| `success` | boolean | 固定为 `false` |
| `error.code` | string | 错误代码（大写下划线格式），唯一标识错误类型 |
| `error.message` | string | 人类可读的错误摘要信息 |
| `error.category` | string | 错误类别：`client_error`（4xx）/ `server_error`（5xx） |
| `error.severity` | string | 严重程度：`info` / `warning` / `error` / `critical` |
| `error.timestamp` | datetime | 错误发生的 ISO 8601 时间戳 |
| `error.request_id` | string | 请求追踪 ID，用于日志关联和技术支持 |
| `error.details` | array | 详细错误信息数组（每个元素对应一个具体的参数错误） |
| `error.recovery_actions` | array | 建议的恢复操作步骤列表 |
| `error.documentation_url` | string | 相关文档链接（可选） |
| `error.support_contact` | string | 技术支持联系方式（可选） |
| `http_status_code` | integer | HTTP 状态码（冗余字段，方便客户端直接读取） |

**details 数组元素结构**：

| 字段名 | 类型 | 说明 |
|-------|------|------|
| `field` | string | 出错参数的 JSON Path（如 `task_context.task_name`） |
| `issue` | string | 具体的错误描述 |
| `constraint` | string | 违反的约束条件 |
| `actual_value` | any | 实际传入的值（脱敏处理敏感信息） |
| `suggestion` | string | 修正建议 |

**常见错误代码一览表**：

| HTTP 状态码 | 错误代码 | 类别 | 说明 | 常见原因 |
|-----------|---------|------|------|---------|
| 400 | `VALIDATION_ERROR` | client_error | 请求参数验证失败 | 缺少必填参数、参数格式错误、值超出范围 |
| 400 | `INVALID_JSON` | client_error | JSON 格式错误 | 请求体不是合法的 JSON、语法错误 |
| 401 | `UNAUTHORIZED` | client_error | 未授权 | 缺少或无效的认证凭据 |
| 403 | `FORBIDDEN` | client_error | 权限不足 | API Key 无权访问该端点或功能 |
| 404 | `NOT_FOUND` | client_error | 资源不存在 | 查询的报告 ID 不存在、端点路径错误 |
| 429 | `RATE_LIMIT_EXCEEDED` | client_error | 超过速率限制 | 请求频率过高，需等待冷却期 |
| 500 | `INTERNAL_SERVER_ERROR` | server_error | 服务器内部错误 | 服务端异常，请联系技术支持 |
| 502 | `BAD_GATEWAY` | server_error | 网关错误 | 上游服务不可用 |
| 503 | `SERVICE_UNAVAILABLE` | server_error | 服务暂不可用 | 服务过载或维护中，稍后重试 |
| 504 | `GATEWAY_TIMEOUT` | server_error | 网关超时 | 处理时间过长，建议使用异步接口 |

---

## 4. 参数验证规则表

下表汇总了所有输入参数的验证规则，供快速查阅：

| 参数 | 类型检查 | 范围检查 | 必填检查 | 默认值应用 | 冲突检测 | 其他约束 |
|------|---------|---------|---------|-----------|---------|---------|
| **task_context** | ✅ object | N/A | ✅ 必填 | N/A | N/A | 至少包含 task_name |
| &nbsp;&nbsp;└─ task_name | ✅ string | 长度 2-200 | ✅ 必填 | N/A | N/A | 不允许纯空格/控制字符 |
| &nbsp;&nbsp;└─ task_type | ✅ enum | 6个有效值 | ❌ 可选 | auto-detect | N/A | 值必须为预定义枚举之一 |
| &nbsp;&nbsp;└─ time_range | ✅ object | N/A | ❌ 可选 | 自动提取 | N/A | 如提供则需包含有效时间 |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─ start_time | ✅ datetime | ISO 8601 | ❌ 可选 | 自动提取 | 必须早于 end_time | 合法的日期时间格式 |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─ end_time | ✅ datetime | ISO 8601 | ❌ 可选 | 自动提取/当前时间 | 必须晚于 start_time | 合法的日期时间格式 |
| &nbsp;&nbsp;└─ description | ✅ string | 长度 10-2000 | ❌ 可选 | 自动提取 | N/A | N/A |
| &nbsp;&nbsp;└─ participants | ✅ array | 最多50个元素 | ❌ 可选 | [] | N/A | 每个元素需包含 name 和 role |
| &nbsp;&nbsp;└─ context_data | ✅ object | N/A | ❌ 可选 | {} | N/A | 子字段按各自规则验证 |
| **generation_options** | ✅ object | N/A | ❌ 可选 | 全部使用默认值 | N/A | N/A |
| &nbsp;&nbsp;└─ detail_level | ✅ enum | 3个有效值 | ❌ 可选 | standard | N/A | 值必须为 summary/standard/detailed |
| &nbsp;&nbsp;└─ template_variant | ✅ enum | 4个有效值 | ❌ 可选 | standard | 与 detail_level 可能冲突时优先 | 值必须为预定义枚举之一 |
| &nbsp;&nbsp;└─ included_chapters | ✅ int[] | 1-10, 最多10个 | ❌ 可选 | [] (全包含) | 与 excluded_chapters 互斥 | 必须包含1、9、10中的至少1个 |
| &nbsp;&nbsp;└─ excluded_chapters | ✅ int[] | 1-10, 最多9个 | ❌ 可选 | [] (不排除) | 与 included_chapters 互斥 | 不能排除1、9、10的全部 |
| &nbsp;&nbsp;└─ language_style | ✅ enum | 3个有效值 | ❌ 可选 | professional | N/A | 值必须为 professional/casual/academic |
| &nbsp;&nbsp;└─ focus_dimensions | ✅ enum[] | 1-5个有效值 | ❌ 可选 | [] (全维度) | N/A | 值必须是预定义的5个维度之一 |
| &nbsp;&nbsp;└─ output_format | ✅ enum | 3个有效值 | ❌ 可选 | markdown | N/A | 值必须为 markdown/json/html |
| **output_config** | ✅ object | N/A | ❌ 可选 | 全部使用默认值 | N/A | N/A |
| &nbsp;&nbsp;└─ save_to_file | ✅ boolean | true/false | ❌ 可选 | true | N/A | N/A |
| &nbsp;&nbsp;└─ file_path | ✅ string | 有效路径 | ❌ 可选 | 自动生成 | N/A | 父目录必须存在或可创建 |
| &nbsp;&nbsp;└─ include_metadata | ✅ boolean | true/false | ❌ 可选 | true | N/A | N/A |
| &nbsp;&nbsp;└─ append_to_existing | ✅ boolean | true/false | ❌ 可选 | false | N/A | 仅对已存在文件生效 |
| &nbsp;&nbsp;└─ encoding | ✅ enum | 4个有效值 | ❌ 可选 | utf-8 | N/A | 值必须是支持的编码之一 |
| &nbsp;&nbsp;└─ custom_header | ✅ string | ≤5000字符 | ❌ 可选 | null | N/A | 支持 Markdown 格式 |
| &nbsp;&nbsp;└─ custom_footer | ✅ string | ≤5000字符 | ❌ 可选 | null | N/A | 支持 Markdown 格式 |

**验证流程说明**：

1. **类型检查**：首先验证每个参数的数据类型是否正确（如 string 不能传 number）
2. **必填检查**：验证所有标记为必填的参数是否存在
3. **范围检查**：验证参数值是否在允许的范围内（长度、枚举值、数值区间等）
4. **默认值应用**：对未提供的可选参数应用默认值
5. **冲突检测**：检查参数之间是否存在逻辑冲突（如 included_chapters 和 excluded_chapters 同时使用）
6. **业务规则校验**：执行特定业务逻辑的验证（如时间顺序、章节依赖关系等）

**验证顺序**：按照上表从上到下的顺序依次执行，一旦某步失败即返回错误，后续步骤不再执行。这确保了错误信息的确定性和可预测性。

---

## 5. 调用示例

### 5.1 示例一：最小调用（仅 task_name）

**请求报文**：

```json
{
  "task_context": {
    "task_name": "Docker容器化部署学习"
  }
}
```

**说明**：系统自动推断所有可选参数。完整示例请参阅 [examples-v2.md](examples-v2.md#示例-2-sprint-复盘最小化调用)。

---

### 5.2 示例二：标准调用（常用配置组合）

**请求报文**：

```json
{
  "task_context": {
    "task_name": "电商平台支付模块重构",
    "task_type": "development",
    "time_range": {
      "start_time": "2026-03-15T09:00:00+08:00",
      "end_time": "2026-03-28T18:00:00+08:00"
    },
    "description": "对电商平台的核心支付模块进行技术重构...",
    "participants": [
      {"name": "张伟", "role": "Tech Lead"},
      {"name": "李娜", "role": "后端开发"}
    ]
  },
  "generation_options": {
    "detail_level": "standard",
    "excluded_chapters": [7],
    "focus_dimensions": ["goal_achievement", "time_efficiency", "problem_patterns"]
  },
  "output_config": {
    "save_to_file": true,
    "file_path": "./reports/payment-refactor-20260328.md"
  }
}
```

**说明**：软件开发类任务的典型配置。完整示例请参阅 [examples-v2.md](examples-v2.md#示例-1-软件开发任务标准调用)。

---

### 5.3 示例三：完全配置调用（所有参数都指定）

**请求报文**：

```json
{
  "task_context": {
    "task_name": "React 18框架深入学习与实践",
    "task_type": "learning",
    "time_range": {
      "start_time": "2026-02-15T20:00:00+08:00",
      "end_time": "2026-03-28T23:59:59+08:00"
    },
    "description": "系统学习React 18框架，掌握Hooks、Redux Toolkit等技能",
    "participants": [
      {"name": "学习者本人", "role": "学习者"}
    ],
    "context_data": {
      "objectives": ["掌握React 18核心概念", "熟练使用Redux Toolkit"],
      "technologies": ["React 18", "TypeScript", "Redux Toolkit"]
    }
  },
  "generation_options": {
    "detail_level": "detailed",
    "template_variant": "learning",
    "included_chapters": [1, 2, 3, 4, 5, 6, 8, 9, 10]
  },
  "output_config": {
    "save_to_file": true,
    "file_path": "D:/learning-logs/react18-learning-journey-20260328.md"
  }
}
```

**说明**：学习类任务的详细配置，使用学习专用模板。完整示例请参阅 [examples-v2.md](examples-v2.md)。

---

## 6. 版本历史与变更记录

### v2.0 (2026-04-09) - 初始版本

**发布日期**: 2026年4月9日

**版本类型**: Major Release（主版本发布）

**新增功能**：

✅ **核心接口**
- 实现 `/generate` 同步报告生成端点
- 实现 `/generate/async` 异步报告生成端点
- 实现 `/status/{report_id}` 异步任务状态查询端点
- 实现 `/templates` 可用模板列表查询端点
- 实现 `/validate` 参数预验证端点

✅ **任务上下文 (task_context)**
- 支持 task_name、task_type、time_range、description、participants、context_data 等完整参数体系
- 实现自动检测任务类型（auto-detect）功能
- 支持从对话历史自动提取时间范围和基本信息
- 支持丰富的上下文数据扩展（目标、约束、工具、技术栈等）

✅ **生成选项 (generation_options)**
- 提供3种详细程度（summary/standard/detailed）
- 提供4种模板变体（summary/standard/detailed/learning）
- 支持灵活的章节包含/排除定制
- 提供3种语言风格（professional/casual/academic）
- 支持5个分析维度的焦点定制
- 提供3种输出格式（markdown/json/html）

✅ **输出配置 (output_config)**
- 支持自动或手动指定文件保存路径
- 支持 YAML Frontmatter 元数据包含
- 支持文件追加模式和多种编码格式
- 支持自定义报告头部和尾部内容

✅ **质量控制**
- 实现完整的参数验证体系（类型、范围、必填、冲突检测）
- 提供详细的错误响应格式（错误码、恢复建议、文档链接）
- 内置报告质量检查机制（完整性、准确性、质量评分）
- 提供丰富的统计数据（阶段数、决策数、问题数、建议数、核心指标）

✅ **认证与限流**
- 支持 API Key 和 OAuth 2.0/JWT 认证方式
- 实现分层速率限制（免费/专业/企业版）
- 提供标准的 Rate Limit 响应头

**已知限制**：

⚠️ 当前版本不支持以下功能（计划在未来版本中加入）：
- 用户自定义报告模板上传
- 多语言报告生成（目前仅支持中文）
- 报告版本管理和差异对比
- 批量报告生成（一次生成多个任务的总结）
- Webhook 回调通知（异步任务完成时触发）
- 报告协作编辑和评论功能
- 与外部知识库系统的集成（如 Notion、Confluence）
- AI 增强分析（使用 LLM 进行更深度的洞察挖掘）

**兼容性说明**：

- ✅ 向后兼容承诺：v1.x 系列的所有小版本更新都将保持 API 的向后兼容性
- ✅ 废弃策略：如需废弃某个参数或端点，将在至少2个小版本前通过 `warnings` 字段发出废弃通知，并在文档中标注 `@deprecated`
- ✅ 数据迁移：升级过程中不会影响已生成的历史报告文件

**文档配套**：

- 📘 本文档（API 接口参考文档）
- 📘 SKILL.md（技能完整说明文档）
- 📘 examples-v2.md（典型使用场景示例）
- 📘 templates.md（报告模板变体详细说明）
- 📘 terminology.md（术语表和概念定义）
- 📘 evals/evals.json（评估标准和测试用例）

**反馈与支持**：

- 📧 技术支持邮箱：support@task-execution-summary.com
- 🐛 问题反馈：https://github.com/task-execution-summary/issues
- 📚 官方文档：https://docs.task-execution-summary.com
- 💬 社区论坛：https://community.task-execution-summary.com

---

## 附录

### A. 快速参考卡片

**最小可用请求**：

```json
{"task_context": {"task_name": "你的任务名称"}}
```

**常用标准请求**：

```json
{
  "task_context": {
    "task_name": "任务名称",
    "task_type": "development|management|operations|research|learning",
    "time_range": {"start_time": "ISO8601", "end_time": "ISO8601"}
  },
  "generation_options": {
    "detail_level": "summary|standard|detailed",
    "output_format": "markdown|json|html"
  }
}
```

**响应状态速查**：

| 状态码 | 含义 | 处理建议 |
|-------|------|---------|
| 200 | 成功 | 解析 report.content 获取报告 |
| 400 | 参数错误 | 查看 error.details 修正请求 |
| 401 | 未认证 | 添加 Authorization 头 |
| 429 | 限流 | 等待 X-RateLimit-Reset 后重试 |
| 500 | 服务器错误 | 稍后重试或联系支持 |

### B. 性能基准

| 指标 | summary | standard | detailed |
|------|---------|----------|----------|
| 平均生成时间 | 1-2秒 | 3-5秒 | 8-15秒 |
| 平均报告字数 | 500-800字 | 3000-5000字 | 8000-15000字 |
| 内存占用峰值 | ~50MB | ~120MB | ~300MB |
| 并发支持（同实例） | 10 | 5 | 2 |

*注：以上数据基于标准硬件配置（4核CPU、8GB内存）的测试结果，实际性能可能因任务复杂度和服务器负载而有所差异*

### C. 相关文档索引

| 文档名称 | 路径 | 说明 |
|---------|------|------|
| 技能完整说明 | `SKILL.md` | 技能的功能、原理和使用指南 |
| 使用示例 | `references/examples-v2.md` | 4个典型场景的详细示例 |
| 模板规范 | `references/templates.md` | 4种模板变体的完整结构和填写指南 |
| 术语表 | `references/terminology.md` | 专业术语的定义和解释 |
| 评估标准 | `evals/evals.json` | 报告质量的评估维度和测试用例 |

---

> **文档结束**
>
> *本文档由 Task Execution Summary Generator Team 维护*
>
> *最后更新时间: 2026-04-09 14:30:22 CST*
>
> *下次计划更新: 根据用户反馈和功能迭代动态调整*
