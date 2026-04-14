---
version: "1.0-quick"
updated: "2026-04-15"
skill_version: "v1.0+"
---

# API 接口速查

> **完整 API 详解请查看 [api-reference.md](api-reference.md)**

---

## 基础信息

| 属性 | 值 |
|------|---|
| API 版本 | v1.0 |
| 协议 | RESTful API / JSON-RPC |
| 数据格式 | JSON (请求) / Markdown 或 JSON (响应) |
| 编码 | UTF-8 |

### 基础 URL
```
生产环境: https://api.task-execution-summary.com/v1
测试环境: https://staging-api.task-execution-summary.com/v1
本地开发: http://localhost:8080/v1
```

---

## API 端点一览

| 端点路径 | 方法 | 功能描述 | 认证要求 |
|---------|------|---------|---------|
| `/generate` | POST | 生成任务执行总结报告 | 可选 |
| `/generate/async` | POST | 异步生成报告 | 可选 |
| `/status/{report_id}` | GET | 查询异步任务状态 | 可选 |
| `/templates` | GET | 获取可用模板列表 | 无需认证 |
| `/validate` | POST | 验证请求参数合法性 | 无需认证 |

---

## 请求参数速查

### task_context 对象（必填）

| 参数 | 类型 | 必填 | 默认值 | 约束 |
|------|------|------|--------|------|
| `task_name` | string | ✅ | - | 长度 2-200 |
| `task_type` | enum | ❌ | auto-detect | development/management/operations/research/learning |
| `time_range` | object | ❌ | 自动提取 | {start_time, end_time} |
| `description` | string | ❌ | 自动提取 | 长度 10-2000 |
| `participants` | array | ❌ | [] | 最多50个元素 |
| `context_data` | object | ❌ | {} | 自定义上下文数据 |

### generation_options 对象（可选）

| 参数 | 类型 | 必填 | 默认值 | 可选值 |
|------|------|------|--------|--------|
| `detail_level` | enum | ❌ | standard | summary/standard/detailed |
| `template_variant` | enum | ❌ | standard | summary/standard/detailed/learning |
| `included_chapters` | int[] | ❌ | [] (全包含) | 1-10 |
| `excluded_chapters` | int[] | ❌ | [] (不排除) | 1-10 |
| `language_style` | enum | ❌ | professional | professional/casual/academic |
| `focus_dimensions` | enum[] | ❌ | [] (全维度) | 最多5个维度 |
| `output_format` | enum | ❌ | markdown | markdown/json/html |

### output_config 对象（可选）

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `save_to_file` | boolean | ❌ | true | 是否保存到文件 |
| `file_path` | string | ❌ | 自动生成 | 保存路径 |
| `include_metadata` | boolean | ❌ | true | 包含 YAML Frontmatter |
| `append_to_existing` | boolean | ❌ | false | 追加到已有文件 |
| `encoding` | enum | ❌ | utf-8 | utf-8/gbk/gb2312/ascii |
| `custom_header` | string | ❌ | null | 自定义头部（≤5000字符） |
| `custom_footer` | string | ❌ | null | 自定义尾部（≤5000字符） |

---

## 参数详细说明

### detail_level 详细程度

| 值 | 预计篇幅 | 包含内容 | 适用场景 |
|---|---------|---------|---------|
| `summary` | 2-3页（500-800字） | 核心章节（第1章完整 + 第10章摘要） | 快速汇报、周报 |
| `standard` | 8-15页（3000-5000字） | 完整10章结构 | 常规复盘、知识分享 |
| `detailed` | 20-30页（8000-15000字） | 所有10章深入 + 完整附录 | 深度复盘、审计 |

### task_type 任务类型

| 值 | 说明 | 分析侧重 |
|---|------|---------|
| `development` | 软件开发类 | 技术决策、代码质量、问题解决 |
| `management` | 项目管理类 | 进度管理、资源分配、团队协作 |
| `operations` | 运维排查类 | 问题诊断、根因分析、预防措施 |
| `research` | 技术研究类 | 技术对比、方案论证、创新发现 |
| `learning` | 学习成长类 | 知识掌握、学习方法、能力提升 |
| `auto-detect` | 自动检测（默认） | 智能识别任务特征 |

### language_style 语言风格

| 值 | 特点 | 适用场景 |
|---|------|---------|
| `professional` | 专业、客观、准确 | 正式报告、项目归档 |
| `casual` | 轻松、亲切、易懂 | 团队内部交流、个人笔记 |
| `academic` | 严谨、学术化 | 研究报告、论文支撑 |

### focus_dimensions 分析维度

| 值 | 维度名称 | 分析重点 |
|---|---------|---------|
| `goal_achievement` | 目标达成度 | 完成率、成果质量、偏差分析 |
| `time_efficiency` | 时间管理效能 | 各阶段耗时、瓶颈识别 |
| `resource_utilization` | 资源利用效率 | 工具使用率、成本效益 |
| `problem_patterns` | 问题解决模式 | 问题分类、高频模式 |
| `collaboration` | 协作效果 | 沟通效率、分工合理性 |

---

## 章节编号对照

| 编号 | 章节名称 | 建议保留程度 |
|-----|---------|------------|
| 1 | 执行概览 | ★★★★★ 必须 |
| 2 | 任务背景与目标 | ★★★★☆ 推荐 |
| 3 | 执行过程详解 | ★★★★☆ 推荐 |
| 4 | 关键决策分析 | ★★★☆☆ 可选 |
| 5 | 问题与解决方案 | ★★★★★ 必须 |
| 6 | 资源使用情况 | ★★★☆☆ 可选 |
| 7 | 团队协作分析 | ★★☆☆☆ 可选 |
| 8 | 多维度分析 | ★★★★☆ 推荐 |
| 9 | 经验总结与方法论 | ★★★★★ 必须 |
| 10 | 改进建议与行动计划 | ★★★★★ 必须 |

