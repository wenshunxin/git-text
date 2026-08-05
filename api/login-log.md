# 登录日志

## 登录日志列表（分页）

**描述**: 分页查询系统登录日志记录，支持按登录账号和时间范围筛选。

**Method**: `GET`

**URL**: `/api/loginLog/list`

**Auth**: 需要认证（请求头携带 `Authorization: Bearer <token>`，token 由 `/api/auth/login` 接口返回）

**Content-Type**: `application/x-www-form-urlencoded`

### 请求参数（Query）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| pageNum | integer | 是 | 1 | 当前页码，从 1 开始 |
| pageSize | integer | 是 | 10 | 每页记录数 |
| loginName | string | 否 | - | 登录账号，支持模糊匹配 |
| startTime | string | 否 | - | 开始时间，格式: yyyy-MM-dd HH:mm:ss |
| endTime | string | 否 | - | 结束时间，格式: yyyy-MM-dd HH:mm:ss |

### 请求示例

```
GET /api/loginLog/list?pageNum=1&pageSize=10&loginName=admin&startTime=2025-01-01 00:00:00&endTime=2025-12-31 23:59:59
```

### 响应示例

#### 成功响应

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "total": 100,
    "list": [
      {
        "infoId": 1024,
        "loginName": "admin",
        "ipaddr": "127.0.0.1",
        "loginLocation": "内网IP",
        "browser": "Chrome 12",
        "os": "Mac OS X",
        "status": "0",
        "msg": "登录成功",
        "loginTime": "2025-08-05 10:30:00"
      },
      {
        "infoId": 1023,
        "loginName": "zhangsan",
        "ipaddr": "192.168.1.100",
        "loginLocation": "局域网",
        "browser": "Firefox 11",
        "os": "Windows 10",
        "status": "1",
        "msg": "密码错误",
        "loginTime": "2025-08-05 10:25:00"
      }
    ]
  }
}
```

#### 失败响应（Token 缺失或无效）

```json
{
  "rtState": false,
  "rtMsg": "请先登录",
  "rtData": null
}
```

### 响应字段说明

**顶层 R 结构**

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求处理结果，true=成功，false=失败 |
| rtMsg | string | 提示信息，成功时为空字符串，失败时包含错误描述 |
| rtData | object | 业务数据，包含分页信息和记录列表 |

**rtData 分页结构**

| 字段 | 类型 | 说明 |
|------|------|------|
| total | integer | 符合筛选条件的总记录数 |
| list | array[LoginLog] | 当前页登录日志记录列表 |

**LoginLog 对象**

| 字段 | 类型 | 说明 |
|------|------|------|
| infoId | long | 访问记录 ID |
| loginName | string | 登录账号 |
| ipaddr | string | 登录 IP 地址 |
| loginLocation | string | 登录地点 |
| browser | string | 浏览器类型及版本 |
| os | string | 操作系统 |
| status | string | 登录状态，"0"=成功，"1"=失败 |
| msg | string | 提示消息（成功信息或失败原因） |
| loginTime | string | 访问时间，格式: yyyy-MM-dd HH:mm:ss |

---

### 错误码说明

| 错误信息 | 原因 |
|----------|------|
| 请先登录 | 未携带 Authorization 请求头，或 Token 已过期/无效 |
| rtState=false | 服务端处理异常，具体原因见 rtMsg 字段 |
