# 网课助手题库 API 文档

## API 概览

网课助手题库 API 是一个为在线教育平台提供自动答题和学习辅助功能的接口系统。该 API 设计遵循以下原则：

- **简单性**：API 接口设计简洁明了，易于集成和使用
- **可扩展性**：支持多种题库来源和答题模式
- **安全性**：使用 token 认证机制确保 API 调用的安全性
- **高效性**：优化的请求处理流程，确保快速响应

本 API 主要包含题库查询、智能答题和 AI 辅助功能三大模块，为用户提供全面的学习辅助服务。

## API 端点详细说明

### 1. 题库查询 API

#### 1.1 题目搜索

- **接口路径**：`/search`
- **请求方法**：POST
- **请求参数**：

| 参数名 | 类型 | 必填 | 描述 | 示例值 |
|--------|------|------|------|--------|
| question | String | 是 | 题目内容 | "中国的首都是哪个城市？" |
| key | String | 是 | API 访问令牌 | "your_api_token" |
| platform | String | 否 | 平台标识 | "cx" |

- **响应格式**：

成功响应：
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "question": "中国的首都是哪个城市？",
    "answer": "北京",
    "analysis": "北京是中华人民共和国的首都",
    "options": ["上海", "北京", "广州", "深圳"],
    "type": "single"
  }
}
```

错误响应：
```json
{
  "code": 1001,
  "message": "题目不存在",
  "data": null
}
```

- **使用示例**：

```javascript
// 使用 GM_xmlhttpRequest 发送请求
GM_xmlhttpRequest({
  method: "POST",
  url: "http://api.tikuhai.com/search",
  headers: {
    "Content-Type": "application/json"
  },
  data: JSON.stringify({
    "question": "中国的首都是哪个城市？",
    "key": configStore.queryApis[0].token,
    "platform": "cx"
  }),
  onload: function(response) {
    const result = JSON.parse(response.responseText);
    if (result.code === 0) {
      // 处理成功响应
      console.log("答案:", result.data.answer);
    } else {
      // 处理错误响应
      console.error("查询失败:", result.message);
    }
  }
});
```

#### 1.2 批量题目查询

- **接口路径**：`/batch`
- **请求方法**：POST
- **请求参数**：

| 参数名 | 类型 | 必填 | 描述 | 示例值 |
|--------|------|------|------|--------|
| questions | Array | 是 | 题目内容数组 | ["题目1", "题目2"] |
| key | String | 是 | API 访问令牌 | "your_api_token" |
| platform | String | 否 | 平台标识 | "cx" |

- **响应格式**：

成功响应：
```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "question": "题目1",
      "answer": "答案1",
      "analysis": "解析1",
      "options": ["选项A", "选项B", "选项C", "选项D"],
      "type": "single"
    },
    {
      "question": "题目2",
      "answer": "答案2",
      "analysis": "解析2",
      "options": ["选项A", "选项B"],
      "type": "multiple"
    }
  ]
}
```

错误响应：
```json
{
  "code": 1002,
  "message": "批量查询失败",
  "data": null
}
```

### 2. AI 辅助功能 API 

#### 2.1 AI 题目分析

- **接口路径**：`/ai/analyze`
- **请求方法**：POST
- **请求参数**：

| 参数名 | 类型 | 必填 | 描述 | 示例值 |
|--------|------|------|------|--------|
| question | String | 是 | 题目内容 | "简述量子力学的基本原理" |
| key | String | 是 | API 访问令牌 | "your_api_token" |
| apiKey | String | 是 | 星火 API 密钥 | "your_spark_api_key" |

- **响应格式**：

成功响应：
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "analysis": "量子力学的基本原理包括...(详细分析)",
    "suggested_answer": "量子力学是描述微观粒子行为的物理学分支...",
    "usage": {
      "prompt_tokens": 25,
      "completion_tokens": 150,
      "total_tokens": 175
    }
  }
}
```