**约束**: 必须包含1、9、10中的至少1个；不能排除1、9、10的全部。

---

## 成功响应结构

```json
{
  "success": true,
  "report_id": "rpt_20260409_abc123def456",
  "timestamp": "2026-04-09T14:30:22+08:00",
  "processing_time_ms": 3250,
  "report": {
    "title": "任务执行总结报告：xxx",
    "content": "# Markdown 内容...",
    "word_count": 4580,
    "chapter_count": 10,
    "metadata": { /* 报告元数据 */ }
  },
  "quality_check": {
    "completeness_rate": 0.95,
    "accuracy_confidence": 0.97,
    "overall_quality_score": 92
  },
  "statistics": {
    "total_phases": 6,
    "total_decisions": 8,
    "total_problems": 12,
    "suggestions_count": 7,
    "key_metrics": {
      "goal_achievement_rate": 0.93,
      "time_efficiency_ratio": 1.05,
      "resource_utilization_rate": 0.78,
      "problem_resolution_rate": 1.0
    }
  },
  "file_info": {
    "saved": true,
    "path": "./reports/xxx.md",
    "size_bytes": 24568
  }
}
```

---

## 错误响应结构

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
        "suggestion": "请提供2-200字符的任务名称"
      }
    ],
    "recovery_actions": ["修复建议1", "修复建议2"]
  },
  "http_status_code": 400
}
```

---

## HTTP 状态码速查

| 状态码 | 错误代码 | 说明 | 常见原因 |
|-------|---------|------|---------|
| 200 | - | 成功 | 报告生成完成 |
| 206 | PARTIAL_CONTENT | 部分内容 | 降级生成，含警告 |
| 400 | VALIDATION_ERROR | 参数验证失败 | 缺少参数、格式错误 |
| 400 | INVALID_JSON | JSON格式错误 | 语法错误 |
| 401 | UNAUTHORIZED | 未授权 | 缺少认证凭据 |
| 403 | FORBIDDEN | 权限不足 | API Key无权访问 |
| 404 | NOT_FOUND | 资源不存在 | 报告ID不存在 |
| 429 | RATE_LIMIT_EXCEEDED | 超过速率限制 | 请求频率过高 |
| 500 | INTERNAL_SERVER_ERROR | 服务器内部错误 | 服务端异常 |
| 502 | BAD_GATEWAY | 网关错误 | 上游服务不可用 |
| 503 | SERVICE_UNAVAILABLE | 服务暂不可用 | 服务过载或维护 |
| 504 | GATEWAY_TIMEOUT | 网关超时 | 处理时间过长 |

---

## 速率限制

| 计费层级 | 限制 | 说明 |
|---------|------|------|
| 免费版 | 100 次/小时 | 适合个人学习和测试 |
| 专业版 | 1000 次/小时 | 适合团队日常使用 |
| 企业版 | 自定义 | 适合大规模集成 |

**响应头**:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000
```

---

## 调用示例

### 最小调用（仅 task_name）

```json
{
  "task_context": {
    "task_name": "Docker容器化部署学习"
  }
}
```

### 标准调用（常用配置）

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
    "focus_dimensions": ["goal_achievement", "time_efficiency"],
    "output_format": "markdown"
  },
  "output_config": {
    "save_to_file": true,
    "file_path": "./reports/payment-refactor.md"
  }
}
```

### 完全配置调用

```json
{
  "task_context": {
    "task_name": "用户认证模块开发",
    "task_type": "development",
    "time_range": {
      "start_time": "2026-04-01T09:00:00+08:00",
      "end_time": "2026-04-03T17:30:00+08:00"
    },
    "description": "开发基于JWT的用户认证模块...",
    "participants": [
      {"name": "张伟", "role": "后端开发"},
      {"name": "李娜", "role": "前端开发"}
    ],
    "context_data": {
      "objectives": ["实现用户注册登录", "支持OAuth2.0"],
      "technologies": ["TypeScript", "Express.js", "JWT"]
    }
  },
  "generation_options": {
    "detail_level": "detailed",
    "template_variant": "standard",
    "included_chapters": [1, 2, 3, 4, 5, 6, 8, 9, 10],
    "language_style": "professional",
    "focus_dimensions": ["goal_achievement", "time_efficiency", "problem_patterns"],
    "output_format": "markdown"
  },
  "output_config": {
    "save_to_file": true,
    "file_path": "./reports/auth-module.md",
    "include_metadata": true,
    "encoding": "utf-8"
  }
}
```

---

## cURL 调用示例

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

# 异步调用
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

# 查询异步任务状态
curl -X GET https://api.task-execution-summary.com/v1/status/rpt_xxx \
  -H "Authorization: Bearer YOUR_API_KEY"

# 获取模板列表
curl -X GET https://api.task-execution-summary.com/v1/templates

# 验证参数
curl -X POST https://api.task-execution-summary.com/v1/validate \
  -H "Content-Type: application/json" \
  -d '{"task_context": {"task_name": "测试"}}'
```

---

## 认证方式

### API Key 认证

```http
Authorization: Bearer your_api_key_here
```

或在查询参数中：
```
?api_key=your_api_key_here
```

### 获取 API Key

1. 注册开发者账号
2. 在控制台创建应用
3. 生成 API Key（支持设置过期时间和权限范围）
4. 在请求头中携带 API Key

---

*本文档为快速参考版，完整信息请参阅 [api-reference.md](api-reference.md)*
