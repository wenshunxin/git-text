# 数据库表结构文档

> 数据库：MySQL，地址 `192.168.0.106:3306`，库名 `wsx`
> 驱动：`com.mysql.jdbc.Driver`（mysql-connector-java 5.1.49）
> 维护方：全栈开发组
> 状态：**现行**（与 `docs/api/api-contract.md` 第八章数据模型保持同步）

本文档汇总项目全部数据库表结构，是数据库设计的单一事实来源。表结构变更须同步更新本文档与 `api-contract.md`。

---

## 一、命名与通用约定

| 约定 | 说明 |
|------|------|
| 命名风格 | 表名、字段名均为蛇形（snake_case） |
| 主键 | `<前缀>_id`，`BIGINT` 或 `INT`，`AUTO_INCREMENT` |
| 逻辑删除 | `del_flag`（CHAR，`0`=存在 `2`=删除）或 `del_status`（TINYINT，`1`=正常 `0`=删除） |
| 物理删除 | 直接 `DELETE FROM`，无删除标志字段 |
| 时间字段 | `DATETIME`，Java 实体映射为 `String`（格式 `yyyy-MM-dd HH:mm:ss`），写入用 `NOW()` |
| 字符集 | `utf8mb4`（`chat_message` 需支持 Emoji） |
| 建表方式 | `sys_user`/`sys_role`/`sys_menu` 为 RuoYi 原版已存在；`order`/`chat_session`/`chat_message`/`sys_post` 可由对应 `createTable` 接口建（`IF NOT EXISTS`） |

> **注意**：`order` 是 MySQL 保留字，SQL 中须用反引号 `` `order` ``。

---

## 二、删除策略一览

| 表 | 删除方式 | 标志/机制 | 说明 |
|----|----------|-----------|------|
| `sys_user` | 逻辑删除 | `del_flag='2'` | 查询 `WHERE del_flag='0'` |
| `sys_role` | 逻辑删除 | `del_flag='2'` | 查询 `WHERE del_flag='0'` |
| `sys_menu` | 物理删除 | `DELETE FROM` | 删除前校验子菜单，有子菜单则拒绝 |
| `sys_post` | 物理删除 | `DELETE FROM` | RuoYi 原版无 `del_flag` |
| `order` | 逻辑删除 | `del_status=0` | 查询 `WHERE del_status=1` |
| `chat_session` | 物理删除 | `DELETE FROM` | 级联删除关联 `chat_message` |
| `chat_message` | 物理删除 | `DELETE FROM` | — |

---

## 三、表结构详情

### 3.1 `sys_user`（用户表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `user_id` | BIGINT | 否 | AUTO_INCREMENT | 主键 |
| `login_name` | VARCHAR | 否 | | 登录账号 |
| `user_name` | VARCHAR | 否 | | 姓名 |
| `sex` | VARCHAR | 是 | | 性别 |
| `avatar` | VARCHAR | 是 | | 头像 URL |
| `password` | VARCHAR | 否 | | 密码 `MD5(loginName+password+salt)` |
| `salt` | VARCHAR | 否 | | 盐（6 位随机数字） |
| `status` | CHAR(1) | 否 | | `0`=正常 `1`=停用 |
| `del_flag` | CHAR(1) | 否 | `0` | `0`=存在 `2`=删除 |
| `create_time` | DATETIME | 是 | | 创建时间 |
| `update_time` | DATETIME | 是 | | 更新时间 |

- 查询：`WHERE del_flag='0'`，排序 `ORDER BY user_id DESC`
- 登录：按 `login_name` 查询，校验 `status` 与密码

### 3.2 `sys_role`（角色表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `role_id` | BIGINT | 否 | AUTO_INCREMENT | 主键 |
| `role_name` | VARCHAR | 否 | | 角色名称 |
| `role_key` | VARCHAR | 否 | | 角色权限字符 |
| `role_sort` | INT | 否 | | 显示顺序 |
| `data_scope` | CHAR(1) | 否 | | `1`=全部 `2`=自定义 |
| `status` | CHAR(1) | 否 | | `0`=正常 `1`=停用 |
| `del_flag` | CHAR(1) | 否 | `0` | `0`=正常 `2`=已删除 |
| `create_by` | VARCHAR | 是 | | 创建者 |
| `create_time` | DATETIME | 是 | | 创建时间 |
| `update_by` | VARCHAR | 是 | | 更新者 |
| `update_time` | DATETIME | 是 | | 更新时间 |
| `remark` | VARCHAR | 是 | | 备注 |

- 查询：`WHERE del_flag='0'`，排序 `ORDER BY role_sort ASC, role_id DESC`

### 3.3 `sys_menu`（菜单表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `menu_id` | BIGINT | 否 | AUTO_INCREMENT | 主键 |
| `menu_name` | VARCHAR | 否 | | 菜单名称 |
| `parent_id` | BIGINT | 否 | `0` | 父菜单 ID（`0`=顶级） |
| `order_num` | INT | 否 | | 显示顺序 |
| `url` | VARCHAR | 是 | | 路由地址 |
| `target` | VARCHAR | 是 | | 打开方式 |
| `menu_type` | CHAR(1) | 否 | | `M`=目录 `C`=菜单 `F`=按钮 |
| `visible` | CHAR(1) | 否 | | `0`=显示 `1`=隐藏 |
| `is_refresh` | CHAR(1) | 否 | `1` | 是否刷新 |
| `perms` | VARCHAR | 是 | | 权限标识 |
| `icon` | VARCHAR | 是 | | 图标 |
| `create_by`/`create_time`/`update_by`/`update_time`/`remark` | — | 是 | | 审计字段 |

