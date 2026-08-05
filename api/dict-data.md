# 字典数据 - 字典数据 CRUD + 建表

## 字典数据列表（分页）

**描述**：分页查询字典数据列表，支持按字典类型编码精确筛选、按字典标签模糊筛选、按状态筛选。

- **Method**：`GET`
- **URL**：`/api/dictData/list`
- **Auth**：需要认证

### 请求参数（Query）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| page | integer | 是 | 1 | 当前页码 |
| pageSize | integer | 是 | 10 | 每页条数 |
| dictType | string | 否 | — | 字典类型编码（精确匹配） |
| dictLabel | string | 否 | — | 字典标签（模糊匹配） |
| status | string | 否 | — | 状态：0-正常 1-停用 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "list": [
      {
        "dictCode": 1,
        "dictSort": 0,
        "dictLabel": "男",
        "dictValue": "1",
        "dictType": "sys_user_sex",
        "listClass": "primary",
        "status": "0",
        "createBy": "admin",
        "createTime": "2025-01-01 12:00:00",
        "updateBy": null,
        "updateTime": null,
        "remark": ""
      },
      {
        "dictCode": 2,
        "dictSort": 1,
        "dictLabel": "女",
        "dictValue": "2",
        "dictType": "sys_user_sex",
        "listClass": "danger",
        "status": "0",
        "createBy": "admin",
        "createTime": "2025-01-01 12:00:00",
        "updateBy": null,
        "updateTime": null,
        "remark": ""
      }
    ],
    "total": 2
  }
}
```

---

## 新增字典数据

**描述**：新增一条字典数据记录。

- **Method**：`POST`
- **URL**：`/api/dictData/create`
- **Auth**：需要认证
- **Content-Type**：`application/json`

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| dictSort | integer | 否 | — | 排序号 |
| dictLabel | string | 是 | — | 字典标签 |
| dictValue | string | 是 | — | 字典值 |
| dictType | string | 是 | — | 字典类型编码 |
| listClass | string | 否 | — | 样式：default / primary / success / info / warning / danger |
| status | string | 否 | — | 状态：0-正常 1-停用 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 编辑字典数据

**描述**：编辑已有字典数据记录，`dictCode` 为必填项，其他字段按需传入（只更新传入的字段）。

- **Method**：`POST`
- **URL**：`/api/dictData/update`
- **Auth**：需要认证
- **Content-Type**：`application/json`

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| dictCode | integer (int64) | 是 | — | 字典编码（主键） |
| dictSort | integer | 否 | — | 排序号 |
| dictLabel | string | 否 | — | 字典标签 |
| dictValue | string | 否 | — | 字典值 |
| dictType | string | 否 | — | 字典类型编码 |
| listClass | string | 否 | — | 样式：default / primary / success / info / warning / danger |
| status | string | 否 | — | 状态：0-正常 1-停用 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 删除字典数据

**描述**：逻辑删除字典数据记录，将 `del_flag` 标记为 `'2'`，并非物理删除。

- **Method**：`POST`
- **URL**：`/api/dictData/delete`
- **Auth**：需要认证
- **Content-Type**：`application/json`

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| dictCode | integer (int64) | 是 | — | 字典编码（主键） |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 创建字典数据表

**描述**：在数据库中创建字典数据表（`sys_dict_data`），使用前需先调用此接口。

- **Method**：`POST`
- **URL**：`/api/dictData/createTable`
- **Auth**：无需认证（白名单）

### 请求参数

无请求参数。

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "字典数据表创建成功"
}
```
