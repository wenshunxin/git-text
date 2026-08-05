# 字典管理模块接口契约

> 后端：Spring Boot 1.2.3 + Java 8 + JdbcTemplate
> 前端：Vue 3 + TypeScript + Ant Design Vue 4
> 所属模块：字典管理（`sys_dict_type` + `sys_dict_data`）
> 关联文档：[数据库表结构](../database/sys-dict.md) · [设计总览](../tasks/dict-module/design.md)
> 状态：**设计稿**

本文档定义字典管理模块 10 个接口的契约。本模块**不修改**全局 [api-contract.md](./api-contract.md)，独立成文。

---

## 一、通用约定

| 约定 | 说明 |
|------|------|
| API 前缀 | Controller 标注 `@ApiPrefix`，自动加 `/api` 前缀 |
| 统一响应体 | `R`（`rtData`/`rtState`/`rtMsg`），成功 `R.ok(data)`，业务失败 `R.fail(msg)` |
| HTTP 方法 | 全部 POST；`list` 走 Query（`@RequestParam`），`create`/`update`/`delete` 走 Body（`@RequestBody`），`createTable` 无参 |
| 认证 | 除 `createTable` 外均需 `Authorization: Bearer <token>`，由 `TokenFilter` 校验 |
| 分页 | 入参 `page`（从 1 开始，默认 1）/ `pageSize`（默认 10），返回 `{list, total}`，SQL `LIMIT ? OFFSET ?`，`offset=(page-1)*pageSize` |
| 命名 | 数据库蛇形 `dict_type` ↔ Java/TS 驼峰 `dictType` |
| 时间字段 | Java 实体为 `String`，写入用 `NOW()` |
| 路径风格 | 驼峰双词：`/api/dictType/**`、`/api/dictData/**`（与 `/api/loginLog` 一致） |

---

## 二、字典类型接口（`/api/dictType`）

### 2.1 `POST /api/dictType/list` 分页查询

**请求（Query）：**

```
page=1&pageSize=10&dictName=性别&dictType=sys_user_sex&status=0
```

- `dictName`、`dictType` 模糊匹配：`LIKE CONCAT('%',?,'%')`（实现用 `"%" + param + "%"`）。
- `status` 精确匹配：`AND status = ?`（仅 `0`/`1`）。
- 所有筛选参数可选，为空不拼条件。

**响应：**

```json
{
  "rtData": {
    "list": [
      {
        "dictId": 1,
        "dictName": "用户性别",
        "dictType": "sys_user_sex",
        "status": "0",
        "delFlag": "0",
        "createBy": "admin",
        "createTime": "2026-08-05 10:00:00",
        "updateBy": "admin",
        "updateTime": "2026-08-05 10:00:00",
        "remark": "用户性别字典"
      }
    ],
    "total": 1
  },
  "rtState": true,
  "rtMsg": ""
}
```

**SQL：**

```sql
SELECT dict_id, dict_name, dict_type, status, del_flag,
       create_by, create_time, update_by, update_time, remark
FROM sys_dict_type
WHERE del_flag = '0'
  [AND dict_name LIKE ?]
  [AND dict_type LIKE ?]
  [AND status = ?]
ORDER BY dict_id DESC
LIMIT ? OFFSET ?
```

### 2.2 `POST /api/dictType/create` 新增

**请求体：**

```json
{
  "dictName": "用户性别",
  "dictType": "sys_user_sex",
  "status": "0",
  "remark": "用户性别字典",
  "createBy": "admin",
  "updateBy": "admin"
}
```

- **唯一性校验**：先查 `WHERE del_flag='0' AND dict_type=?`，>0 返回 `R.fail("字典类型已存在")`。

**SQL：**

```sql
INSERT INTO sys_dict_type (dict_name, dict_type, status, del_flag,
                           create_by, create_time, update_by, update_time, remark)
VALUES (?, ?, ?, '0', ?, NOW(), ?, NOW(), ?)
```

### 2.3 `POST /api/dictType/update` 编辑

**请求体（`dictId` 必填）：**

```json
{
  "dictId": 1,
  "dictName": "用户性别",
  "dictType": "sys_user_sex",
  "status": "0",
  "remark": "已更新",
  "updateBy": "admin"
}
```

