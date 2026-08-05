# 菜单管理

菜单树形结构 CRUD

---

## 菜单列表（全量）

查询所有菜单，以树形结构返回。

- **Method：** GET
- **URL：** /api/menu/list
- **Auth：** 需要认证

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": [
    {
      "menuId": 1,
      "menuName": "系统管理",
      "parentId": 0,
      "orderNum": 1,
      "url": "/system",
      "target": "menuItem",
      "menuType": "M",
      "visible": "0",
      "perms": "",
      "icon": "system",
      "children": []
    }
  ]
}
```

**说明：** `rtData` 为 `Menu[]` 数组，每个菜单节点包含 `children` 子菜单列表，构成完整树形结构。

---

## 查询单条菜单

根据菜单 ID 查询单条菜单记录。

- **Method：** POST
- **URL：** /api/menu/getById
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| menuId | integer | 是 | 菜单ID |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "menuId": 1,
    "menuName": "系统管理",
    "parentId": 0,
    "orderNum": 1,
    "url": "/system",
    "target": "menuItem",
    "menuType": "M",
    "visible": "0",
    "perms": "",
    "icon": "system"
  }
}
```

---

## 新增菜单

新增一条菜单记录，`menuName` 为必填字段。

- **Method：** POST
- **URL：** /api/menu/create
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| menuName | string | 是 | - | 菜单名称 |
| parentId | integer | 否 | - | 父菜单ID |
| orderNum | integer | 否 | - | 排序号 |
| url | string | 否 | - | 路由地址 |
| target | string | 否 | - | 打开方式 |
| menuType | string | 否 | - | M-目录 C-菜单 F-按钮 |
| visible | string | 否 | - | 0-显示 1-隐藏 |
| perms | string | 否 | - | 权限标识 |
| icon | string | 否 | - | 图标 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 编辑菜单

编辑一条已有菜单，`menuId` 为必填字段。

- **Method：** POST
- **URL：** /api/menu/update
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| menuId | integer | 是 | - | 菜单ID |
| menuName | string | 否 | - | 菜单名称 |
| parentId | integer | 否 | - | 父菜单ID |
| orderNum | integer | 否 | - | 排序号 |
| url | string | 否 | - | 路由地址 |
| menuType | string | 否 | - | M-目录 C-菜单 F-按钮 |
| visible | string | 否 | - | 0-显示 1-隐藏 |
| icon | string | 否 | - | 图标 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 删除菜单

删除一条菜单记录，含子菜单检查：存在子菜单时不可删除。

- **Method：** POST
- **URL：** /api/menu/delete
- **Auth：** 需要认证
- **Content-Type：** application/json

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| menuId | integer | 是 | - | 菜单ID |

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
  "rtMsg": "存在子菜单，不允许删除",
  "rtData": null
}
```
