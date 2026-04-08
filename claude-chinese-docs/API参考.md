# Claude API 参考

## 概述

Claude API 提供了强大的自然语言处理能力，支持消息交互、工具使用、多模态输入等功能。

## 核心端点

### messages

#### POST /v1/messages

创建并发送消息到 Claude 模型。

#### 请求参数

```json
{
  "model": "claude-3-sonnet-20240229",
  "max_tokens": 1000,
  "messages": [
    {
      "role": "user",
      "content": "Hello, Claude!"
    }
  ],
  "system": "You are a helpful assistant.",
  "temperature": 0.7,
  "top_p": 1,
  "stream": false
}
```

#### 参数说明

| 参数 | 类型 | 必需 | 描述 |
|------|------|------|------|
| model | string | 是 | 要使用的模型名称 |
| max_tokens | integer | 是 | 最大输出令牌数 |
| messages | array | 是 | 消息列表 |
| system | string | 否 | 系统提示词 |
| temperature | float | 否 | 温度参数 (0-1) |
| top_p | float | 否 | top-p 采样参数 |
| stream | boolean | 否 | 是否流式响应 |
| tools | array | 否 | 可用工具列表 |
| tool_choice | object | 否 | 工具选择策略 |

#### 消息格式

```json
{
  "role": "user",  // user, assistant, 或 system
  "content": "消息内容"
}
```

#### 响应格式

```json
{
  "id": "msg_123",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I help you today?"
    }
  ],
  "model": "claude-3-sonnet-20240229",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 10,
    "output_tokens": 20,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  }
}
```

## 模型列表

### Claude 3 系列

#### Claude 3 Opus
- **ID**: claude-3-opus-20240229
- **能力**: 最强大的模型，适合复杂任务
- **特点**: 最高性能，支持长上下文

#### Claude 3 Sonnet
- **ID**: claude-3-sonnet-20240229
- **能力**: 平衡性能和成本
- **特点**: 速度较快，性价比高

#### Claude 3 Haiku
- **ID**: claude-3-haiku-20240229
- **能力**: 最快的模型
- **特点**: 响应速度极快，适合简单任务

## 工具使用

### 工具定义

```json
{
  "name": "calculator",
  "description": "A simple calculator for mathematical operations",
  "input_schema": {
    "type": "object",
    "properties": {
      "expression": {
        "type": "string",
        "description": "数学表达式，如 '2 + 2'"
      }
    },
    "required": ["expression"]
  }
}
```

### 工具调用

当模型需要使用工具时，响应格式如下：

```json
{
  "id": "msg_123",
  "type": "message",
  "role": "assistant",
  "content": [],
  "model": "claude-3-sonnet-20240229",
  "stop_reason": "tool_use",
  "usage": {
    "input_tokens": 15,
    "output_tokens": 25,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  },
  "tool_calls": [
    {
      "id": "tool_abc123",
      "type": "function",
      "function": {
        "name": "calculator",
        "arguments": "{\"expression\": \"2 + 2\"}"
      }
    }
  ]
}
```

### 工具结果

```json
{
  "tool_results": [
    {
      "call_id": "tool_abc123",
      "type": "tool_result",
      "content": [
        {
          "type": "text",
          "text": "4"
        }
      ]
    }
  ]
}
```

## 流式响应

### 流式请求

```json
{
  "model": "claude-3-sonnet-20240229",
  "max_tokens": 1000,
  "messages": [
    {"role": "user", "content": "Hello"}
  ],
  "stream": true
}
```

### 流式响应格式

```json
{
  "type": "message_delta",
  "delta": {
    "type": "text",
    "text": "Hello"
  }
}
```

### 完整流式响应

```json
{
  "type": "message_stop",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 10,
    "output_tokens": 50,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  }
}
```

## 错误处理

### 错误类型

#### AuthenticationError
```python
{
  "type": "error",
  "error": {
    "type": "authentication_error",
    "message": "Invalid API key"
  }
}
```

