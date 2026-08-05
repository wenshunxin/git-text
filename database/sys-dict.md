# 字典管理模块数据库表结构

> 数据库：MySQL，地址 `192.168.0.106:3306`，库名 `wsx`
> 驱动：`com.mysql.jdbc.Driver`（mysql-connector-java 5.1.49）
> 所属模块：字典管理（`sys_dict_type` + `sys_dict_data`）
> 关联文档：[接口契约](../api/dict-api.md) · [设计总览](../tasks/dict-module/design.md)
> 状态：**设计稿**

本文档定义字典管理模块两张表的 DDL 与字段语义，是本模块数据库设计的单一事实来源。本模块**不修改**全局 [database-schema.md](./database-schema.md)，独立成文。

---

## 一、概述

字典管理采用 RuoYi 风格双表结构：

- `sys_dict_type`（字典类型表）：维护字典分类，如「用户性别 `sys_user_sex`」「状态 `sys_status`」。
- `sys_dict_data`（字典数据表）：维护每个类型下的具体字典项，如 `0=男 1=女 2=未知`。

两表通过 `dict_type` 字段在应用层关联（无物理外键），与项目 `sys_user`/`sys_role`/`sys_post` 之间不建物理外键的约定一致。

### 通用约定

| 约定 | 说明 |
|------|------|
| 命名风格 | 表名、字段名蛇形（snake_case） |
| 主键 | `dict_id` / `dict_code`，`BIGINT`，`AUTO_INCREMENT` |
| 逻辑删除 | `del_flag`（CHAR(1)，`0`=存在 `2`=删除），所有查询带 `WHERE del_flag='0'` |
| 时间字段 | `DATETIME`，Java 实体映射为 `String`（`yyyy-MM-dd HH:mm:ss`），写入用 `NOW()` |
| 字符集 | `utf8`（与 `sys_role`/`sys_post` 一致） |
| 建表方式 | `CREATE TABLE IF NOT EXISTS`，由 `/api/dictType/createTable`、`/api/dictData/createTable` 幂等执行 |

---

## 二、`sys_dict_type`（字典类型表）

### 2.1 建表 DDL

```sql
CREATE TABLE IF NOT EXISTS sys_dict_type (
  dict_id     bigint       NOT NULL AUTO_INCREMENT COMMENT '字典主键',
  dict_name   varchar(100) NOT NULL DEFAULT ''     COMMENT '字典名称',
  dict_type   varchar(100) NOT NULL DEFAULT ''     COMMENT '字典类型',
  status      char(1)      NOT NULL DEFAULT '0'    COMMENT '状态（0正常 1停用）',
  del_flag    char(1)      NOT NULL DEFAULT '0'    COMMENT '删除标志（0存在 2删除）',
  create_by   varchar(64)  DEFAULT ''              COMMENT '创建者',
  create_time datetime                             COMMENT '创建时间',
  update_by   varchar(64)  DEFAULT ''              COMMENT '更新者',
  update_time datetime                             COMMENT '更新时间',
  remark      varchar(500) DEFAULT NULL            COMMENT '备注',
  PRIMARY KEY (dict_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='字典类型表';
```

### 2.2 字段说明

| 列名 | SQL 类型 | 约束/默认 | 含义 |
|------|----------|-----------|------|
| `dict_id` | BIGINT | PK, AUTO_INCREMENT | 字典主键 |
| `dict_name` | VARCHAR(100) | NOT NULL DEFAULT '' | 字典名称（如「用户性别」） |
| `dict_type` | VARCHAR(100) | NOT NULL DEFAULT '' | 字典类型（如 `sys_user_sex`），应用层唯一 |
| `status` | CHAR(1) | NOT NULL DEFAULT '0' | `0`=正常 `1`=停用 |
| `del_flag` | CHAR(1) | NOT NULL DEFAULT '0' | `0`=存在 `2`=删除 |
| `create_by` | VARCHAR(64) | DEFAULT '' | 创建者（登录账号） |
| `create_time` | DATETIME | - | 创建时间（`NOW()`） |
| `update_by` | VARCHAR(64) | DEFAULT '' | 更新者（登录账号） |
| `update_time` | DATETIME | - | 更新时间（`NOW()`） |
| `remark` | VARCHAR(500) | DEFAULT NULL | 备注 |

- 查询：`WHERE del_flag='0'`，排序 `ORDER BY dict_id DESC`。
- `dict_type` 不建数据库唯一索引（见第四节）。

---

## 三、`sys_dict_data`（字典数据表）

### 3.1 建表 DDL

