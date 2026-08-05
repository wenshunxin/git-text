# 用户管理

用户 CRUD：分页查询、新增、编辑、逻辑删除

---

## 用户列表（分页）

分页查询用户列表，支持姓名/性别/账号模糊筛选。

**Method:** `GET`

**URL:** `/api/user/list`

**Auth:** 需要认证

**Content-Type:** `application/x-www-form-urlencoded`

**请求参数（Query）:**

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| page | integer | 是 | 1 | 页码 |
| pageSize | integer | 是 | 3 | 每页条数 |
| userName | string | 否 | - | 用户姓名（模糊） |
| sex | string | 否 | - | 性别（0-男 1-女 2-未知） |
| loginName | string | 否 | - | 登录账号（模糊） |

**响应示例（成功）:**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "list": [
      {
        "userId": 1,
        "loginName": "admin",
        "userName": "管理员",
        "sex": "0",
        "avatar": "/upload/avatar.jpg",
        "status": "0",
        "delFlag": "0"
      }
    ],
    "total": 10
  }
}
```

**响应字段说明:**

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求状态，true 表示成功 |
| rtMsg | string | 错误信息，成功时为空字符串 |
| rtData.list | array | 用户列表 |
| rtData.list[].userId | integer | 用户ID |
| rtData.list[].loginName | string | 登录账号 |
| rtData.list[].userName | string | 用户姓名 |
| rtData.list[].sex | string | 性别：0-男 1-女 2-未知 |
| rtData.list[].avatar | string | 头像路径 |
| rtData.list[].status | string | 状态：0-正常 1-停用 |
| rtData.list[].delFlag | string | 删除标记：0-存在 2-删除 |
| rtData.total | integer | 总记录数 |

**错误码说明:**

| 错误信息 | 说明 |
|----------|------|
| rtState: false, rtMsg: "token 无效或已过期" | 未登录或 token 过期 |

---

## 新增用户

新增用户，默认密码 123456。

**Method:** `POST`

**URL:** `/api/user/create`

**Auth:** 需要认证

**Content-Type:** `application/json`

**请求参数（Body）:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| loginName | string | 是 | 登录账号 |
| userName | string | 是 | 用户姓名 |
| sex | string | 否 | 性别：0-男 1-女 2-未知 |
| status | string | 否 | 状态：0-正常 1-停用 |

**请求示例:**

```json
{
  "loginName": "zhangsan",
  "userName": "张三",
  "sex": "0",
  "status": "0"
}
```

**响应示例（成功）:**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

**响应示例（失败）:**

```json
{
  "rtState": false,
  "rtMsg": "该账号已被注册",
  "rtData": null
}
```

**错误码说明:**

| 错误信息 | 说明 |
|----------|------|
| "该账号已被注册" | loginName 重复 |
| "用户名或账号不能为空" | loginName 或 userName 未填写 |

---

## 编辑用户

编辑用户姓名/性别/头像。

**Method:** `POST`

**URL:** `/api/user/update`

**Auth:** 需要认证

**Content-Type:** `application/json`

**请求参数（Body）:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | integer | 是 | 用户ID |
| userName | string | 否 | 用户姓名 |
| sex | string | 否 | 性别：0-男 1-女 2-未知 |
| avatar | string | 否 | 头像路径 |

**请求示例:**

```json
{
  "userId": 1,
  "userName": "管理员",
  "sex": "0",
  "avatar": "/upload/avatar.jpg"
}
```

**响应示例（成功）:**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

**响应示例（失败）:**

```json
{
  "rtState": false,
  "rtMsg": "用户不存在",
  "rtData": null
}
```

**错误码说明:**

| 错误信息 | 说明 |
|----------|------|
| "用户ID不能为空" | userId 未填写 |
| "用户不存在" | userId 对应的用户不存在或已删除 |

---

## 删除用户

逻辑删除用户（delFlag = '2'）。

**Method:** `POST`

**URL:** `/api/user/delete`

**Auth:** 需要认证

**Content-Type:** `application/json`

**请求参数（Body）:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userId | integer | 是 | 用户ID |

**请求示例:**

```json
{
  "userId": 1
}
```

**响应示例（成功）:**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

**响应示例（失败）:**

```json
{
  "rtState": false,
  "rtMsg": "用户不存在",
  "rtData": null
}
```

**错误码说明:**

| 错误信息 | 说明 |
|----------|------|
| "用户ID不能为空" | userId 未填写 |
| "用户不存在" | userId 对应的用户不存在或已删除 |