错误响应：
```json
{
  "code": 2001,
  "message": "AI 分析失败: API错误",
  "data": null
}
```

- **使用示例**：

```javascript
// 使用星火 API 分析题目
async function analyzeQuestion(question, apiKey) {
  const response = await fetch("http://api.tikuhai.com/ai/analyze", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      "question": question,
      "key": configStore.queryApis[0].token,
      "apiKey": apiKey
    })
  });
  
  const result = await response.json();
  if (result.code === 0) {
    return result.data.analysis;
  } else {
    throw new Error(result.message);
  }
}
```

#### 2.2 AI 对话接口

- **接口路径**：`/ai/chat`
- **请求方法**：POST
- **请求参数**：

| 参数名 | 类型 | 必填 | 描述 | 示例值 |
|--------|------|------|------|--------|
| messages | Array | 是 | 对话消息数组 | [{"role": "user", "content": "你好"}] |
| key | String | 是 | API 访问令牌 | "your_api_token" |
| apiKey | String | 是 | 星火 API 密钥 | "your_spark_api_key" |
| options | Object | 否 | 模型参数配置 | {"temperature": 0.7} |

- **响应格式**：

成功响应：
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "reply": "你好！我是星火大模型，有什么可以帮助你的？",
    "usage": {
      "prompt_tokens": 10,
      "completion_tokens": 20,
      "total_tokens": 30
    },
    "sid": "session_id_123456"
  }
}
```

错误响应：
```json
{
  "code": 2002,
  "message": "AI 对话失败: 请求超时",
  "data": null
}
```

## 错误代码对照表

| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| 0 | 成功 | - |
| 1001 | 题目不存在 | 检查题目内容或尝试其他题库 |
| 1002 | 批量查询失败 | 减少批量查询数量或检查网络连接 |
| 1003 | 参数错误 | 检查请求参数格式和必填项 |
| 1004 | 认证失败 | 检查 API token 是否正确 |
| 2001 | AI 分析失败 | 检查星火 API 密钥和请求格式 |
| 2002 | AI 对话失败 | 检查网络连接或重试请求 |
| 3001 | 服务器内部错误 | 联系管理员或稍后重试 |

## 版本控制说明

当前 API 版本: v0.2.0

版本更新规则:
- 主版本号 (x.0.0): 不兼容的 API 变更
- 次版本号 (0.x.0): 向后兼容的功能性新增
- 修订版本号 (0.0.x): 向后兼容的问题修正

### 版本历史

- v0.2.0: 集成大模型  API，新增 AI 辅助功能
- v0.1.0: 初始版本，基础题库查询功能

## 星火大模型 X1 API 集成说明

### 集成位置和方式

大模型  API (`spark_x1_final.js`) 作为题库 API 的扩展模块集成，主要用于提供 AI 辅助分析和对话功能。集成方式如下：

1. 将 `spark_x1_final.js` 作为模块导入到主题库 API 系统中
2. 创建新的 API 端点 `/ai/analyze` 和 `/ai/chat` 用于处理 AI 相关请求
3. 使用 `SparkX1ApiHandler` 类处理与星火大模型的通信

### 集成代码示例

```javascript
// 导入星火大模型模块
const { SparkX1ApiHandler, simpleChat } = require('./spark_x1_final.js');

// AI 题目分析处理函数
async function handleAIAnalyze(req, res) {
  const { question, apiKey } = req.body;
  
  try {
    const apiHandler = new SparkX1ApiHandler();
    apiHandler.setAuthToken(apiKey);
    
    const prompt = `请分析并解答以下题目: ${question}`;
    const messages = [{ role: 'user', content: prompt }];
    const result = await apiHandler.chat(messages);
    
    res.json({
      code: 0,
      message: 'success',
      data: {
        analysis: result.reply,
        suggested_answer: extractAnswer(result.reply),
        usage: result.usage
      }
    });
  } catch (error) {
    res.json({
      code: 2001,
      message: `AI 分析失败: ${error.message}`,
      data: null
    });
  }
}