```sql
CREATE TABLE IF NOT EXISTS sys_dict_data (
  dict_code   bigint       NOT NULL AUTO_INCREMENT COMMENT '字典编码',
  dict_sort   int          NOT NULL DEFAULT 0      COMMENT '字典排序',
  dict_label  varchar(100) NOT NULL DEFAULT ''     COMMENT '字典标签',
  dict_value  varchar(100) NOT NULL DEFAULT ''     COMMENT '字典键值',
  dict_type   varchar(100) NOT NULL DEFAULT ''     COMMENT '字典类型',
  list_class  varchar(100) DEFAULT NULL            COMMENT '表格回显样式（default/primary/success/info/warning/danger）',
  status      char(1)      NOT NULL DEFAULT '0'    COMMENT '状态（0正常 1停用）',
  del_flag    char(1)      NOT NULL DEFAULT '0'    COMMENT '删除标志（0存在 2删除）',
  create_by   varchar(64)  DEFAULT ''              COMMENT '创建者',
  create_time datetime                             COMMENT '创建时间',
  update_by   varchar(64)  DEFAULT ''              COMMENT '更新者',
  update_time datetime                             COMMENT '更新时间',
  remark      varchar(500) DEFAULT NULL            COMMENT '备注',
  PRIMARY KEY (dict_code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='字典数据表';
```

### 3.2 字段说明

| 列名 | SQL 类型 | 约束/默认 | 含义 |
|------|----------|-----------|------|
| `dict_code` | BIGINT | PK, AUTO_INCREMENT | 字典编码（主键） |
| `dict_sort` | INT | NOT NULL DEFAULT 0 | 显示顺序（升序） |
| `dict_label` | VARCHAR(100) | NOT NULL DEFAULT '' | 字典标签（如「男」） |
| `dict_value` | VARCHAR(100) | NOT NULL DEFAULT '' | 字典键值（如 `0`） |
| `dict_type` | VARCHAR(100) | NOT NULL DEFAULT '' | 字典类型，关联 `sys_dict_type.dict_type` |
| `list_class` | VARCHAR(100) | DEFAULT NULL | 表格回显样式：`default`/`primary`/`success`/`info`/`warning`/`danger`，供前端 Tag 配色 |
| `status` | CHAR(1) | NOT NULL DEFAULT '0' | `0`=正常 `1`=停用 |
| `del_flag` | CHAR(1) | NOT NULL DEFAULT '0' | `0`=存在 `2`=删除 |
| `create_by` | VARCHAR(64) | DEFAULT '' | 创建者 |
| `create_time` | DATETIME | - | 创建时间 |
| `update_by` | VARCHAR(64) | DEFAULT '' | 更新者 |
| `update_time` | DATETIME | - | 更新时间 |
| `remark` | VARCHAR(500) | DEFAULT NULL | 备注 |

- 查询：`WHERE del_flag='0'`，排序 `ORDER BY dict_sort ASC, dict_code DESC`。

> 相比 RuoYi 原版 `sys_dict_data`，本模块去除 `css_class`（样式属性，过细）与 `is_default`（默认值逻辑本模块用不到），保留 `list_class`（前端 Tag 配色为字典核心展示特性）。遵循 YAGNI，与项目 `sys_post`/`sys_role` 精简风格一致。

---

## 四、删除策略与唯一性约定

### 4.1 逻辑删除

两表均使用 `del_flag` 逻辑删除：

- **查询**：一律 `WHERE del_flag='0'`，已删除数据不可见。
- **删除**：`UPDATE ... SET del_flag='2', update_time=NOW() WHERE <主键>=?`，不物理删除，数据可追溯。
- **新增**：`del_flag` 由 SQL 默认值 `'0'` 写入，前端无需传。

### 4.2 级联删除

删除一个字典类型时，**级联逻辑删除**其下全部字典数据：

```sql
-- 1) 先逻辑删除该类型下所有数据
UPDATE sys_dict_data SET del_flag='2', update_time=NOW() WHERE dict_type=?;
-- 2) 再逻辑删除类型本身
UPDATE sys_dict_type SET del_flag='2', update_time=NOW() WHERE dict_id=?;
```

> 删除类型前不做「有数据则拒绝」拦截，直接级联清理，与 RuoYi 原版行为一致；数据可追溯（`del_flag='2'`）。

### 4.3 `dict_type` 唯一性

`dict_type` 是字典类型的业务标识，**不建数据库唯一索引**，改由应用层校验：

- 新增/编辑类型时，查询 `SELECT COUNT(*) FROM sys_dict_type WHERE del_flag='0' AND dict_type=? AND dict_id<>?`（编辑排除自身），>0 则返回「字典类型已存在」。
- 不加 DB 唯一索引的原因：逻辑删除后唯一索引会阻止重建同名类型；且项目 `sys_user.login_name`/`sys_role.role_key`/`sys_post.post_code` 均无 DB 唯一索引、靠应用层校验，保持一致。

### 4.4 `dict_type` 变更同步

编辑字典类型时若 `dict_type` 发生变更，须同步更新 `sys_dict_data.dict_type`，保证关联一致：

```sql
UPDATE sys_dict_data SET dict_type=?, update_time=NOW() WHERE dict_type=? AND del_flag='0';
```

---

## 五、表关系

```
sys_dict_type.dict_type ──< sys_dict_data.dict_type（应用层关联，无物理外键）
```

- `sys_dict_data.dict_type` 的取值应存在于 `sys_dict_type.dict_type`（`del_flag='0'`）中，由应用层保证，不建外键约束。
- 删除类型时通过第 4.2 节级联删除保持数据一致。
- 类型 `dict_type` 变更时通过第 4.4 节同步保持数据一致。