- **唯一性校验**：查 `WHERE del_flag='0' AND dict_type=? AND dict_id<>?`，>0 返回 `R.fail("字典类型已存在")`。
- 若 `dictType` 变更，需同步更新 `sys_dict_data.dict_type`（保证关联一致，见 [sys-dict.md 4.4 节](../database/sys-dict.md)）。

**SQL：**

```sql
UPDATE sys_dict_type
SET dict_name=?, dict_type=?, status=?, update_by=?, update_time=NOW(), remark=?
WHERE dict_id=?
```

> 若 `dictType` 变更，额外执行：`UPDATE sys_dict_data SET dict_type=?, update_time=NOW() WHERE dict_type=? AND del_flag='0'`。

### 2.4 `POST /api/dictType/delete` 逻辑删除（含级联）

**请求体：**

```json
{ "dictId": 1 }
```

- **级联逻辑删除**：先删数据，再删类型（见 [sys-dict.md 4.2 节](../database/sys-dict.md)）。

**SQL：**

```sql
-- 1) 查出 dict_type
SELECT dict_type FROM sys_dict_type WHERE dict_id=?;
-- 2) 级联逻辑删除数据
UPDATE sys_dict_data SET del_flag='2', update_time=NOW() WHERE dict_type=?;
-- 3) 逻辑删除类型
UPDATE sys_dict_type SET del_flag='2', update_time=NOW() WHERE dict_id=?;
```

### 2.5 `POST /api/dictType/createTable` 建表

无参，执行 [sys-dict.md 2.1 节](../database/sys-dict.md) DDL，返回 `R.ok("字典类型表创建成功")`。**无需 token**，须加入 `TokenFilter` 白名单。

---

## 三、字典数据接口（`/api/dictData`）

### 3.1 `POST /api/dictData/list` 分页查询

**请求（Query）：**

```
page=1&pageSize=10&dictType=sys_user_sex&dictLabel=男&status=0
```

- `dictType` 精确匹配（按类型查数据，是数据页的核心筛选条件）。
- `dictLabel` 模糊匹配：`LIKE`。
- `status` 精确匹配。
- `dictType` 为空时查全量（管理场景）。

**响应：**

```json
{
  "rtData": {
    "list": [
      {
        "dictCode": 1,
        "dictSort": 1,
        "dictLabel": "男",
        "dictValue": "0",
        "dictType": "sys_user_sex",
        "listClass": "primary",
        "status": "0",
        "delFlag": "0",
        "createBy": "admin",
        "createTime": "2026-08-05 10:00:00",
        "updateBy": "admin",
        "updateTime": "2026-08-05 10:00:00",
        "remark": ""
      }
    ],
    "total": 1
  },
  "rtState": true,
  "rtMsg": ""
}
```

**SQL：**

```sql
SELECT dict_code, dict_sort, dict_label, dict_value, dict_type, list_class,
       status, del_flag, create_by, create_time, update_by, update_time, remark
FROM sys_dict_data
WHERE del_flag = '0'
  [AND dict_type = ?]
  [AND dict_label LIKE ?]
  [AND status = ?]
ORDER BY dict_sort ASC, dict_code DESC
LIMIT ? OFFSET ?
```

### 3.2 `POST /api/dictData/create` 新增

**请求体：**

```json
{
  "dictSort": 1,
  "dictLabel": "男",
  "dictValue": "0",
  "dictType": "sys_user_sex",
  "listClass": "primary",
  "status": "0",
  "remark": "",
  "createBy": "admin",
  "updateBy": "admin"
}
```

**SQL：**

```sql
INSERT INTO sys_dict_data (dict_sort, dict_label, dict_value, dict_type, list_class,
                           status, del_flag, create_by, create_time, update_by, update_time, remark)
VALUES (?, ?, ?, ?, ?, ?, '0', ?, NOW(), ?, NOW(), ?)
```

### 3.3 `POST /api/dictData/update` 编辑

**请求体（`dictCode` 必填）：**

```json
{
  "dictCode": 1,
  "dictSort": 1,
  "dictLabel": "男",
  "dictValue": "0",
  "dictType": "sys_user_sex",
  "listClass": "primary",
  "status": "0",
  "remark": "已更新",
  "updateBy": "admin"
}
```

**SQL：**

```sql
UPDATE sys_dict_data
SET dict_sort=?, dict_label=?, dict_value=?, dict_type=?, list_class=?,
    status=?, update_by=?, update_time=NOW(), remark=?
WHERE dict_code=?
```