// AI 对话处理函数
async function handleAIChat(req, res) {
  const { messages, apiKey, options } = req.body;
  
  try {
    const apiHandler = new SparkX1ApiHandler();
    apiHandler.setAuthToken(apiKey);
    
    const result = await apiHandler.chat(messages, options);
    
    res.json({
      code: 0,
      message: 'success',
      data: {
        reply: result.reply,
        usage: result.usage,
        sid: result.sid
      }
    });
  } catch (error) {
    res.json({
      code: 2002,
      message: `AI 对话失败: ${error.message}`,
      data: null
    });
  }
}

// 辅助函数：从 AI 回复中提取答案
function extractAnswer(reply) {
  // 简单实现，实际可能需要更复杂的解析逻辑
  const answerMatch = reply.match(/答案[:：]\s*(.+?)(?:\n|$)/);
  return answerMatch ? answerMatch[1].trim() : reply.substring(0, 100) + '...';
}
```

### 接口测试用例

#### 测试用例 1: AI 题目分析

```javascript
// 测试 AI 题目分析接口
async function testAIAnalyze() {
  const testQuestion = "简述牛顿第二定律及其应用";
  const apiKey = "your_spark_api_key";
  
  try {
    const response = await fetch("http://api.tikuhai.com/ai/analyze", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        "question": testQuestion,
        "key": "your_api_token",
        "apiKey": apiKey
      })
    });
    
    const result = await response.json();
    console.log("测试结果:", result);
    
    if (result.code === 0) {
      console.log("测试成功 ✓");
      console.log("分析结果:", result.data.analysis);
    } else {
      console.log("测试失败 ✗");
      console.log("错误信息:", result.message);
    }
  } catch (error) {
    console.error("测试异常:", error);
  }
}
```

#### 测试用例 2: AI 对话接口

```javascript
// 测试 AI 对话接口
async function testAIChat() {
  const testMessages = [
    { role: "user", content: "请帮我解释量子纠缠现象" }
  ];
  const apiKey = "your_spark_api_key";
  
  try {
    const response = await fetch("http://api.tikuhai.com/ai/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        "messages": testMessages,
        "key": "your_api_token",
        "apiKey": apiKey,
        "options": {
          "temperature": 0.8,
          "max_tokens": 1000
        }
      })
    });
    
    const result = await response.json();
    console.log("测试结果:", result);
    
    if (result.code === 0) {
      console.log("测试成功 ✓");
      console.log("AI 回复:", result.data.reply);
      console.log("Token 使用:", result.data.usage);
    } else {
      console.log("测试失败 ✗");
      console.log("错误信息:", result.message);
    }
  } catch (error) {
    console.error("测试异常:", error);
  }
}
```

#### 测试用例 3: 综合测试

```javascript
// 综合测试：先查询题库，如无结果则使用 AI 分析
async function comprehensiveTest() {
  const testQuestion = "什么是区块链技术？它有哪些应用场景？";
  const apiKey = "your_spark_api_key";
  
  try {
    // 先查询题库
    let response = await fetch("http://api.tikuhai.com/search", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        "question": testQuestion,
        "key": "your_api_token"
      })
    });
    
    let result = await response.json();
    
    // 如果题库中没有答案，使用 AI 分析
    if (result.code !== 0) {
      console.log("题库中未找到答案，尝试 AI 分析...");
      
      response = await fetch("http://api.tikuhai.com/ai/analyze", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          "question": testQuestion,
          "key": "your_api_token",
          "apiKey": apiKey
        })
      });
      
      result = await response.json();
    }
    
    console.log("最终结果:", result);
    
    if (result.code === 0) {
      console.log("测试成功 ✓");
      console.log("答案:", result.data.answer || result.data.analysis);
    } else {
      console.log("测试失败 ✗");
      console.log("错误信息:", result.message);
    }
  } catch (error) {
    console.error("测试异常:", error);
  }
}
```
