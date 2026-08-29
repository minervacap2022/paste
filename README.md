# 文本处理服务 (Paste Service) 开发文档

## 模块简介

本模块提供了一个轻量级的文本处理服务，主要功能是接收用户输入的文本，去除其中的 `<br/>`、`<br>`、`<br />` 标签，并将其替换为标准的换行符 `\n`。

此服务包含：
1. **前端界面**: 简洁的Web界面，供用户直接粘贴和处理文本。
2. **后端API**: `/api/paste/process`，处理核心逻辑并负责数据记录。
3. **日志记录**: 所有处理请求均会被记录，包含原始文本、处理后文本及元数据。

## 部署与调用方式

### 1. 现有系统集成

本服务已集成到 `panoramic-intelligence` 主服务中。

- **访问地址**: `https://www.panor.tech/paste`
- **API端点**: `POST https://www.panor.tech/api/paste/process`

### 2. API 调用规范

未来其他模块或第三方系统调用此服务时，请遵循以下规范：

**请求:**
- **URL**: `/api/paste/process`
- **Method**: `POST`
- **Headers**: `Content-Type: application/json`
- **Body**:
  ```json
  {
    "text": "包含<br>标签的<br/>原始文本"
  }
  ```

**响应:**
- **Status**: `200 OK`
- **Body**:
  ```json
  {
    "success": true,
    "processedText": "包含\n标签的\n原始文本",
    "message": "Processed and logged successfully"
  }
  ```

### 3. 未来开发与集成

#### 前端集成
如果需要在其他页面（如 CMS、评论系统）中使用此功能，建议封装为一个通用的 JavaScript 工具函数：

```javascript
async function cleanTextFormat(rawText) {
    const response = await fetch('/api/paste/process', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text: rawText })
    });
    const result = await response.json();
    return result.processedText;
}
```

#### 后端微服务化
当前服务作为 `express` 路由的一部分运行。随着负载增加，可以将其拆分为独立的微服务：
1. 将 `server.js` 中的 `/api/paste/process` 路由逻辑提取到独立的 Node.js 服务中。
2. 使用 Nginx 反向代理将 `/paste` 路径流量指向新服务端口。

### 4. 数据库集成与数据追踪

为了满足 "make sure all kinds of data is stored and tracked" 的要求，当前版本采用了文件日志系统。

#### 当前存储机制
- **位置**: `/www/wwwroot/www.panor.tech/logs/paste/YYYY-MM-DD.jsonl`
- **格式**: JSONL (每行一个 JSON 对象)
- **内容**: 时间戳、IP地址、输入长度、输出长度、原始文本(fullInput)、处理后文本(fullOutput)。

#### 未来数据库规划
随着数据量增长，建议迁移至结构化数据库（如 MongoDB 或 MySQL）：

**MongoDB Schema 设计建议:**

```javascript
const PasteLogSchema = new Schema({
    createdAt: { type: Date, default: Date.now },
    clientIp: String,
    metadata: {
        userAgent: String,
        referer: String
    },
    content: {
        original: String,     // 存储原始输入
        processed: String,    // 存储处理结果
        originalLength: Number,
        processedLength: Number
    },
    processingTimeMs: Number  // 性能追踪
});
```

**集成步骤:**
1. 在 `server/` 目录下引入 `mongoose` 或其他 ORM。
2. 在 `/api/paste/process` 路由中，替换 `fs.appendFile` 为数据库插入操作。
3. 添加定期归档任务，将旧数据迁移至冷存储（如 AWS S3），以降低数据库成本。

## 维护与监控

- **日志查看**: 检查 `logs/paste/` 目录下的日志文件以监控使用情况。
- **错误排查**: 任何 API 错误都会记录在 PM2 的错误日志中 (`logs/err.log`)。



<!-- minervacap-pre-hiklik-promotion -->
> **Discover Klik:** https://pre.hiklik.ai/
<!-- /minervacap-pre-hiklik-promotion -->
