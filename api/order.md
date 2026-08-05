# 订单管理

订单 CRUD + 统计 + 建表

服务器地址：`http://localhost:8081`

---

## 订单列表（分页）

**描述：** 分页查询订单列表，支持按订单状态和关键词（客户名称或订单号模糊匹配）筛选。

**Method：** `GET`

**URL：** `/api/order/list`

**Auth：** 需要认证

### 请求参数（Query）

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| page | integer | 是 | 1 | 页码 |
| pageSize | integer | 是 | 10 | 每页条数 |
| orderStatus | integer | 否 | - | 0-待付款 1-进行中 2-已完成 3-已取消 |
| keyword | string | 否 | - | 客户名称或订单号（模糊） |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "list": [],
    "total": 25
  }
}
```

---

## 订单统计

**描述：** 获取各状态订单数量统计。

**Method：** `GET`

**URL：** `/api/order/stats`

**Auth：** 需要认证

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": {
    "all": 25,
    "pending": 5,
    "inProgress": 8,
    "completed": 10
  }
}
```

---

## 新增订单

**描述：** 新增一条订单，订单号由后端自动生成（yyyyMMdd + 4位随机数）。

**Method：** `POST`

**URL：** `/api/order/create`

**Auth：** 需要认证

**Content-Type：** `application/json`

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| customerName | string | 是 | 客户名称 |
| quantity | integer | 是 | 数量 |
| unitPrice | number | 是 | 单价 |
| totalAmount | number | 是 | 总金额 |
| orderStatus | integer | 否 | 0-待付款 1-进行中 2-已完成 3-已取消 |
| paymentMethod | string | 否 | 付款方式 |
| remark | string | 否 | 备注 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 编辑订单

**描述：** 编辑已有订单，根据 id 更新。

**Method：** `POST`

**URL：** `/api/order/update`

**Auth：** 需要认证

**Content-Type：** `application/json`

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | integer | 是 | 订单ID |
| customerName | string | 否 | 客户名称 |
| quantity | integer | 否 | 数量 |
| unitPrice | number | 否 | 单价 |
| totalAmount | number | 否 | 总金额 |
| orderStatus | integer | 否 | 0-待付款 1-进行中 2-已完成 3-已取消 |
| paymentMethod | string | 否 | 付款方式 |
| remark | string | 否 | 备注 |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 删除订单

**描述：** 逻辑删除订单（delStatus = 0）。

**Method：** `POST`

**URL：** `/api/order/delete`

**Auth：** 需要认证

**Content-Type：** `application/json`

### 请求参数（Body）

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | integer | 是 | 订单ID |

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": null
}
```

---

## 创建订单表

**描述：** 在数据库中创建订单数据表（白名单接口，无需认证）。

**Method：** `POST`

**URL：** `/api/order/createTable`

**Auth：** 无需认证（白名单）

**Content-Type：** `application/json`

### 响应示例

```json
{
  "rtState": true,
  "rtMsg": "",
  "rtData": "订单表创建成功"
}
```
