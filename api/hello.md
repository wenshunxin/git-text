# Hello模块 - Hello World 和问候接口

本地开发环境：`http://localhost:8081`

---

## Hello World

**描述**: 返回纯文本 "Hello World"

**Method**: `GET`

**URL**: `/`

**Auth**: 无需认证（白名单）

**请求参数**: 无

**响应示例**:

```
Hello World
```

---

## JSON问候

**描述**: 返回 JSON 格式问候语

**Method**: `GET`

**URL**: `/api/hello`

**Auth**: 需要认证（Bearer Token，格式 `Bearer <token>`，由登录接口返回的 JWT 提供）

**请求参数**: 无

**响应示例**（统一响应体 `R` 结构）：

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "你好 世界！"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求是否成功，`true` 表示成功 |
| rtMsg | string | 提示信息，成功时为空字符串 |
| rtData | string | 响应数据，问候语字符串 |
