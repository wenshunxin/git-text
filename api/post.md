# 岗位管理 - 岗位 CRUD + 建表

## 岗位列表（分页）

**描述：** 分页查询岗位列表，支持按岗位编码、岗位名称模糊搜索及状态筛选。

- **Method：** GET
- **URL：** /api/post/list
- **Auth：** 需要认证
- **Content-Type：** application/x-www-form-urlencoded

**请求参数（Query）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | integer | 是 | 页码，默认值：1 |
| pageSize | integer | 是 | 每页条数，默认值：10 |
| postCode | string | 否 | 岗位编码（模糊匹配） |
| postName | string | 否 | 岗位名称（模糊匹配） |
| status | string | 否 | 状态：0-正常 1-停用 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "list": [
      {
        "postId": 1,
        "postCode": "ceo",
        "postName": "董事长",
        "postSort": 1,
        "status": "0"
      }
    ],
    "total": 5
  }
}
```

---

## 新增岗位

**描述：** 新增一条岗位记录，`postCode` 和 `postName` 为必填项。

- **Method：** POST
- **URL：** /api/post/create
- **Auth：** 需要认证
- **Content-Type：** application/json

**请求参数（Body）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| postCode | string | 是 | 岗位编码 |
| postName | string | 是 | 岗位名称 |
| postSort | integer | 否 | 排序号 |
| status | string | 否 | 状态：0-正常 1-停用 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 编辑岗位

**描述：** 根据 `postId` 编辑指定岗位的信息。

- **Method：** POST
- **URL：** /api/post/update
- **Auth：** 需要认证
- **Content-Type：** application/json

**请求参数（Body）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| postId | integer (int64) | 是 | 岗位ID |
| postCode | string | 否 | 岗位编码 |
| postName | string | 否 | 岗位名称 |
| postSort | integer | 否 | 排序号 |
| status | string | 否 | 状态：0-正常 1-停用 |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 删除岗位

**描述：** 根据 `postId` 物理删除指定岗位（sys_post 表无 del_flag 字段，执行真实删除）。

- **Method：** POST
- **URL：** /api/post/delete
- **Auth：** 需要认证
- **Content-Type：** application/json

**请求参数（Body）：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| postId | integer (int64) | 是 | 要删除的岗位ID |

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 创建岗位表

**描述：** 执行建表 SQL，创建 sys_post 岗位表（白名单接口，无需 Token 即可调用）。

- **Method：** POST
- **URL：** /api/post/createTable
- **Auth：** 无需认证（白名单）

**请求参数：** 无

**响应示例：**

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "岗位表创建成功"
}
```
