# 06_ReActAgentHILApiMultiSessionStreamTest

## 项目简介

这是基于05项目创建的支持流式返回的ReAct智能体项目。核心业务逻辑保持不变，主要添加了流式返回功能，让用户能够实时看到AI的回复过程，而不是等待完整回复后才显示。

## 主要特性

### 🆕 新增功能
- **流式返回**: 支持实时显示AI的文本回复，提供更好的用户体验
- **双模式支持**: 用户可以选择流式模式或普通模式
- **实时工具调用提示**: 在流式模式下实时显示工具调用状态
- **Server-Sent Events (SSE)**: 使用标准的SSE协议实现流式传输

### 📋 继承功能
- 多用户多会话支持
- Human-in-the-loop (HIL) 工具审查
- 长期记忆管理
- 会话状态管理
- Redis会话存储
- PostgreSQL数据持久化
- MCP工具集成

## 技术实现

### 后端改动
1. **新增流式API端点**: `/agent/invoke/stream`
2. **StreamChunk数据模型**: 定义流式响应块结构
3. **流式处理函数**: 基于LangGraph的`astream`方法实现
4. **SSE响应**: 使用FastAPI的StreamingResponse

### 前端改动
1. **异步流式客户端**: 使用aiohttp处理流式响应
2. **实时显示**: 逐字符显示AI回复
3. **模式选择**: 用户可选择流式或普通模式
4. **工具调用提示**: 实时显示工具使用状态

## 流式返回模式说明

### 流式块类型
- `text_chunk`: AI生成的文本内容片段（真正的token级别流式）
- `tool_call`: 工具调用通知（非流式，但实时通知）
- `interrupt`: 智能体执行中断
- `completed`: 执行完成
- `error`: 执行错误

### 流式策略
- **AI文本回复**: 使用LangGraph的`messages`流式模式，实现真正的token级别流式输出
- **工具调用**: 不使用流式，但实时通知用户工具使用情况
- **HIL中断**: 保持原有的中断处理机制

### 技术实现
- 使用`stream_mode="messages"`获取LLM的真实token流
- LLM配置启用`streaming=True`以支持token级别流式
- 每个chunk包含AIMessageChunk，其content为单个或多个token
- 避免了假流式（先获取完整内容再模拟流式）的问题

## 使用方法

### 启动服务

1. **启动数据库服务**:
```bash
# PostgreSQL
cd docker/postgresql && docker-compose up -d

# Redis  
cd docker/redis && docker-compose up -d
```

2. **启动后端服务**:
```bash
python 01_backendServer.py
```

3. **启动前端客户端**:
```bash
python 02_frontendServer.py
```

### 使用流式功能

1. 启动前端客户端后，选择模式：
   - `stream`: 流式模式 - 实时显示AI回复
   - `normal`: 普通模式 - 等待完整回复后显示

2. 在流式模式下：
   - AI文本会逐字符实时显示
   - 工具调用会立即显示通知
   - 保持所有原有的HIL功能

## API接口

### 新增接口

#### POST /agent/invoke/stream
流式调用智能体接口

**请求参数**:
```json
{
    "user_id": "string",
    "session_id": "string", 
    "query": "string",
    "system_message": "string"
}
```

**响应格式**: Server-Sent Events
```
data: {"type": "text_chunk", "content": "Hello", "session_id": "...", "timestamp": 1234567890}

data: {"type": "tool_call", "data": {"tool_calls": [...]}, "session_id": "...", "timestamp": 1234567890}

data: {"type": "completed", "data": {...}, "session_id": "...", "timestamp": 1234567890}
```

## 依赖要求

### 新增依赖
- `aiohttp`: 异步HTTP客户端，用于处理流式响应

### 原有依赖
- fastapi
- langgraph  
- langchain
- redis
- psycopg
- rich
- 等等...

## 配置说明

配置文件位置: `utils/config.py`

主要配置项与05项目保持一致：
- 数据库连接
- Redis配置
- LLM配置
- 日志配置

## 项目结构

```
06_ReActAgentHILApiMultiSessionStreamTest/
├── 01_backendServer.py          # 后端服务器（新增流式API）
├── 02_frontendServer.py         # 前端客户端（新增流式处理）
├── utils/
│   ├── config.py               # 配置文件
│   ├── llms.py                 # LLM配置
│   └── tools.py                # 工具配置
├── docker/                     # Docker配置
├── docs/                       # 文档
├── logfile/                    # 日志文件
└── README.md                   # 本文件
```

## 对比05项目的改进

1. **用户体验提升**: 流式返回让用户能够实时看到AI思考过程
2. **响应速度感知**: 即使总时间相同，流式返回让用户感觉响应更快
3. **保持兼容性**: 支持双模式，用户可根据需要选择
4. **技术现代化**: 采用SSE标准，符合现代Web应用趋势

## 注意事项

1. 流式模式需要稳定的网络连接
2. 工具调用部分保持非流式，确保HIL功能正常
3. 错误处理机制保持与原项目一致
4. 会话状态管理逻辑不变

## 作者

@南哥AGI研习社 (B站 or YouTube 搜索"南哥AGI研习社")

## 更新日志

### v1.0 (基于05项目)
- ✅ 新增流式返回功能
- ✅ 支持双模式选择
- ✅ 保持所有原有功能
- ✅ 优化用户体验



