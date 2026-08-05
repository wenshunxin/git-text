# 字典类型

字典类型 CRUD + 建表

---

## 字典类型列表（分页）

**描述：** 分页查询字典类型列表，支持按字典名称、字典类型编码模糊搜索，按状态筛选。

- **Method：** GET
- **URL：** /api/dictType/list
- **Auth：** 需要认证
- **Content-Type：** application/x-www-form-urlencoded

### 请求参数（Query）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| page | integer | 是 | 1 | 页码 |
| pageSize | integer | 是 | 10 | 每页条数 |
| dictName | string | 否 | - | 字典名称（模糊） |
| dictType | string | 否 | - | 字典类型编码（模糊） |
| status | string | 否 | - | 状态：0-正常 1-停用 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "list": [
      {
        "dictId": 1,
        "dictName": "用户性别",
        "dictType": "sys_user_sex",
        "status": "0",
        "remark": "用户性别列表",
        "createBy": "admin",
        "createTime": "2024-01-01 00:00:00",
        "updateBy": null,
        "updateTime": null
      }
    ],
    "total": 5
  }
}
```

---

## 新增字典类型

**描述：** 新增字典类型，先校验 dictType 唯一性。dictName 和 dictType 为必填。

- **Method：** POST
- **URL：** /api/dictType/create
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| dictName | string | 是 | - | 字典名称 |
| dictType | string | 是 | - | 字典类型编码（唯一） |
| status | string | 否 | - | 0-正常 1-停用 |
| remark | string | 否 | - | 备注 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

### 错误示例

```json
{
  "rtState": false,
  "rtMsg": "字典类型编码已存在",
  "rtData": null
}
```

---

## 编辑字典类型

**描述：** 编辑字典类型，先校验唯一性，若 dictType 变更则同步更新 dict_data 表中的对应记录。

- **Method：** POST
- **URL：** /api/dictType/update
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| dictId | integer | 是 | - | 字典ID |
| dictName | string | 否 | - | 字典名称 |
| dictType | string | 否 | - | 字典类型编码 |
| status | string | 否 | - | 状态 |
| remark | string | 否 | - | 备注 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 删除字典类型

**描述：** 逻辑删除字典类型（del_flag 置为 2），同时级联逻辑删除关联的 dict_data 记录。

- **Method：** POST
- **URL：** /api/dictType/delete
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| dictId | integer | 是 | - | 字典ID |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 创建字典类型表

**描述：** 首次部署建表（幂等，表已存在则跳过）。

- **Method：** POST
- **URL：** /api/dictType/createTable
- **Auth：** 无需认证（白名单）

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "字典类型表创建成功"
}
```
