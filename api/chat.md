# AI聊天

AI 聊天会话管理 + SSE 流式消息

---

## 创建聊天表

数据库建表接口，用于初始化聊天所需的 tables。该接口为白名单接口，无需 Token 即可调用。

**Method:** POST

**URL:** `/api/chat/createTables`

**Auth:** 无需认证（白名单）

**请求参数：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| - | - | - | 无请求参数 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "聊天表创建成功"
}
```

---

## 会话列表

获取当前用户的聊天会话列表。

**Method:** GET

**URL:** `/api/chat/sessions`

**Auth:** 需要认证

**请求参数：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| - | - | - | 无请求参数 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "ChatSession[]"
}
```

---

## 创建会话

创建一个新的聊天会话。

**Method:** POST

**URL:** `/api/chat/session/create`

**Auth:** 需要认证

**请求参数（Body JSON）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| title | string | 否 | 会话标题 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "ChatSession"
}
```

> 失败时 rtState=false。

---

## 删除会话

删除指定的聊天会话。

**Method:** POST

**URL:** `/api/chat/session/delete`

**Auth:** 需要认证

**请求参数（Body JSON）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| sessionId | integer | 是 | 会话ID |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

> 失败时 rtState=false。

---

## 更新会话标题

修改指定会话的标题。

**Method:** POST

**URL:** `/api/chat/session/updateTitle`

**Auth:** 需要认证

**请求参数（Body JSON）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| sessionId | integer | 是 | 会话ID |
| title | string | 是 | 新标题 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

> 失败时 rtState=false。

---

## 更新会话图标

修改指定会话的图标 URL。

**Method:** POST

**URL:** `/api/chat/session/updateIcon`

**Auth:** 需要认证

**请求参数（Body JSON）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| sessionId | integer | 是 | 会话ID |
| iconUrl | string | 是 | 图标 URL |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 消息列表

获取指定会话的聊天消息列表。

**Method:** GET

**URL:** `/api/chat/messages`

**Auth:** 需要认证

**请求参数（Query）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| sessionId | integer | 是 | 会话ID |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "ChatMessage[]"
}
```

---

## 发送消息（SSE 流式）

向 AI 发送消息并流式返回回复。采用 Server-Sent Events 协议，响应 `Content-Type: text/event-stream`。

**事件类型说明：**

| 事件类型 | 说明 |
|----------|------|
| start | 流式传输开始，包含 sessionId |
| chunk | 流式内容片段，包含 content 字段 |
| done | 流式传输完成，包含 sessionId |
| error | 发生错误 |

**Method:** POST

**URL:** `/api/chat/send`

**Auth:** 需要认证

**Response Content-Type:** `text/event-stream`

**请求参数（Body JSON）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| sessionId | integer | 是 | 会话ID |
| message | string | 是 | 用户消息（不能为空） |

**响应示例（SSE 事件流）：**

```json
data: {"type":"start","sessionId":1}

data: {"type":"chunk","content":"你好..."}

data: {"type":"done","sessionId":1}
```

---

**认证说明：** 除"创建聊天表"为白名单接口外，其余接口均需在请求头中携带 `Authorization: Bearer <token>`，token 由登录接口 `/api/auth/login` 返回。