- **无 `del_flag`**，全量查询，排序 `ORDER BY parent_id ASC, order_num ASC`
- 删除：物理删除，删除前校验子菜单

### 3.4 `sys_post`（岗位表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `post_id` | BIGINT | 否 | AUTO_INCREMENT | 主键 |
| `post_code` | VARCHAR(64) | 否 | | 岗位编码 |
| `post_name` | VARCHAR(50) | 否 | | 岗位名称 |
| `post_sort` | INT | 否 | `0` | 显示顺序 |
| `status` | CHAR(1) | 否 | `0` | `0`=正常 `1`=停用 |
| `create_by` | VARCHAR(64) | 是 | | 创建者 |
| `create_time` | DATETIME | 是 | | 创建时间 |
| `update_by` | VARCHAR(64) | 是 | | 更新者 |
| `update_time` | DATETIME | 是 | | 更新时间 |
| `remark` | VARCHAR(500) | 是 | | 备注 |

- **RuoYi 原版结构，无 `del_flag`**，删除为物理删除
- 查询：`WHERE 1=1`（可叠加 `post_code`/`post_name` LIKE、`status` 精确），排序 `ORDER BY post_sort ASC, post_id DESC`
- 建表：`/api/post/createTable`（`IF NOT EXISTS`）

> 历史教训：生成岗位模块时曾误加 `del_flag`，因表已存在 `IF NOT EXISTS` 未重建，导致 `Unknown column 'del_flag'`。详见 `docs/tasks/post-fix-del-flag.md`。

### 3.5 `order`（订单表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `id` | INT | 否 | AUTO_INCREMENT | 主键 |
| `order_no` | VARCHAR(50) | 否 | | 订单号（后端生成：`yyyyMMdd`+4位随机） |
| `customer_name` | VARCHAR(100) | 否 | | 客户名称 |
| `quantity` | INT | 否 | `0` | 数量 |
| `unit_price` | DECIMAL(10,2) | 否 | `0` | 单价 |
| `total_amount` | DECIMAL(10,2) | 否 | `0` | 金额 |
| `order_status` | TINYINT | 否 | `0` | `0`=待付款 `1`=进行中 `2`=已完成 `3`=已取消 |
| `payment_method` | VARCHAR(50) | 是 | | 付款方式 |
| `remark` | VARCHAR(500) | 是 | | 备注 |
| `add_time` | DATETIME | 是 | CURRENT_TIMESTAMP | 创建时间 |
| `update_time` | DATETIME | 是 | ON UPDATE CURRENT_TIMESTAMP | 更新时间 |
| `del_status` | TINYINT | 否 | `1` | `1`=正常 `0`=已删除 |

- 查询：`WHERE del_status=1`，`keyword` 模糊匹配 `order_no` 或 `customer_name`，排序 `ORDER BY id DESC`
- ⚠️ `order` 为保留字，SQL 须反引号

### 3.6 `chat_session`（聊天会话表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `id` | INT | 否 | AUTO_INCREMENT | 主键 |
| `user_sid` | INT | 否 | | 用户 ID（= `sys_user.user_id`） |
| `title` | VARCHAR(100) | 否 | `新的聊天` | 会话标题 |
| `ai_model` | VARCHAR(50) | 否 | `deepseek-chat` | AI 模型 |
| `icon_url` | VARCHAR(500) | 是 | | 会话头像 URL |
| `created_at` | DATETIME | 是 | CURRENT_TIMESTAMP | 创建时间 |
| `updated_at` | DATETIME | 是 | ON UPDATE CURRENT_TIMESTAMP | 更新时间 |

- 按 `user_sid` 查询，排序 `ORDER BY updated_at DESC`
- 删除：物理删除，级联删除 `chat_message`
- 遗留：`user_sid` 字段名未随 `sys_user` 重命名改动，值存 `user_id`

### 3.7 `chat_message`（聊天消息表）

| 列名 | 类型 | 可空 | 默认 | 含义 |
|------|------|------|------|------|
| `id` | INT | 否 | AUTO_INCREMENT | 主键 |
| `session_id` | INT | 否 | | 会话 ID |
| `role` | VARCHAR(20) | 否 | | `user` / `assistant` |
| `content` | TEXT | 否 | | 消息内容 |
| `created_at` | DATETIME | 是 | CURRENT_TIMESTAMP | 创建时间 |

- 字符集 `utf8mb4`（支持 Emoji）
- 按 `session_id` 查询，排序 `ORDER BY created_at ASC`（时间正序）

---

## 四、表关系

```
sys_user.user_id ──< chat_session.user_sid
chat_session.id  ──< chat_message.session_id
sys_menu.parent_id ─> sys_menu.menu_id（自引用树形）
```

> `sys_user`/`sys_role`/`sys_menu`/`sys_post` 之间在当前项目中**未建立物理外键**，关联通过应用层维护。