#### RateLimitError
```python
{
  "type": "error",
  "error": {
    "type": "rate_limit_error",
    "message": "Rate limit exceeded"
  }
}
```

#### APIError
```python
{
  "type": "error",
  "error": {
    "type": "api_error",
    "message": "An unexpected error occurred"
  }
}
```

### 错误代码

| 代码 | 描述 |
|------|------|
| 400 | 请求参数错误 |
| 401 | 认证失败 |
| 403 | 权限不足 |
| 429 | 请求过于频繁 |
| 500 | 服务器内部错误 |
| 529 | 过载错误 |

## 限制

### 令牌限制
- 输入令牌数：根据模型不同而异
  - Claude 3 Opus：200K tokens
  - Claude 3 Sonnet：200K tokens
  - Claude 3 Haiku：200K tokens
- 输出令牌数：默认最大 4096 tokens
- 可通过 `max_tokens` 参数调整

### 请求限制
- 每分钟请求数：根据订阅级别而异
- 并发请求数：根据订阅级别而异
- 文件大小：最大 10MB

### 成本
- 不同模型有不同的定价
- 按输入和输出令牌数量计费
- 支持批量请求折扣

## 最佳实践

### 1. 合理设置令牌数

```python
# 对于简单问答
response = client.messages.create(
    model="claude-3-haiku-20240229",
    max_tokens=100,
    messages=[{"role": "user", "content": "你是谁？"}]
)

# 对于复杂任务
response = client.messages.create(
    model="claude-3-opus-20240229",
    max_tokens=2000,
    messages=[{"role": "user", "content": complex_prompt}]
)
```

### 2. 使用系统提示词

```python
# 设置角色和行为
response = client.messages.create(
    model="claude-3-sonnet-20240229",
    system="你是一个专业的编程助手，回答要准确、简洁。",
    messages=[{"role": "user", "content": "解释 Python 装饰器"}]
)
```

### 3. 管理上下文

```python
# 保持适中的对话历史
conversation_history = [
    {"role": "system", "content": "你是一个翻译助手"},
    {"role": "user", "content": "Hello"},
    {"role": "assistant", "content": "你好！"}
]

# 添加新消息并限制长度
conversation_history.append({"role": "user", "content": "How are you?"})
if len(conversation_history) > 20:  # 限制历史长度
    conversation_history.pop(1)  # 移除最早的系统消息
```

### 4. 错误处理和重试

```python
import time
from anthropic import APIError

def safe_api_call(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except APIError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt
            time.sleep(wait_time)
```

## 示例代码

### Python SDK

```python
import anthropic

client = anthropic.Anthropic(api_key="your-api-key")

# 基本请求
response = client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "Hello Claude!"}
    ]
)

# 流式请求
with client.messages.stream(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    messages=[{"role": "user", "content": "Write a poem"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

# 使用工具
response = client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    tools=[
        {
            "name": "get_weather",
            "description": "Get current weather",
            "input_schema": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市名称"
                    }
                },
                "required": ["location"]
            }
        }
    ],
    messages=[
        {"role": "user", "content": "What's the weather in Beijing?"}
    ]
)
```

### JavaScript SDK

```javascript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: 'your-api-key',
});

// 基本请求
const response = await anthropic.messages.create({
  model: 'claude-3-sonnet-20240229',
  max_tokens: 1000,
  messages: [
    { role: 'user', content: 'Hello Claude!' }
  ]
});

// 流式请求
const stream = await anthropic.messages.create({
  model: 'claude-3-sonnet-20240229',
  max_tokens: 1000,
  messages: [{ role: 'user', content: 'Write a poem' }],
  stream: true
});

for await (const chunk of stream) {
  console.log(chunk.delta?.text);
}
```

---

*本文档基于 Claude API 官方文档整理，建议参考最新文档。*