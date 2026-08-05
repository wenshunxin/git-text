# 认证模块

用户登录认证与头像更新

---

## 用户登录

校验账号密码，成功返回 JWT Token 及用户信息。

**Method:** `POST`

**URL:** `/api/auth/login`

**Auth:** 无需认证（白名单）

**请求参数 (Body - JSON):**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| loginName | string | 是 | 登录账号 |
| password | string | 是 | 密码（明文，服务端按 RuoYi MD5 算法校验） |

**响应示例 (200):**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "token": "eyJhbG...",
    "loginName": "admin",
    "userName": "管理员",
    "avatar": "/upload/avatar/2025/xxx.jpg"
  }
}
```

**说明：** 失败时 `rtState` 为 `false`，`rtMsg` 包含错误信息。

---

## 更新头像

更新当前登录用户的头像 URL。

**Method:** `POST`

**URL:** `/api/auth/updateAvatar`

**Auth:** 需要认证（Bearer JWT，格式: `Authorization: Bearer <token>`）

**请求参数 (Body - JSON):**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| avatar | string | 是 | 头像 URL 路径 |

**响应示例 (200):**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "avatar": "/upload/avatar/2025/xxx.jpg"
  }
}
```

**说明：** 失败时 `rtState` 为 `false`，`rtMsg` 包含错误信息。