### 3.4 `POST /api/dictData/delete` 逻辑删除

**请求体：**

```json
{ "dictCode": 1 }
```

**SQL：**

```sql
UPDATE sys_dict_data SET del_flag='2', update_time=NOW() WHERE dict_code=?
```

### 3.5 `POST /api/dictData/createTable` 建表

无参，执行 [sys-dict.md 3.1 节](../database/sys-dict.md) DDL，返回 `R.ok("字典数据表创建成功")`。**无需 token**，须加入 `TokenFilter` 白名单。

---

## 四、接口清单总表

| # | 方法 | 路径 | 认证 | 入参 | rtData |
|---|------|------|------|------|--------|
| 1 | POST | `/api/dictType/list` | 是 | Query: page,pageSize,dictName?,dictType?,status? | `{list,total}` |
| 2 | POST | `/api/dictType/create` | 是 | Body: DictType | null |
| 3 | POST | `/api/dictType/update` | 是 | Body: DictType（dictId 必填） | null |
| 4 | POST | `/api/dictType/delete` | 是 | Body: `{dictId}` | null |
| 5 | POST | `/api/dictType/createTable` | 否 | 无 | `"字典类型表创建成功"` |
| 6 | POST | `/api/dictData/list` | 是 | Query: page,pageSize,dictType?,dictLabel?,status? | `{list,total}` |
| 7 | POST | `/api/dictData/create` | 是 | Body: DictData | null |
| 8 | POST | `/api/dictData/update` | 是 | Body: DictData（dictCode 必填） | null |
| 9 | POST | `/api/dictData/delete` | 是 | Body: `{dictCode}` | null |
| 10 | POST | `/api/dictData/createTable` | 否 | 无 | `"字典数据表创建成功"` |

---

## 五、TokenFilter 白名单

在 `filter/TokenFilter.java` 白名单分支追加两行（与 `/api/post/createTable` 并列）：

```java
// 白名单：登录接口、建表接口和静态资源不需要 token
String path = request.getServletPath();
if ("/api/auth/login".equals(path) || "/".equals(path)
        || path.startsWith("/upload/")
        || path.startsWith("/api/order/createTable")
        || path.startsWith("/api/post/createTable")
        || path.startsWith("/api/chat/createTables")
        || path.startsWith("/api/dictType/createTable")    // 新增
        || path.startsWith("/api/dictData/createTable")) {  // 新增
    chain.doFilter(request, response);
    return;
}
```

---

## 六、字段映射表

### 6.1 `sys_dict_type`

| 数据库列 | Java 字段（类型） | TS 字段（类型） |
|----------|-------------------|-----------------|
| `dict_id` | `dictId` (Long) | `dictId: number` |
| `dict_name` | `dictName` (String) | `dictName: string` |
| `dict_type` | `dictType` (String) | `dictType: string` |
| `status` | `status` (String) | `status: string` |
| `del_flag` | `delFlag` (String) | `delFlag: string` |
| `create_by` | `createBy` (String) | `createBy: string` |
| `create_time` | `createTime` (String) | `createTime: string` |
| `update_by` | `updateBy` (String) | `updateBy: string` |
| `update_time` | `updateTime` (String) | `updateTime: string` |
| `remark` | `remark` (String) | `remark: string` |

### 6.2 `sys_dict_data`

| 数据库列 | Java 字段（类型） | TS 字段（类型） |
|----------|-------------------|-----------------|
| `dict_code` | `dictCode` (Long) | `dictCode: number` |
| `dict_sort` | `dictSort` (Integer) | `dictSort: number` |
| `dict_label` | `dictLabel` (String) | `dictLabel: string` |
| `dict_value` | `dictValue` (String) | `dictValue: string` |
| `dict_type` | `dictType` (String) | `dictType: string` |
| `list_class` | `listClass` (String) | `listClass: string` |
| `status` | `status` (String) | `status: string` |
| `del_flag` | `delFlag` (String) | `delFlag: string` |
| `create_by`/`create_time`/`update_by`/`update_time`/`remark` | 同 6.1 | 同 6.1 |

> 时间字段在 Java 实体统一为 `String`，由 MySQL `DATETIME` 转换，前端按字符串处理，不做 `Date` 转换。
