# Claude API 中文文档

这是 Anthropic Claude API 的中文文档，包含了完整的 API 参考、使用示例和实现指南。

## 目录

- [API 参考](#api-参考)
- [快速开始](#快速开始)
- [使用示例](#使用示例)
- [实现指南](#实现指南)
- [认证](#认证)
- [最佳实践](#最佳实践)

## API 参考

### 消息 (Messages)

#### 创建消息 (createMessage)

创建并发送消息到 Claude 模型。

```python
import anthropic

client = anthropic.Anthropic(
    api_key="your-api-key",
)

message = client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "Hello, Claude!"}
    ]
)
```

#### 参数说明

- `model`: 要使用的模型名称
- `max_tokens`: 最大输出令牌数
- `messages`: 消息列表，包含角色和内容
- `system`: 系统提示词
- `temperature`: 温度参数 (0-1)
- `top_p`: top-p 采样参数

### 工具使用 (Tools)

Claude 支持使用工具来扩展其功能。

```python
client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    tools=[
        {
            "name": "calculator",
            "description": "A simple calculator",
            "input_schema": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "数学表达式"
                    }
                },
                "required": ["expression"]
            }
        }
    ],
    messages=[
        {"role": "user", "content": "计算 2 + 2"}
    ]
)
```

## 快速开始

### 1. 安装 SDK

```bash
pip install anthropic
```

### 2. 基本使用

```python
import anthropic

# 初始化客户端
client = anthropic.Anthropic(
    api_key="your-api-key",
)

# 发送简单消息
message = client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "你好，请介绍一下自己。"}
    ]
)

print(message.content[0].text)
```

### 3. 流式响应

```python
with client.messages.stream(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "请写一首关于春天的诗"}
    ]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## 使用示例

### 聊天机器人

```python
class ClaudeChatbot:
    def __init__(self, api_key):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.conversation_history = []

    def add_message(self, role, content):
        self.conversation_history.append({"role": role, "content": content})

    def get_response(self, user_input):
        self.add_message("user", user_input)

        response = self.client.messages.create(
            model="claude-3-sonnet-20240229",
            max_tokens=1000,
            messages=self.conversation_history
        )

        ai_response = response.content[0].text
        self.add_message("assistant", ai_response)

        return ai_response
```

### 文档摘要

```python
def summarize_document(document, max_length=500):
    client = anthropic.Anthropic(api_key="your-api-key")

    response = client.messages.create(
        model="claude-3-sonnet-20240229",
        max_tokens=max_length,
        messages=[
            {
                "role": "user",
                "content": f"请总结以下文档，长度不超过 {max_length} 字符：\n\n{document}"
            }
        ]
    )

    return response.content[0].text
```

### 代码生成

```python
def generate_code(prompt, language="python"):
    client = anthropic.Anthropic(api_key="your-api-key")

    response = client.messages.create(
        model="claude-3-sonnet-20240229",
        max_tokens=2000,
        messages=[
            {
                "role": "user",
                "content": f"请用 {language} 编写以下功能的代码：\n\n{prompt}"
            }
        ]
    )

    return response.content[0].text
```

## 实现指南

### 错误处理

```python
from anthropic import APIError, RateLimitError, AuthenticationError

try:
    response = client.messages.create(
        model="claude-3-sonnet-20240229",
        max_tokens=1000,
        messages=[{"role": "user", "content": "Hello"}]
    )
except AuthenticationError:
    print("API 密钥无效")
except RateLimitError:
    print("请求过于频繁，请稍后再试")
except APIError as e:
    print(f"API 错误: {e}")
```

### 重试机制

```python
import time
from anthropic import APIError

def retry_with_backoff(func, max_retries=3, initial_delay=1):
    for attempt in range(max_retries):
        try:
            return func()
        except APIError as e:
            if attempt == max_retries - 1:
                raise
            delay = initial_delay * (2 ** attempt)
            time.sleep(delay)
```

### 缓存策略

```python
import hashlib

def get_cache_key(prompt, model):
    key_string = f"{prompt}_{model}"
    return hashlib.md5(key_string.encode()).hexdigest()

def cached_query(prompt, model, cache_dict, max_age=3600):
    cache_key = get_cache_key(prompt, model)

    if cache_key in cache_dict:
        cached_data, timestamp = cache_dict[cache_key]
        if time.time() - timestamp < max_age:
            return cached_data

    # 执行查询并缓存结果
    response = client.messages.create(
        model=model,
        max_tokens=1000,
        messages=[{"role": "user", "content": prompt}]
    )

    cache_dict[cache_key] = (response.content[0].text, time.time())
    return response.content[0].text
```

## 认证

### API 密钥设置

```bash
# 设置环境变量
export ANTHROPIC_API_KEY="your-api-key"

# 或者在代码中直接设置
client = anthropic.Anthropic(api_key="your-api-key")
```

### 身份验证流程

1. 登录 [Anthropic 控制台](https://console.anthropic.com)
2. 创建新的 API 密钥
3. 安全存储 API 密钥
4. 在应用程序中使用

## 最佳实践

### 1. 合理设置令牌限制

```python
# 根据任务需求设置合适的 max_tokens
response = client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=500,  # 对于简单回答使用较小的值
    messages=[{"role": "user", "content": "你是谁？"}]
)
```

### 2. 使用系统提示词

```python
response = client.messages.create(
    model="claude-3-sonnet-20240229",
    max_tokens=1000,
    system="你是一个专业的助手，回答要简洁明了。",
    messages=[{"role": "user", "content": "解释量子计算"}]
)
```

### 3. 管理对话历史

```python
# 保持适中的对话历史长度
MAX_HISTORY = 10

def add_to_history(history, role, content):
    history.append({"role": role, "content": content})
    if len(history) > MAX_HISTORY:
        history.pop(0)
```

### 4. 监控和日志

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def log_query(prompt, response):
    logger.info(f"Query: {prompt[:100]}...")
    logger.info(f"Response length: {len(response)} characters")
```

## 常见问题

### Q: 如何处理长文本？
A: 使用分块处理，将长文本分成多个部分分别处理。

### Q: 如何提高响应速度？
A: 使用流式响应和缓存机制。

### Q: 如何避免费用超支？
A: 设置预算监控和请求限制。

---

*最后更新：2024年*