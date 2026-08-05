# 仪表盘

## 仪表盘统计

**描述**: 用户/订单数、访问趋势、技术栈等

**Method**: GET

**URL**: /api/dashboard/stats

**Auth**: 需要认证（Bearer JWT）

**请求头**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| Authorization | string | 是 | Bearer <token>，登录接口返回的 JWT token |

**请求参数**: 无

**响应示例**:

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "userCount": 15,
    "orderCount": 25,
    "visitTrend": "[{\"date\": \"2026-07-29\", \"value\": 62345}, {\"date\": \"2026-07-30\", \"value\": 58920}]",
    "dailyAvgVisit": 57044,
    "authTrend": "[{\"hour\": 0, \"value\": 120}, {\"hour\": 1, \"value\": 85}]",
    "authCount": 9970,
    "techStack": "[{\"name\": \"Spring Boot\", \"version\": \"1.2.3\", \"upgrade\": false}, {\"name\": \"Vue\", \"version\": \"3.x\", \"upgrade\": true}]",
    "openProjects": "[{\"name\": \"RuoYi\", \"desc\": \"若依管理系统\", \"color\": \"#1890ff\"}]",
    "quickActions": "[{\"label\": \"新建用户\", \"key\": \"createUser\", \"color\": \"#52c41a\", \"badge\": 0}]"
  }
}
```

**rtData 字段说明**:

| 字段名 | 类型 | 说明 |
|--------|------|------|
| userCount | number | 用户总数 |
| orderCount | number | 订单总数 |
| visitTrend | array | 访问趋势，元素含 date（日期）和 value（访问量） |
| dailyAvgVisit | number | 日均访问量 |
| authTrend | array | 认证趋势，元素含 hour（小时）和 value（认证次数） |
| authCount | number | 认证总次数 |
| techStack | array | 技术栈列表，元素含 name（名称）、version（版本）、upgrade（是否有升级） |
| openProjects | array | 开源项目列表，元素含 name（名称）、desc（描述）、color（颜色） |
| quickActions | array | 快捷操作列表，元素含 label（标签）、key（键名）、color（颜色）、badge（徽标数） |

---
