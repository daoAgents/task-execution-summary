# V3 关键技术 PoC 验证方案

**文档版本**: v1.0
**创建日期**: 2026-04-14
**状态**: DRAFT - 规划阶段
**关联文档**: [v3-roadmap.md](v3-roadmap.md)

---

## 目录

- [1. 概述](#1-概述)
- [2. 模块 B1: 自定义模板引擎 PoC](#2-模块-b1-自定义模板引擎-poc)
- [3. 模块 B3: 外部工具集成 PoC](#3-模块-b3-外部工具集成-poc)
- [4. 模块 B5: 团队协作引擎 PoC](#4-模块-b5-团队协作引擎-poc)
- [5. PoC 执行总计划](#5-poc-执行总计划)
- [6. 附录](#6-附录)

---

## 1. 概述

### 1.1 PoC 验证整体目标

在 V3 大规模投入研发前，通过概念验证（Proof of Concept）方式验证三个高风险模块的技术可行性，降低技术选型失误风险，为后续正式开发提供可靠的技术决策依据。

```
┌─────────────────────────────────────────────────────────────────────┐
│                     PoC 验证战略定位                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   技术选型阶段              →              正式开发阶段              │
│   ┌─────────────┐                         ┌──────────────┐          │
│   │  方案对比    │      ┌──────────→      │  工程化实现   │          │
│   │  原型验证    │                         │  生产部署    │          │
│   │  风险识别    │                         │  持续迭代    │          │
│   └─────────────┘                         └──────────────┘          │
│         ↑                                    ↑                      │
│         └────────── PoC 验证 ────────────────┘                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 风险等级排序

| 优先级 | 模块 | 风险等级 | 风险来源 | 失败影响 |
|-------|------|---------|---------|---------|
| **P0** | B5 团队协作 | 高 | 实时协作算法复杂、并发场景难测 | 核心卖点无法实现，产品定位崩塌 |
| **P1** | B3 外部工具集成 | 中高 | 第三方 API 不稳定、OAuth 流程复杂 | 生态连接能力受限，扩展性受损 |
| **P2** | B1 自定义模板 | 中 | 多格式转换复杂、安全边界难控 | 功能受限，但核心流程可用 |

### 1.3 总体时间预算

| 模块 | 时间估算 | 人力投入 | 并行度 |
|------|---------|---------|-------|
| B1 模板引擎 | 7 天 | 1 名全栈工程师 | 可与 B3 并行 |
| B3 工具集成 | 10 天 | 1 名后端工程师 | 可与 B1 并行 |
| B5 协作引擎 | 12 天 | 2 名工程师（前后端各1） | 独立进行 |
| **总计** | **8 周** | **3 名工程师** | 考虑并行后 |

---

## 2. 模块 B1: 自定义模板引擎 PoC

### 2.1 验证目标

#### 2.1.1 核心目标

1. **多格式输出验证**：证明模板引擎能支持 Markdown/HTML/DOCX/PDF 四种输出格式
2. **变量系统验证**：验证模板变量系统的灵活性和安全性（防注入）
3. **性能基准验证**：建立渲染性能基线，确认满足生产环境要求

#### 2.1.2 验证范围边界

| 范围维度 | 包含内容 | 排除内容 |
|---------|---------|---------|
| **输入格式** | HTML、Markdown 模板 | PPTX、LaTeX（P2优先级） |
| **输出格式** | HTML、Markdown、DOCX、PDF | 实时预览、版本控制 |
| **变量类型** | 字符串、数字、数组、对象、布尔值 | 自定义过滤器、辅助函数 |
| **安全特性** | XSS 防护、变量转义 | 沙箱隔离、代码执行限制 |
| **性能测试** | 单用户渲染延迟 | 并发压力测试（移至正式阶段） |

### 2.2 技术方案选项对比

#### 2.2.1 候选方案概述

| 方案 | 技术基础 | 核心特点 | 适用场景 |
|------|---------|---------|---------|
| **方案 A** | Nunjucks | Jinja2 的 JS 移植版，功能强大，语法成熟 | 复杂模板逻辑、企业级应用 |
| **方案 B** | Handlebars | 逻辑轻量，安全沙箱好，学习曲线平缓 | 简单模板、安全性要求高 |
| **方案 C** | 自研 DSL | 完全可控，可深度定制，无外部依赖 | 特殊需求、长期维护 |

#### 2.2.2 详细对比矩阵

| 对比维度 | 方案 A: Nunjucks | 方案 B: Handlebars | 方案 C: 自研 DSL |
|---------|-----------------|-------------------|-----------------|
| **学习曲线** | 中等（需熟悉 Jinja2 语法） | 低（Mustache 风格，直观） | 高（需学习自定义语法） |
| **渲染性能** | 高（预编译模板，~5ms/模板） | 高（轻量级，~3ms/模板） | 取决于实现（预估 10-20ms） |
| **安全沙箱** | 中（支持沙箱模式，需配置） | 高（默认无逻辑执行，安全） | 完全可控（可实现最高安全） |
| **多格式支持** | 需配合库（pandoc/html-pdf） | 需配合库（同上） | 需自行实现转换层 |
| **社区活跃度** | 高（GitHub 8k+ stars） | 高（GitHub 17k+ stars） | 无（自行维护） |
| **包大小** | ~150KB（含依赖） | ~80KB（核心） | 取决于实现（预估 50-100KB） |
| **模板继承** | 原生支持（extends/block） | 需插件支持 | 需自行实现 |
| **条件逻辑** | 完整（if/for/macro） | 基础（if/each，无复杂逻辑） | 完全自定义 |
| **调试支持** | 良好（错误信息详细） | 良好（行号定位） | 需自行开发 |
| **长期维护** | 依赖社区 | 依赖社区 | 完全自主 |

#### 2.2.3 推荐方案

**初步推荐：方案 B (Handlebars) + 方案 A (Nunjucks) 备选**

决策理由：
1. **安全性优先**：Handlebars 默认无代码执行，符合安全沙箱要求
2. **快速验证**：学习成本低，PoC 阶段可快速产出结果
3. **扩展路径**：如 Handlebars 无法满足复杂需求，可平滑迁移至 Nunjucks
4. **社区生态**：两者均有活跃社区，长期维护有保障

### 2.3 验证步骤（可执行）

#### 2.3.1 环境准备

```bash
# 1. 创建 PoC 项目目录
mkdir -p poc/b1-template-engine
cd poc/b1-template-engine

# 2. 初始化 Node.js 项目
npm init -y

# 3. 安装候选方案依赖
npm install nunjucks handlebars

# 4. 安装格式转换依赖
npm install marked html-pdf-node docx

# 5. 安装测试框架
npm install jest benchmark
```

#### 2.3.2 测试模板定义

创建 `templates/test-template.html`：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{{task_name}} - 执行总结报告</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; }
        .header { background: #f0f0f0; padding: 20px; border-radius: 8px; }
        .score { font-size: 24px; color: {{#if (gte quality_score 90)}}green{{else}}orange{{/if}}; }
        table { width: 100%; border-collapse: collapse; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    </style>
</head>
<body>
    <div class="header">
        <h1>{{task_name}}</h1>
        <p>执行时间：{{execution_time}} | 报告日期：{{report_date}}</p>
        <p class="score">质量评分：{{quality_score}}/100</p>
    </div>
    
    <h2>关键决策</h2>
    {{#if key_decisions}}
    <table>
        <tr><th>决策</th><th>选择</th><th>影响</th></tr>
        {{#each key_decisions}}
        <tr>
            <td>{{title}}</td>
            <td>{{selected_option}}</td>
            <td>{{impact}}</td>
        </tr>
        {{/each}}
    </table>
    {{else}}
    <p>本次任务无重大决策记录。</p>
    {{/if}}
    
    <h2>问题解决</h2>
    <ul>
    {{#each problems_solved}}
        <li>{{this}}</li>
    {{/each}}
    </ul>
    
    {{#if suggestions}}
    <h2>改进建议</h2>
    <ul>
    {{#each suggestions}}
        <li>{{this}}</li>
    {{/each}}
    </ul>
    {{/if}}
    
    <footer>
        <p>作者：{{author_name}} | 团队：{{team_name}}</p>
    </footer>
</body>
</html>
```

#### 2.3.3 核心验证代码

创建 `src/engine-comparison.js`：

```javascript
const nunjucks = require('nunjucks');
const Handlebars = require('handlebars');
const marked = require('marked');
const htmlPdf = require('html-pdf-node');
const docx = require('docx');

// 注册 Handlebars 辅助函数
Handlebars.registerHelper('gte', function(a, b) {
    return a >= b;
});

// 测试数据（50+ 变量）
const testData = {
    task_name: "用户认证模块重构",
    task_type: "development",
    execution_time: "约 3 天 5 小时",
    report_date: "2026-04-14",
    quality_score: 94.5,
    author_name: "张三",
    team_name: "后端研发组",
    chapter_1_title: "任务目标与背景",
    chapter_1_content: "### 1.1 初始目标...",
    // ... 更多章节变量
    summary_text: "本次任务完成了用户认证模块的重构...",
    key_decisions: [
        {
            id: "D1",
            title: "认证方案选择",
            selected_option: "JWT + Refresh Token",
            impact: "高"
        },
        {
            id: "D2", 
            title: "密码加密算法",
            selected_option: "bcrypt (cost=12)",
            impact: "中"
        }
        // ... 更多决策
    ],
    problems_solved: [
        "Session 跨域共享问题",
        "Token 刷新机制设计",
        "密码强度校验规则"
    ],
    suggestions: [
        "引入 Redis 缓存 Token 黑名单",
        "增加登录失败告警机制"
    ],
    // 扩展变量
    custom_field_project_code: "PROJ-2026-042",
    custom_field_sprint_number: "Sprint 23",
    custom_field_reviewer: "李四",
    // ... 共 50+ 变量
};

class TemplateEnginePoC {
    constructor() {
        this.results = {
            nunjucks: {},
            handlebars: {}
        };
        this.templateHTML = [
            '<html><head><title>{{report_title}}</title></head>',
            '<body>',
            '<h1>{{report_title}}</h1>',
            '<p>任务名称: {{task_name}}</p>',
            '<p>执行时间: {{execution_time}}</p>',
            '<ul>',
            '{% for item in key_findings %}<li>{{item}}</li>{% endfor %}',
            '</ul>',
            '<table><tr><th>指标</th><th>值</th></tr>',
            '{% for metric in metrics %}<tr><td>{{metric.name}}</td><td>{{metric.value}}</td></tr>{% endfor %}',
            '</table>',
            '</body></html>'
        ].join('\n');
    }

    // 1. 渲染性能测试
    async benchmarkRendering() {
        const iterations = 1000;
        
        // Nunjucks 测试
        const njStart = process.hrtime.bigint();
        for (let i = 0; i < iterations; i++) {
            nunjucks.renderString(this.templateHTML, testData);
        }
        const njEnd = process.hrtime.bigint();
        this.results.nunjucks.renderTime = Number(njEnd - njStart) / 1e6 / iterations;
        
        // Handlebars 测试（预编译）
        const compiled = Handlebars.compile(this.templateHTML);
        const hbStart = process.hrtime.bigint();
        for (let i = 0; i < iterations; i++) {
            compiled(testData);
        }
        const hbEnd = process.hrtime.bigint();
        this.results.handlebars.renderTime = Number(hbEnd - hbStart) / 1e6 / iterations;
    }

    // 2. XSS 防护测试
    testXSSProtection() {
        const xssPayloads = [
            '<script>alert("xss")</script>',
            'javascript:alert("xss")',
            '<img src=x onerror=alert("xss")>',
            '{{constructor.constructor("alert(1)")()}}',
            '\\x3cscript\\x3ealert(1)\\x3c/script\\x3e'
        ];
        
        const maliciousData = {
            ...testData,
            task_name: xssPayloads[0],
            author_name: xssPayloads[1]
        };
        
        // 验证输出中是否包含未转义的脚本
        const output = Handlebars.compile(this.templateHTML)(maliciousData);
        const hasUnescapedScript = output.includes('<script>') || 
                                   output.includes('javascript:alert');
        
        this.results.xssProtection = !hasUnescapedScript;
        return this.results.xssProtection;
    }

    // 3. 多格式输出测试
    async testMultiFormatOutput() {
        const html = Handlebars.compile(this.templateHTML)(testData);
        
        // HTML → Markdown
        const markdown = this.htmlToMarkdown(html);
        
        // HTML → DOCX
        const docxBuffer = await this.htmlToDocx(html);
        
        // HTML → PDF
        const pdfBuffer = await this.htmlToPdf(html);
        
        this.results.formatFidelity = {
            markdown: this.calculateFidelity(html, markdown),
            docx: this.calculateFidelity(html, docxBuffer.toString()),
            pdf: true // PDF 为二进制格式，单独验证
        };
    }

    htmlToMarkdown(html) {
        // 简化实现，实际使用 turndown 等库
        return html.replace(/<[^>]+>/g, '');
    }

    async htmlToDocx(html) {
        const doc = new docx.Document({
            sections: [{
                properties: {},
                children: [
                    new docx.Paragraph({
                        children: [new docx.TextRun(html.substring(0, 1000))]
                    })
                ]
            }]
        });
        return docx.Packer.toBuffer(doc);
    }

    async htmlToPdf(html) {
        const file = { content: html };
        return htmlPdf.generatePdf(file, { format: 'A4' });
    }

    calculateFidelity(original, converted) {
        // 计算格式保真度（简化实现）
        const originalLength = original.length;
        const convertedLength = converted.length;
        return Math.min(100, (convertedLength / originalLength) * 100);
    }
}

module.exports = TemplateEnginePoC;
```

#### 2.3.4 自动化测试脚本

创建 `tests/benchmark.test.js`：

```javascript
const TemplateEnginePoC = require('../src/engine-comparison');

describe('B1 Template Engine PoC', () => {
    let poc;
    
    beforeAll(() => {
        poc = new TemplateEnginePoC();
    });

    test('渲染延迟应小于 200ms', async () => {
        await poc.benchmarkRendering();
        expect(poc.results.handlebars.renderTime).toBeLessThan(200);
        expect(poc.results.nunjucks.renderTime).toBeLessThan(200);
    });

    test('XSS 防护测试通过率 100%', () => {
        const passed = poc.testXSSProtection();
        expect(passed).toBe(true);
    });

    test('4种格式输出格式保真度 > 95%', async () => {
        await poc.testMultiFormatOutput();
        expect(poc.results.formatFidelity.markdown).toBeGreaterThan(95);
        expect(poc.results.formatFidelity.docx).toBeGreaterThan(95);
    });
});
```

### 2.4 成功标准（量化）

| 指标 | 目标值 | 测量方法 | 通过标准 |
|------|-------|---------|---------|
| **模板渲染延迟** | < 200ms | 1000 次渲染取平均 | 平均值 ≤ 200ms |
| **XSS 防护测试** | 100% 通过 | 5 种常见攻击向量 | 全部拦截 |
| **HTML 输出保真度** | > 95% | 结构完整性检查 | 标签闭合率 100% |
| **Markdown 输出保真度** | > 95% | 语义等价性对比 | 文本内容匹配度 ≥ 95% |
| **DOCX 输出保真度** | > 90% | 样式还原度检查 | 核心样式保留 |
| **PDF 输出保真度** | > 90% | 视觉对比 | 排版一致性 |
| **内存占用** | < 100MB | 渲染 100 个模板 | 峰值内存 ≤ 100MB |

### 2.5 风险点与缓解预案

| 风险编号 | 风险描述 | 发生概率 | 影响程度 | 缓解措施 |
|---------|---------|---------|---------|---------|
| B1-R1 | 复杂模板渲染超时 | 中 | 高 | 实现模板预编译缓存；设置渲染超时机制 |
| B1-R2 | XSS 绕过攻击 | 低 | 高 | 使用 DOMPurify 二次过滤；安全审计 |
| B1-R3 | DOCX/PDF 格式严重失真 | 中 | 中 | 引入专业转换库（pandoc）；降级为纯文本输出 |
| B1-R4 | 模板语法学习成本高 | 中 | 低 | 提供可视化编辑器；模板示例库 |
| B1-R5 | 大模板文件内存溢出 | 低 | 中 | 流式渲染；模板大小限制（< 5MB） |

### 2.6 时间估算

| 阶段 | 任务 | 时间 | 产出物 |
|------|------|------|-------|
| Day 1 | 环境搭建 + 方案调研 | 1 天 | 技术选型报告 |
| Day 2-3 | 核心引擎实现 | 2 天 | 可运行的原型 |
| Day 4-5 | 多格式转换实现 | 2 天 | 4 种格式输出 Demo |
| Day 6 | 安全测试 + 性能优化 | 1 天 | 测试报告 |
| Day 7 | 文档编写 + 评审 | 1 天 | PoC 总结报告 |
| **总计** | | **7 天** | |

---

## 3. 模块 B3: 外部工具集成 PoC

### 3.1 验证目标

#### 3.1.1 核心目标

1. **Confluence Cloud REST API v2 验证**：验证 Markdown → wiki 内容转换可行性
2. **Notion API Block 映射验证**：验证 Block 级别内容映射可行性
3. **OAuth 2.0 完整流程验证**：验证认证流程的可靠性和用户体验

#### 3.1.2 验证范围边界

| 范围维度 | 包含内容 | 排除内容 |
|---------|---------|---------|
| **目标平台** | Confluence Cloud、Notion、飞书 | MediaWiki、GitBook（P2优先级） |
| **API 版本** | Confluence REST API v2、Notion API v2022-06-28 | 旧版 API、私有 API |
| **内容类型** | Markdown 文本、表格、代码块、图片 | 复杂嵌套表格、视频嵌入 |
| **OAuth 流程** | Authorization Code + PKCE | Client Credentials、Device Flow |
| **同步模式** | 单向发布（报告 → 平台） | 双向同步、增量更新 |

### 3.2 技术方案选项对比

#### 3.2.1 平台 API 对比

| 平台 | API 类型 | 认证方式 | 速率限制 | Markdown 支持 |
|------|---------|---------|---------|--------------|
| **Confluence Cloud** | REST API v2 | OAuth 2.0 + PKCE | ~25 req/s | 需转换为 XHTML Storage Format |
| **Notion** | REST API | OAuth 2.0 + Internal | ~3 req/s | 需映射为 Block 结构 |
| **飞书 (Lark)** | Open API | OAuth 2.0 + App | ~20 req/s | 需转换为 Doc 格式 |

#### 3.2.2 详细对比矩阵

| 对比维度 | Confluence | Notion | 飞书 |
|---------|-----------|--------|------|
| **API 稳定性** | 高（Atlassian 企业级） | 高（官方维护） | 中（更新频繁） |
| **速率限制** | 25 req/s | 3 req/s（较低） | 20 req/s |
| **Markdown 原生支持** | 需转换 | 需 Block 映射 | 需转换 |
| **认证复杂度** | 中（PKCE 流程） | 低（标准 OAuth） | 中（App 权限） |
| **文档结构支持** | 完整（页面树） | 完整（Block 嵌套） | 中等 |
| **附件上传** | 支持（10MB/文件） | 支持（5MB/文件） | 支持（20MB/文件） |
| **Webhook 支持** | 支持 | 支持 | 支持 |
| **社区 SDK** | 丰富（多语言） | 丰富（官方 JS/TS） | 较少 |
| **企业合规** | 高（SOC2/ISO27001） | 高（SOC2） | 中（国内合规） |

### 3.3 验证步骤（可执行）

#### 3.3.1 环境准备

```bash
# 1. 创建 PoC 项目目录
mkdir -p poc/b3-tool-integration
cd poc/b3-tool-integration

# 2. 初始化项目
npm init -y

# 3. 安装依赖
npm install axios express simple-oauth2

# 4. 安装 Markdown 处理库
npm install marked turndown jsdom
```

#### 3.3.2 Confluence API 验证

**Step 1: OAuth 2.0 认证流程**

```javascript
// src/confluence-auth.js
const { AuthorizationCode } = require('simple-oauth2');

const config = {
    client: {
        id: process.env.CONFLUENCE_CLIENT_ID,
        secret: process.env.CONFLUENCE_CLIENT_SECRET
    },
    auth: {
        tokenHost: 'https://auth.atlassian.com',
        authorizePath: '/authorize',
        tokenPath: '/oauth/token'
    }
};

const client = new AuthorizationCode(config);

// 1. 生成授权 URL
function getAuthorizationUrl() {
    return client.authorizeURL({
        redirect_uri: 'http://localhost:3000/callback',
        scope: 'read:confluence-content write:confluence-content',
        state: 'random-state-string'
    });
}

// 2. 交换授权码获取 Token
async function getToken(code) {
    const tokenParams = {
        code,
        redirect_uri: 'http://localhost:3000/callback'
    };
    
    try {
        const accessToken = await client.getToken(tokenParams);
        return accessToken;
    } catch (error) {
        console.error('Token 获取失败:', error.message);
        throw error;
    }
}

module.exports = { getAuthorizationUrl, getToken };
```

**Step 2: API 调用测试（curl 示例）**

```bash
# 获取当前用户信息
curl -X GET \
  'https://api.atlassian.com/me' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Accept: application/json'

# 获取可访问的 Confluence 站点
curl -X GET \
  'https://api.atlassian.com/oauth/token/accessible-resources' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN'

# 创建页面（使用 REST API v2）
curl -X POST \
  'https://your-domain.atlassian.net/wiki/api/v2/pages' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "spaceId": "YOUR_SPACE_ID",
    "status": "current",
    "title": "任务执行总结报告 - 2026-04-14",
    "parentId": "PARENT_PAGE_ID",
    "body": {
      "representation": "storage",
      "value": "<h1>任务名称</h1><p>内容...</p>"
    }
  }'

# 上传附件
curl -X POST \
  'https://your-domain.atlassian.net/wiki/api/v2/attachments' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'X-Atlassian-Token: no-check' \
  -F 'file=@screenshot.png' \
  -F 'pageId=PAGE_ID'
```

**Step 3: Markdown → XHTML 转换**

```javascript
// src/markdown-to-confluence.js
const marked = require('marked');
const { JSDOM } = require('jsdom');

class MarkdownToConfluence {
    constructor() {
        this.renderer = new marked.Renderer();
        this.setupCustomRenderer();
    }

    setupCustomRenderer() {
        // 自定义表格渲染
        this.renderer.table = (header, body) => {
            return `<table class="confluenceTable">
                <colgroup><col /><col /></colgroup>
                <tbody>${header}${body}</tbody>
            </table>`;
        };

        // 自定义代码块渲染
        this.renderer.code = (code, language) => {
            return `<ac:structured-macro ac:name="code">
                <ac:parameter ac:name="language">${language || 'text'}</ac:parameter>
                <ac:plain-text-body><![CDATA[${code}]]></ac:plain-text-body>
            </ac:structured-macro>`;
        };

        // 自定义信息面板
        this.renderer.blockquote = (quote) => {
            return `<ac:structured-macro ac:name="info">
                <ac:rich-text-body>${quote}</ac:rich-text-body>
            </ac:structured-macro>`;
        };
    }

    convert(markdown) {
        // Markdown → HTML
        const html = marked.parse(markdown, { renderer: this.renderer });
        
        // HTML → Confluence Storage Format
        return this.toStorageFormat(html);
    }

    toStorageFormat(html) {
        // 转换为 Confluence XHTML Storage Format
        const dom = new JSDOM(html);
        const document = dom.window.document;
        
        // 处理图片引用
        const images = document.querySelectorAll('img');
        images.forEach(img => {
            const src = img.getAttribute('src');
            img.outerHTML = `<ac:image>
                <ri:attachment ri:filename="${src}" />
            </ac:image>`;
        });
        
        return document.body.innerHTML;
    }
}

module.exports = MarkdownToConfluence;
```

#### 3.3.3 Notion API 验证

**Step 1: Block 映射测试**

```javascript
// src/markdown-to-notion.js
class MarkdownToNotion {
    constructor() {
        this.blocks = [];
    }

    convert(markdown) {
        const lines = markdown.split('\n');
        
        for (const line of lines) {
            const block = this.parseLine(line);
            if (block) {
                this.blocks.push(block);
            }
        }
        
        return this.blocks;
    }

    parseLine(line) {
        // 标题
        if (line.startsWith('# ')) {
            return {
                object: 'block',
                type: 'heading_1',
                heading_1: {
                    rich_text: [{ type: 'text', text: { content: line.substring(2) } }]
                }
            };
        }
        if (line.startsWith('## ')) {
            return {
                object: 'block',
                type: 'heading_2',
                heading_2: {
                    rich_text: [{ type: 'text', text: { content: line.substring(3) } }]
                }
            };
        }
        if (line.startsWith('### ')) {
            return {
                object: 'block',
                type: 'heading_3',
                heading_3: {
                    rich_text: [{ type: 'text', text: { content: line.substring(4) } }]
                }
            };
        }

        // 无序列表
        if (line.startsWith('- ') || line.startsWith('* ')) {
            return {
                object: 'block',
                type: 'bulleted_list_item',
                bulleted_list_item: {
                    rich_text: [{ type: 'text', text: { content: line.substring(2) } }]
                }
            };
        }

        // 有序列表
        if (/^\d+\.\s/.test(line)) {
            const content = line.replace(/^\d+\.\s/, '');
            return {
                object: 'block',
                type: 'numbered_list_item',
                numbered_list_item: {
                    rich_text: [{ type: 'text', text: { content } }]
                }
            };
        }

        // 代码块
        if (line.startsWith('```')) {
            return { type: 'code_block_start', language: line.substring(3) };
        }

        // 普通段落
        if (line.trim()) {
            return {
                object: 'block',
                type: 'paragraph',
                paragraph: {
                    rich_text: [{ type: 'text', text: { content: line } }]
                }
            };
        }

        return null;
    }

    // 创建页面 API 调用
    async createPage(notion, parentDatabaseId, title, blocks) {
        return await notion.pages.create({
            parent: { database_id: parentDatabaseId },
            properties: {
                title: {
                    title: [{ text: { content: title } }]
                }
            },
            children: blocks
        });
    }
}

module.exports = MarkdownToNotion;
```

**Step 2: Notion API curl 示例**

```bash
# 创建页面
curl -X POST \
  'https://api.notion.com/v1/pages' \
  -H 'Authorization: Bearer YOUR_NOTION_TOKEN' \
  -H 'Notion-Version: 2022-06-28' \
  -H 'Content-Type: application/json' \
  -d '{
    "parent": { "database_id": "YOUR_DATABASE_ID" },
    "properties": {
      "title": {
        "title": [{ "text": { "content": "任务执行总结报告" } }]
      }
    },
    "children": [
      {
        "object": "block",
        "type": "heading_1",
        "heading_1": {
          "rich_text": [{ "type": "text", "text": { "content": "执行概览" } }]
        }
      },
      {
        "object": "block",
        "type": "paragraph",
        "paragraph": {
          "rich_text": [{ "type": "text", "text": { "content": "本次任务..." } }]
        }
      }
    ]
  }'

# 查询数据库
curl -X POST \
  'https://api.notion.com/v1/databases/YOUR_DATABASE_ID/query' \
  -H 'Authorization: Bearer YOUR_NOTION_TOKEN' \
  -H 'Notion-Version: 2022-06-28' \
  -H 'Content-Type: application/json' \
  -d '{"page_size": 10}'
```

### 3.4 成功标准（量化）

| 指标 | 目标值 | 测量方法 | 通过标准 |
|------|-------|---------|---------|
| **API 调用成功率** | > 99% | 100 次连续调用 | 失败次数 ≤ 1 |
| **OAuth 流程完成时间** | < 30s | 从点击授权到获取 Token | 平均时间 ≤ 30s |
| **Markdown 格式保真度** | > 90% | 对比原始与渲染后内容 | 语义一致性 ≥ 90% |
| **图片上传成功率** | > 98% | 50 张不同格式图片 | 失败次数 ≤ 1 |
| **速率限制遵守** | 100% | 监控 API 调用频率 | 无 429 错误 |
| **Token 刷新成功率** | 100% | 模拟 Token 过期场景 | 自动刷新成功 |
| **错误恢复时间** | < 5s | 模拟网络中断后恢复 | 自动重连成功 |

### 3.5 风险点与缓解预案

| 风险编号 | 风险描述 | 发生概率 | 影响程度 | 缓解措施 |
|---------|---------|---------|---------|---------|
| B3-R1 | Confluence 速率限制（25 req/s） | 高 | 中 | 实现请求队列 + 指数退避重试 |
| B3-R2 | Notion 速率限制（3 req/s） | 高 | 高 | 批量 Block 创建；缓存策略 |
| B3-R3 | Markdown 复杂格式丢失 | 中 | 中 | 降级策略（纯文本输出）；格式映射表 |
| B3-R4 | OAuth Token 过期未刷新 | 中 | 高 | 自动刷新机制；Token 有效期监控 |
| B3-R5 | 第三方 API 变更导致失效 | 中 | 高 | 抽象层隔离；API 版本锁定 |
| B3-R6 | 大附件上传超时 | 中 | 低 | 分片上传；异步处理 |

### 3.6 时间估算

| 阶段 | 任务 | 时间 | 产出物 |
|------|------|------|-------|
| Day 1-2 | Confluence OAuth + API 验证 | 2 天 | Confluence 发布 Demo |
| Day 3-4 | Notion API 验证 | 2 天 | Notion 发布 Demo |
| Day 5-6 | 飞书 API 验证 + Markdown 转换 | 2 天 | 飞书发布 Demo |
| Day 7 | 速率限制处理 + 错误恢复 | 1 天 | 鲁棒性测试报告 |
| Day 8-9 | 集成测试 + 性能优化 | 2 天 | 性能测试报告 |
| Day 10 | 文档编写 + 评审 | 1 天 | PoC 总结报告 |
| **总计** | | **10 天** | |

---

## 4. 模块 B5: 团队协作引擎 PoC

### 4.1 验证目标

#### 4.1.1 核心目标

1. **实时协同编辑冲突解决**：验证 CRDT/OT 算法在报告编辑场景的有效性
2. **RBAC 权限模型性能**：验证细粒度权限检查的响应时间
3. **审批工作流状态机**：验证状态转换的正确性和完整性

#### 4.1.2 验证范围边界

| 范围维度 | 包含内容 | 排除内容 |
|---------|---------|---------|
| **协作算法** | Yjs (CRDT)、Automerge (CRDT)、ShareDB (OT) | 自研算法（风险过高） |
| **权限模型** | RBAC（5 级角色） | ABAC、ACL（后续扩展） |
| **工作流引擎** | YAML DSL 定义、状态机执行 | 可视化流程设计器 |
| **并发测试** | 10 并发用户 | 100+ 并发（正式阶段） |
| **离线支持** | 基础离线编辑 | 复杂离线合并（后续扩展） |

### 4.2 技术方案选项对比

#### 4.2.1 实时协作方案对比

| 方案 | 技术基础 | 核心特点 | 适用场景 |
|------|---------|---------|---------|
| **Yjs** | CRDT (YATA 算法) | 性能优秀，生态丰富，支持多种编辑器 | 大型文档、高并发 |
| **Automerge** | CRDT (RGA 算法) | API 简洁，数据格式清晰，易于调试 | 中小型应用、快速开发 |
| **ShareDB** | OT (JSON0) | 成熟稳定，基于 MongoDB，查询能力强 | 已有 MongoDB 基础设施 |

#### 4.2.2 详细对比矩阵

| 对比维度 | Yjs (CRDT) | Automerge (CRDT) | ShareDB (OT) |
|---------|-----------|-----------------|-------------|
| **并发性能** | 极高（10k+ 用户） | 高（1000+ 用户） | 高（1000+ 用户） |
| **离线支持** | 原生支持 | 原生支持 | 需额外实现 |
| **文档膨胀控制** | 优秀（定期 GC） | 良好 | 不涉及 |
| **学习成本** | 中（需理解 CRDT） | 低（API 简洁） | 中（需理解 OT） |
| **包大小** | ~50KB | ~80KB | ~100KB（含依赖） |
| **社区活跃度** | 高（GitHub 18k+ stars） | 中（GitHub 5k+ stars） | 中（GitHub 3k+ stars） |
| **浏览器支持** | 现代浏览器 | 现代浏览器 | 现代浏览器 |
| **服务端要求** | WebSocket 服务器 | WebSocket 服务器 | MongoDB + Node.js |
| **调试工具** | 良好（yjs-demos） | 良好 | 一般 |
| **合并冲突处理** | 自动（无冲突概念） | 自动 | 需处理冲突 |

#### 4.2.3 权限系统方案对比

| 方案 | 技术基础 | 核心特点 | 适用场景 |
|------|---------|---------|---------|
| **Casbin** | 策略引擎 | 功能强大，多语言支持，RBAC/ABAC 原生 | 复杂权限需求 |
| **自研 RBAC** | 自定义实现 | 轻量可控，无外部依赖，针对性强 | 简单固定权限模型 |

| 对比维度 | Casbin | 自研 RBAC |
|---------|--------|----------|
| **权限检查延迟** | ~2-5ms | ~1-3ms |
| **功能丰富度** | 高（支持 ABAC、RBAC） | 中（仅 RBAC） |
| **学习成本** | 中（需理解模型语法） | 低（自定义逻辑） |
| **包大小** | ~100KB | ~20KB |
| **维护成本** | 低（社区维护） | 高（自行维护） |

#### 4.2.4 审批引擎方案对比

| 方案 | 技术基础 | 核心特点 | 适用场景 |
|------|---------|---------|---------|
| **XState** | 状态机库 | 可视化、可测试、TypeScript 友好 | 前端状态管理 |
| **Temporal** | 工作流引擎 | 持久化、分布式、企业级 | 复杂长时间工作流 |
| **自研状态机** | 自定义实现 | 轻量、完全可控 | 简单审批流 |

### 4.3 验证步骤（可执行）

#### 4.3.1 环境准备

```bash
# 1. 创建 PoC 项目目录
mkdir -p poc/b5-collaboration-engine
cd poc/b5-collaboration-engine

# 2. 初始化项目
npm init -y

# 3. 安装实时协作库
npm install yjs y-websocket y-protocols

# 4. 安装权限系统
npm install casbin

# 5. 安装状态机库
npm install xstate

# 6. 安装测试工具
npm install jest artillery
```

#### 4.3.2 实时协作验证（Yjs）

```javascript
// src/collaboration/yjs-poc.js
const Y = require('yjs');
const { WebsocketProvider } = require('y-websocket');

class CollaborationPoC {
    constructor() {
        this.docs = new Map();
        this.providers = new Map();
    }

    // 1. 创建协作文档
    createDocument(docId) {
        const doc = new Y.Doc();
        
        // 创建共享文本类型（用于报告内容）
        const ytext = doc.getText('report-content');
        
        // 创建共享映射（用于元数据）
        const ymap = doc.getMap('report-metadata');
        
        this.docs.set(docId, { doc, ytext, ymap });
        
        return { doc, ytext, ymap };
    }

    // 2. 模拟并发编辑
    simulateConcurrentEdits(docId, numUsers = 10) {
        const { ytext } = this.docs.get(docId);
        const operations = [];
        
        // 模拟 10 个用户同时编辑
        for (let i = 0; i < numUsers; i++) {
            const userId = `user-${i}`;
            const position = Math.floor(Math.random() * ytext.length);
            const content = `[编辑-${i}]`;
            
            operations.push({
                userId,
                position,
                content,
                timestamp: Date.now()
            });
            
            // 执行编辑
            ytext.insert(position, content);
        }
        
        return operations;
    }

    // 3. 冲突解决测试
    testConflictResolution() {
        const docId = 'conflict-test';
        const { doc, ytext } = this.createDocument(docId);
        
        // 初始内容
        ytext.insert(0, 'Hello World');
        
        // 模拟两个用户同时编辑同一位置
        const state1 = Y.encodeStateAsUpdate(doc);
        
        // 用户 A：在位置 5 插入 ", beautiful"
        ytext.insert(5, ', beautiful');
        const updateA = Y.encodeStateAsUpdate(doc);
        
        // 重置并应用用户 B 的操作
        const docB = new Y.Doc();
        Y.applyUpdate(docB, state1);
        const ytextB = docB.getText('report-content');
        
        // 用户 B：在位置 11 插入 "!"
        ytextB.insert(11, '!');
        const updateB = Y.encodeStateAsUpdate(docB);
        
        // 合并两个更新
        Y.applyUpdate(doc, updateB);
        Y.applyUpdate(docB, updateA);
        
        // 验证最终一致性
        const finalA = ytext.toString();
        const finalB = ytextB.toString();
        
        return {
            consistent: finalA === finalB,
            content: finalA,
            expected: 'Hello, beautiful World!'
        };
    }

    // 4. 文档膨胀测试
    testDocumentGrowth(iterations = 1000) {
        const doc = new Y.Doc();
        const ytext = doc.getText('report-content');
        
        const sizes = [];
        
        for (let i = 0; i < iterations; i++) {
            // 插入内容
            ytext.insert(ytext.length, `Line ${i}\n`);
            
            // 删除部分内容（模拟编辑）
            if (i > 100 && i % 10 === 0) {
                ytext.delete(0, 10);
            }
            
            // 记录文档大小
            const update = Y.encodeStateAsUpdate(doc);
            sizes.push(update.length);
        }
        
        return {
            initialSize: sizes[0],
            finalSize: sizes[sizes.length - 1],
            growthRate: sizes[sizes.length - 1] / sizes[0],
            sizes
        };
    }

    // 5. 离线支持测试
    testOfflineSupport() {
        const doc = new Y.Doc();
        const ytext = doc.getText('report-content');
        
        // 离线编辑
        ytext.insert(0, 'Offline edit 1\n');
        ytext.insert(ytext.length, 'Offline edit 2\n');
        
        const offlineUpdate = Y.encodeStateAsUpdate(doc);
        
        // 模拟重新连接，合并到服务端文档
        const serverDoc = new Y.Doc();
        serverDoc.getText('report-content').insert(0, 'Server content\n');
        
        Y.applyUpdate(serverDoc, offlineUpdate);
        
        return {
            mergedContent: serverDoc.getText('report-content').toString(),
            offlineChangesApplied: true
        };
    }
}

module.exports = CollaborationPoC;
```

#### 4.3.3 RBAC 权限验证

```javascript
// src/authorization/rbac-poc.js
const { newEnforcer } = require('casbin');

class RBACPoC {
    constructor() {
        this.enforcer = null;
    }

    async initialize() {
        // 初始化 Casbin（使用内存存储）
        this.enforcer = await newEnforcer(
            'src/authorization/rbac_model.conf',
            'src/authorization/rbac_policy.csv'
        );
    }

    // 1. 权限检查性能测试
    async benchmarkPermissionCheck(iterations = 10000) {
        const start = process.hrtime.bigint();
        
        for (let i = 0; i < iterations; i++) {
            const user = `user-${i % 100}`;
            const resource = `report-${i % 1000}`;
            const action = ['read', 'write', 'delete'][i % 3];
            
            await this.enforcer.enforce(user, resource, action);
        }
        
        const end = process.hrtime.bigint();
        const totalMs = Number(end - start) / 1e6;
        
        return {
            totalTime: totalMs,
            averageTime: totalMs / iterations,
            iterations
        };
    }

    // 2. 角色层次测试
    async testRoleHierarchy() {
        // 定义角色层次：ADMIN > OWNER > EDITOR > REVIEWER > VIEWER
        const roleHierarchy = [
            ['OWNER', 'EDITOR'],
            ['EDITOR', 'REVIEWER'],
            ['REVIEWER', 'VIEWER']
        ];
        
        for (const [parent, child] of roleHierarchy) {
            await this.enforcer.addGroupingPolicy(child, parent);
        }
        
        // 测试权限继承
        const tests = [
            { user: 'admin', role: 'ADMIN', resource: 'report-1', action: 'delete', expected: true },
            { user: 'owner', role: 'OWNER', resource: 'report-1', action: 'write', expected: true },
            { user: 'viewer', role: 'VIEWER', resource: 'report-1', action: 'write', expected: false }
        ];
        
        const results = [];
        for (const test of tests) {
            await this.enforcer.addGroupingPolicy(test.user, test.role);
            const result = await this.enforcer.enforce(test.user, test.resource, test.action);
            results.push({
                ...test,
                actual: result,
                passed: result === test.expected
            });
        }
        
        return results;
    }

    // 3. 资源级权限测试
    async testResourceLevelPermissions() {
        // 为特定资源设置权限
        await this.enforcer.addPolicy('alice', 'report-1', 'write');
        await this.enforcer.addPolicy('alice', 'report-2', 'read');
        await this.enforcer.addPolicy('bob', 'report-1', 'read');
        
        const tests = [
            { user: 'alice', resource: 'report-1', action: 'write', expected: true },
            { user: 'alice', resource: 'report-2', action: 'write', expected: false },
            { user: 'bob', resource: 'report-1', action: 'write', expected: false }
        ];
        
        const results = [];
        for (const test of tests) {
            const result = await this.enforcer.enforce(test.user, test.resource, test.action);
            results.push({
                ...test,
                actual: result,
                passed: result === test.expected
            });
        }
        
        return results;
    }
}

// RBAC 模型配置（rbac_model.conf）
const rbacModelConfig = `
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
`;

// RBAC 策略配置（rbac_policy.csv）
const rbacPolicyConfig = `
p, ADMIN, *, *
p, OWNER, *, read
p, OWNER, *, write
p, EDITOR, *, read
p, EDITOR, *, write
p, REVIEWER, *, read
p, VIEWER, *, read
`;

module.exports = { RBACPoC, rbacModelConfig, rbacPolicyConfig };
```

#### 4.3.4 审批工作流验证

```javascript
// src/workflow/workflow-poc.js
const { createMachine, interpret } = require('xstate');

class WorkflowPoC {
    constructor() {
        this.machine = this.createWorkflowMachine();
    }

    // 1. 创建工作流状态机
    createWorkflowMachine() {
        return createMachine({
            id: 'report-approval',
            initial: 'draft',
            states: {
                draft: {
                    on: {
                        SUBMIT: {
                            target: 'reviewing',
                            cond: 'canSubmit'
                        }
                    }
                },
                reviewing: {
                    on: {
                        APPROVE: {
                            target: 'approved',
                            cond: 'hasRequiredApprovals',
                            actions: 'recordApproval'
                        },
                        REQUEST_CHANGES: {
                            target: 'revisionRequired',
                            actions: 'notifyAuthor'
                        },
                        REJECT: {
                            target: 'rejected',
                            cond: 'canReject'
                        }
                    }
                },
                revisionRequired: {
                    on: {
                        RESUBMIT: {
                            target: 'reviewing',
                            cond: 'canResubmit'
                        }
                    }
                },
                approved: {
                    on: {
                        PUBLISH: {
                            target: 'published',
                            cond: 'canPublish'
                        }
                    }
                },
                published: {
                    type: 'final'
                },
                rejected: {
                    type: 'final'
                }
            }
        }, {
            guards: {
                canSubmit: (context, event) => {
                    return context.userRole === 'EDITOR' || context.userRole === 'OWNER';
                },
                hasRequiredApprovals: (context) => {
                    return context.approvals >= context.requiredApprovals;
                },
                canReject: (context, event) => {
                    return context.userRole === 'ADMIN';
                },
                canResubmit: (context) => {
                    return context.userRole === 'EDITOR' || context.userRole === 'OWNER';
                },
                canPublish: (context) => {
                    return context.userRole === 'ADMIN' || context.userRole === 'OWNER';
                }
            },
            actions: {
                recordApproval: (context) => {
                    context.approvals++;
                    context.approvalHistory.push({
                        user: context.currentUser,
                        timestamp: Date.now()
                    });
                },
                notifyAuthor: (context) => {
                    context.notifications.push({
                        type: 'REVISION_REQUIRED',
                        to: context.author,
                        timestamp: Date.now()
                    });
                }
            }
        });
    }

    // 2. 状态转换测试
    testStateTransitions() {
        const testCases = [
            {
                name: '标准审批流程',
                events: ['SUBMIT', 'APPROVE', 'PUBLISH'],
                context: {
                    userRole: 'EDITOR',
                    approvals: 1,
                    requiredApprovals: 1
                },
                expectedState: 'published'
            },
            {
                name: '需要修改流程',
                events: ['SUBMIT', 'REQUEST_CHANGES', 'RESUBMIT', 'APPROVE', 'PUBLISH'],
                context: {
                    userRole: 'EDITOR',
                    approvals: 1,
                    requiredApprovals: 1
                },
                expectedState: 'published'
            },
            {
                name: '拒绝流程',
                events: ['SUBMIT', 'REJECT'],
                context: {
                    userRole: 'ADMIN',
                    approvals: 0,
                    requiredApprovals: 1
                },
                expectedState: 'rejected'
            }
        ];

        const results = [];
        
        for (const testCase of testCases) {
            const service = interpret(this.machine.withContext(testCase.context));
            service.start();
            
            for (const event of testCase.events) {
                service.send(event);
            }
            
            const finalState = service.state.value;
            results.push({
                name: testCase.name,
                expected: testCase.expectedState,
                actual: finalState,
                passed: finalState === testCase.expectedState
            });
            
            service.stop();
        }
        
        return results;
    }

    // 3. 状态覆盖率测试
    testStateCoverage() {
        const states = ['draft', 'reviewing', 'revisionRequired', 'approved', 'published', 'rejected'];
        const coveredStates = new Set();
        
        // 遍历所有可能的状态转换
        const service = interpret(this.machine);
        service.start();
        
        coveredStates.add(service.state.value);
        
        // 模拟各种转换
        const transitions = [
            { from: 'draft', event: 'SUBMIT', to: 'reviewing' },
            { from: 'reviewing', event: 'APPROVE', to: 'approved' },
            { from: 'reviewing', event: 'REQUEST_CHANGES', to: 'revisionRequired' },
            { from: 'reviewing', event: 'REJECT', to: 'rejected' },
            { from: 'revisionRequired', event: 'RESUBMIT', to: 'reviewing' },
            { from: 'approved', event: 'PUBLISH', to: 'published' }
        ];
        
        for (const transition of transitions) {
            // 重置到初始状态
            service.stop();
            const ctxService = interpret(this.machine.withContext({
                userRole: 'ADMIN',
                approvals: 1,
                requiredApprovals: 1
            }));
            ctxService.start();
            
            // 执行转换
            if (transition.from === 'draft') {
                ctxService.send('SUBMIT');
            }
            
            if (transition.from === 'reviewing') {
                ctxService.send('SUBMIT');
                ctxService.send(transition.event);
            }
            
            coveredStates.add(ctxService.state.value);
            ctxService.stop();
        }
        
        const coverage = (coveredStates.size / states.length) * 100;
        
        return {
            totalStates: states.length,
            coveredStates: coveredStates.size,
            coverage: coverage,
            missingStates: states.filter(s => !coveredStates.has(s))
        };
    }
}

module.exports = WorkflowPoC;
```

### 4.4 成功标准（量化）

| 指标 | 目标值 | 测量方法 | 通过标准 |
|------|-------|---------|---------|
| **并发编辑数据一致性** | 100% | 10 并发用户，1000 次操作 | 最终文档一致 |
| **权限检查延迟** | < 5ms | 10000 次检查取平均 | 平均值 ≤ 5ms |
| **审批状态覆盖率** | 100% | 所有状态转换路径 | 覆盖所有状态 |
| **文档膨胀率** | < 2x | 1000 次编辑后大小 | 最终大小 ≤ 2x 初始大小 |
| **离线编辑恢复成功率** | 100% | 50 次离线场景 | 全部成功合并 |
| **冲突解决成功率** | 100% | 100 次并发冲突场景 | 自动解决无数据丢失 |
| **工作流状态转换延迟** | < 50ms | 1000 次状态转换 | 平均值 ≤ 50ms |

### 4.5 风险点与缓解预案

| 风险编号 | 风险描述 | 发生概率 | 影响程度 | 缓解措施 |
|---------|---------|---------|---------|---------|
| B5-R1 | CRDT 文档膨胀失控 | 中 | 高 | 定期 GC；文档大小监控；历史版本归档 |
| B5-R2 | 权限检查性能瓶颈 | 中 | 高 | 缓存策略；批量检查；异步处理 |
| B5-R3 | 审批流死锁 | 低 | 高 | 超时机制；人工介入接口；状态监控 |
| B5-R4 | 离线编辑冲突无法自动解决 | 中 | 中 | 用户仲裁界面；冲突标记 |
| B5-R5 | WebSocket 连接不稳定 | 中 | 中 | 自动重连；操作队列；断线提示 |
| B5-R6 | 并发编辑性能下降 | 中 | 中 | 操作节流；增量同步；服务端优化 |

### 4.6 时间估算

| 阶段 | 任务 | 时间 | 产出物 |
|------|------|------|-------|
| Day 1-2 | Yjs 集成 + 基础协作功能 | 2 天 | 实时协作 Demo |
| Day 3-4 | 冲突解决 + 离线支持 | 2 天 | 鲁棒性测试报告 |
| Day 5-6 | RBAC 权限系统实现 | 2 天 | 权限检查 Demo |
| Day 7-8 | 审批工作流引擎 | 2 天 | 工作流 Demo |
| Day 9-10 | 集成测试 + 性能优化 | 2 天 | 性能测试报告 |
| Day 11-12 | 文档编写 + 评审 | 2 天 | PoC 总结报告 |
| **总计** | | **12 天** | |

---

## 5. PoC 执行总计划

### 5.1 甘特图（文本形式）

```
周次:      Week 1      Week 2      Week 3      Week 4      Week 5      Week 6      Week 7      Week 8
          ├───────────┼───────────┼───────────┼───────────┼───────────┼───────────┼───────────┤

B1 模板    ████████
引擎 PoC   Day 1-7

B3 集成            ████████████
引擎 PoC           Day 8-17

B5 协作                        ████████████████████
引擎 PoC                       Day 18-29

集成测试                                   ████████████
& 评审                                     Day 25-36

里程碑:    M1(D7)    M2(D17)   M3(D29)              M4(D36)

图例:
████ = 开发活跃期
M# = Go/No-Go 决策节点
```

### 5.2 总工时预算

| 资源类型 | 人数 | 投入比例 | 总工时 | 成本估算 |
|---------|------|---------|--------|---------|
| 全栈工程师 | 1 | 100% | 8周 × 5天 × 8h = 320h | ¥64,000 |
| 后端工程师 | 1 | 100% | 8周 × 5天 × 8h = 320h | ¥64,000 |
| 前端工程师 | 1 | 50% | 4周 × 5天 × 8h = 160h | ¥32,000 |
| **总计** | **3** | | **800h** | **¥160,000** |

### 5.3 资源需求

#### 5.3.1 人员需求

| 角色 | 技能要求 | 职责 | 投入时间 |
|------|---------|------|---------|
| **技术负责人** | 全栈开发、架构设计 | 技术选型、方案评审、风险评估 | 全程 |
| **后端工程师** | Node.js、API 集成 | B3 工具集成、B5 服务端实现 | 全程 |
| **前端工程师** | JavaScript、实时协作 | B1 模板引擎、B5 客户端实现 | 50% |

#### 5.3.2 环境需求

| 环境类型 | 配置要求 | 用途 | 数量 |
|---------|---------|------|------|
| 开发环境 | 8核 16GB | 日常开发 | 3 套 |
| 测试环境 | 4核 8GB | 自动化测试 | 1 套 |
| PoC 演示环境 | 4核 8GB | 评审演示 | 1 套 |

#### 5.3.3 第三方账号需求

| 平台 | 账号类型 | 用途 | 获取方式 |
|------|---------|------|---------|
| Confluence Cloud | Developer 账号 | API 测试 | Atlassian 开发者平台 |
| Notion | Integration 账号 | API 测试 | Notion Integrations |
| 飞书 | 企业测试账号 | API 测试 | 飞书开放平台 |

### 5.4 Go/No-Go 决策节点定义

| 节点 | 时间点 | 决策内容 | 通过标准 | 失败预案 |
|------|-------|---------|---------|---------|
| **M1** | Day 7 | B1 模板引擎 Go/No-Go | 渲染延迟<200ms，XSS 100% 拦截 | 降级为固定模板；使用服务端渲染 |
| **M2** | Day 17 | B3 工具集成 Go/No-Go | API 成功率>99%，OAuth<30s | 减少支持平台；延长超时时间 |
| **M3** | Day 29 | B5 协作引擎 Go/No-Go | 10 并发零丢失，权限<5ms | 降级为异步协作；简化权限模型 |
| **M4** | Day 36 | 整体 Go/No-Go | 2/3 模块通过 | 调整 V3 范围；延长 Phase 1 周期 |

---

## 6. 附录

### 6.1 参考资源链接

#### 6.1.1 官方文档

| 资源 | 链接 | 用途 |
|------|------|------|
| Handlebars 文档 | <https://handlebarsjs.com/> | B1 模板引擎参考 |
| Nunjucks 文档 | <https://mozilla.github.io/nunjucks/> | B1 备选方案参考 |
| Confluence REST API | <https://developer.atlassian.com/cloud/confluence/rest/v2/> | B3 Confluence 集成 |
| Notion API | <https://developers.notion.com/> | B3 Notion 集成 |
| 飞书开放平台 | <https://open.feishu.cn/> | B3 飞书集成 |
| Yjs 文档 | <https://docs.yjs.dev/> | B5 实时协作 |
| Casbin 文档 | <https://casbin.org/> | B5 权限系统 |
| XState 文档 | <https://xstate.js.org/> | B5 工作流引擎 |

#### 6.1.2 技术参考

| 资源 | 链接 | 用途 |
|------|------|------|
| CRDT 技术综述 | <https://crdt.tech/> | B5 协作算法理论 |
| OT vs CRDT | <https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/> | 技术选型参考 |
| OAuth 2.0 + PKCE | <https://oauth.net/2/pkce/> | B3 认证流程 |
| RBAC 最佳实践 | <https://csrc.nist.gov/publications/detail/sp/800-162/final> | B5 权限设计 |

### 6.2 术语表

| 术语 | 全称 | 定义 |
|------|------|------|
| **PoC** | Proof of Concept | 概念验证，用于验证技术方案可行性的原型实现 |
| **CRDT** | Conflict-free Replicated Data Type | 无冲突复制数据类型，用于分布式系统的数据同步 |
| **OT** | Operational Transformation | 操作变换算法，用于实时协作中的并发编辑处理 |
| **RBAC** | Role-Based Access Control | 基于角色的访问控制 |
| **XSS** | Cross-Site Scripting | 跨站脚本攻击，Web 安全漏洞类型 |
| **OAuth 2.0** | Open Authorization 2.0 | 开放授权协议，用于第三方应用授权 |
| **PKCE** | Proof Key for Code Exchange | 密钥交换证明，OAuth 2.0 的安全扩展 |
| **DSL** | Domain-Specific Language | 领域特定语言，针对特定问题域的专用语言 |
| **Rate Limit** | Rate Limiting | 速率限制，API 调用的频率控制机制 |
| **Go/No-Go** | Go/No-Go Decision | 继续/停止决策点，项目里程碑评审机制 |

---

**文档结束**

*本文档为 V3 关键技术 PoC 验证的规划版本，具体执行细节可能根据实际进展进行调整。*

*最后更新: 2026-04-14*
*维护者: Task Execution Summary Generator Team*
