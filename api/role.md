# 角色管理

## 角色列表（分页）

**描述**：分页查询角色列表，支持按角色名称、角色权限字符串筛选。

**Method**：`GET`

**URL**：`/api/role/list`

**Auth**：需要认证

**Content-Type**：`application/x-www-form-urlencoded`

### 请求参数（Query）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| page | integer | 是 | 1 | 当前页码 |
| pageSize | integer | 是 | 10 | 每页条数 |
| roleKey | string | 否 | - | 角色权限字符串 |
| roleName | string | 否 | - | 角色名称 |

### 响应示例（200）

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "list": [
      {
        "roleId": 1,
        "roleName": "超级管理员",
        "roleKey": "admin",
        "roleSort": 1,
        "dataScope": "1",
        "status": "0",
        "delFlag": "0",
        "createTime": "2024-01-01 00:00:00"
      }
    ],
    "total": 3
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求状态，true 表示成功 |
| rtMsg | string | 消息，成功时为空字符串 |
| rtData.list | array | 角色列表 |
| rtData.list[].roleId | integer | 角色 ID |
| rtData.list[].roleName | string | 角色名称 |
| rtData.list[].roleKey | string | 角色权限字符串 |
| rtData.list[].roleSort | integer | 排序号 |
| rtData.list[].dataScope | string | 数据范围：1-全部 2-自定义 |
| rtData.list[].status | string | 状态：0-正常 1-停用 |
| rtData.list[].delFlag | string | 删除标记：0-存在 2-删除 |
| rtData.list[].createTime | string | 创建时间 |
| rtData.total | integer | 总记录数 |

---

## 新增角色

**描述**：新增一个角色，角色名称和角色权限字符串为必填项。

**Method**：`POST`

**URL**：`/api/role/create`

**Auth**：需要认证

**Content-Type**：`application/json`

### 请求参数（Body - JSON）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| roleName | string | 是 | - | 角色名称 |
| roleKey | string | 是 | - | 角色权限字符串 |
| roleSort | integer | 否 | - | 排序号 |
| dataScope | string | 否 | - | 数据范围：1-全部 2-自定义 |
| status | string | 否 | - | 状态：0-正常 1-停用 |

### 请求体示例

```json
{
  "roleName": "普通用户",
  "roleKey": "common",
  "roleSort": 2,
  "dataScope": "2",
  "status": "0"
}
```

### 响应示例（200）

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求状态，true 表示成功 |
| rtMsg | string | 消息，成功时为空字符串；失败时包含错误信息（如"角色名称已存在"） |
| rtData | object\|null | 数据，新增成功时为 null |

---

## 编辑角色

**描述**：编辑已有角色信息，角色 ID 为必填项，其他字段可选（仅更新传入的字段）。

**Method**：`POST`

**URL**：`/api/role/update`

**Auth**：需要认证

**Content-Type**：`application/json`

### 请求参数（Body - JSON）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| roleId | integer | 是 | - | 角色 ID |
| roleName | string | 否 | - | 角色名称 |
| roleKey | string | 否 | - | 角色权限字符串 |
| roleSort | integer | 否 | - | 排序号 |
| dataScope | string | 否 | - | 数据范围：1-全部 2-自定义 |
| status | string | 否 | - | 状态：0-正常 1-停用 |

### 请求体示例

```json
{
  "roleId": 2,
  "roleName": "普通用户（已更名）",
  "status": "0"
}
```

### 响应示例（200）

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求状态，true 表示成功 |
| rtMsg | string | 消息，成功时为空字符串；失败时包含错误信息（如"角色不存在"） |
| rtData | object\|null | 数据，更新成功时为 null |

---

## 删除角色

**描述**：逻辑删除角色（将 delFlag 置为 '2'），角色 ID 为必填项。

**Method**：`POST`

**URL**：`/api/role/delete`

**Auth**：需要认证

**Content-Type**：`application/json`

### 请求参数（Body - JSON）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| roleId | integer | 是 | - | 角色 ID |

### 请求体示例

```json
{
  "roleId": 3
}
```

### 响应示例（200）

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| rtState | boolean | 请求状态，true 表示成功 |
| rtMsg | string | 消息，成功时为空字符串；失败时包含错误信息（如"角色不存在"、"该角色下存在用户，无法删除"） |
| rtData | object\|null | 数据，删除成功时为 null |
