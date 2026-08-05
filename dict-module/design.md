# 字典管理模块设计总览

> 创建日期：2026-08-05
> 适用范围：`demo/`（Spring Boot 后端）+ `frontend/`（Vue 3 前端）
> 所属位置：系统管理子菜单（与「角色/菜单/岗位管理」并列）
> 关联文档：[数据库表结构](../../database/sys-dict.md) · [接口契约](../../api/dict-api.md) · [任务清单](./tasklist.md)
> 状态：**设计稿**

本模块新增「字典类型表 `sys_dict_type`」与「字典数据表 `sys_dict_data`」及完整 CRUD 能力，前后端字段统一 RuoYi 风格，复用现有统一响应体 `R`、`@ApiPrefix` 前缀机制、`TokenFilter` 鉴权与 `usePagination`/`useRequest` 组合式函数，保持与「岗位管理」「角色管理」模块一致的代码风格与目录结构。

---

## 目录

- [一、概述](#一概述)
- [二、数据库设计](#二数据库设计)
- [三、接口设计](#三接口设计)
- [四、后端文件清单](#四后端文件清单)
- [五、前端文件清单与交互](#五前端文件清单与交互)
- [六、删除与唯一性约定](#六删除与唯一性约定)
- [七、验证步骤](#七验证步骤)
- [八、与现有模式的关系](#八与现有模式的关系)

---

## 一、概述

### 1.1 模块目标

字典管理是系统管理子模块之一，用于维护系统内各类枚举数据（如用户性别、状态、是否等），为后续业务模块的下拉框、标签展示等提供统一数据源。本模块提供：

- 字典类型分页查询（按名称、类型、状态筛选）、新增 / 编辑 / 逻辑删除（级联清理数据）
- 字典数据分页查询（按类型、标签、状态筛选）、新增 / 编辑 / 逻辑删除
- 两表首次部署一键建表（`IF NOT EXISTS`，幂等）

### 1.2 所属位置

```
系统管理
├── 角色管理  /sys/role
├── 菜单管理  /sys/menu
├── 岗位管理  /sys/post
└── 字典管理  /sys/dict/type   ← 新增（侧边栏唯一入口）
```

侧边栏只放一个「字典管理」入口进**字典类型列表页**；**字典数据列表页**通过类型页点「数据」按钮路由跳转进入（`/sys/dict/data?dictType=xxx`），不在侧边栏单独列出。这与 RuoYi 原版一致。

### 1.3 设计原则

- 与 `sys_post`/`sys_role` 模块**同构**：相同分层（entity -> dao -> service -> controller）、逻辑删除（`del_flag`）、分页约定（`LIMIT ? OFFSET ?`）、命名与时间字段约定。
- 字段命名 RuoYi 风格（蛇形 DB 列 ↔ 驼峰 Java/TS）。
- 数据访问全用 `JdbcTemplate` 手写 SQL，无 ORM。
- 手写 getter/setter，不依赖 Lombok（遵循 [CLAUDE.md](../../CLAUDE.md) 约定，Lombok 注解处理器不可靠）。

---

## 二、数据库设计

详见 [sys-dict.md](../../database/sys-dict.md)。要点：

- 两表均带 `del_flag`（`0`=存在 / `2`=删除），查询带 `WHERE del_flag='0'`。
- 删除字典类型**级联逻辑删除**其下全部数据。
- `dict_type` 不建 DB 唯一索引，由应用层校验（新增/编辑查 `del_flag='0'` 同名记录）。
- 字符集 `utf8`，与 `sys_role`/`sys_post` 一致。
- 精简：相比 RuoYi 原版去除 `css_class`、`is_default`，保留 `list_class`（前端 Tag 配色）。

---

## 三、接口设计

详见 [dict-api.md](../../api/dict-api.md)。共 10 个 POST 接口：

- 字典类型 `/api/dictType/*`：list / create / update / delete / createTable
- 字典数据 `/api/dictData/*`：list / create / update / delete / createTable

`list` 走 Query，`create`/`update`/`delete` 走 Body，`createTable` 无参免认证。路径驼峰双词，与 `/api/loginLog` 一致。所有接口返回统一响应体 `R`。

---

## 四、后端文件清单

结构与 `PostController`/`PostDao` 一一对应。包 `com.example`。

| 层 | 文件（`demo/src/main/java/com/example/`） | 职责 |
|----|------------------------------------------|------|
| 实体 | `entity/DictType.java` | `sys_dict_type` 实体，10 字段，手写 getter/setter |
| 实体 | `entity/DictData.java` | `sys_dict_data` 实体，13 字段，手写 getter/setter |
| 数据访问 | `dao/DictTypeDao.java` | `JdbcTemplate` + `ROW_MAPPER`；`listByPage`/`count`/`insert`/`update`/`delete`/`createTable`/`existsByDictType`；`delete` 含级联；`insert`/`update` 涉及 `dict_type` 唯一性 |
| 数据访问 | `dao/DictDataDao.java` | 同结构；`listByPage`/`count`/`insert`/`update`/`delete`/`createTable` |
| 业务接口 | `service/DictTypeService.java`、`service/DictDataService.java` | 方法签名与 DAO 对应 |
| 业务实现 | `service/impl/DictTypeServiceImpl.java`、`service/impl/DictDataServiceImpl.java` | `@Service`，委托 DAO |
| 控制器 | `controller/DictTypeController.java` | `@RestController @ApiPrefix @RequestMapping("/dictType")`，5 接口 |
| 控制器 | `controller/DictDataController.java` | `@RestController @ApiPrefix @RequestMapping("/dictData")`，5 接口 |
| 过滤器（**修改**） | `filter/TokenFilter.java` | 白名单追加 `/api/dictType/createTable`、`/api/dictData/createTable` |

### 关键实现点

- **唯一性校验**：`DictTypeDao` 提供 `existsByDictType(String dictType, Long excludeId)`，`insert`（excludeId 为 null）/`update`（excludeId 为当前 dictId）前由 Controller 调用，重复返回 `R.fail("字典类型已存在")`。
- **级联删除**：`DictTypeDao.delete(dictId)` 先查 `dict_type`，再 `UPDATE sys_dict_data SET del_flag='2' WHERE dict_type=?`，再 `UPDATE sys_dict_type SET del_flag='2' WHERE dict_id=?`。
- **`dictType` 变更同步**：`DictTypeDao.update` 若 `dictType` 变更，同步 `UPDATE sys_dict_data SET dict_type=? WHERE dict_type=? AND del_flag='0'`。
- **createTable**：放 DAO 层 `jdbcTemplate.execute(DDL)`（与 `PostDao.createTable` 一致），`CREATE TABLE IF NOT EXISTS` 幂等。

---

## 五、前端文件清单与交互

Vue 3 + TS + Ant Design Vue 4，4 空格缩进，新函数写 JSDoc。

| 类型 | 文件（`frontend/src/`） | 职责 |
|------|------------------------|------|
| API 封装 | `api/dictType/index.ts` | 5 接口：`getDictTypeList`/`createDictType`/`updateDictType`/`deleteDictType`/`createDictTypeTable` |
| API 封装 | `api/dictData/index.ts` | 5 接口：`getDictDataList`/`createDictData`/`updateDictData`/`deleteDictData`/`createDictDataTable` |
| 类型 | `types/models.d.ts`（追加） | `DictTypeInfo`/`DictDataInfo`/`CreateDictTypeParams`/`UpdateDictTypeParams`/`CreateDictDataParams`/`UpdateDictDataParams` |
| 页面 | `views/sys/dict/DictTypeList.vue` | 类型列表页：搜索栏（dictName/dictType/status）+ 表格 + 新增/编辑弹窗 + 删除确认 + 每行「数据」按钮 |
| 页面 | `views/sys/dict/DictDataList.vue` | 数据列表页：进入时读 `route.query.dictType` 作为筛选默认值；搜索栏（dictLabel/status）+ 表格（含 `list_class` Tag 配色）+ CRUD |
| 路由 | `router/index.ts`（追加） | `sys/dict/type`、`sys/dict/data` 两条路由 |
| 菜单 | `layout/AppLayout.vue`（追加） | 「系统管理」children 追加 `{ key:'/sys/dict/type', label:'字典管理' }` |

### 交互细节

- **类型页 -> 数据页**：类型表格操作列加「数据」按钮，点击 `router.push({ path: '/sys/dict/data', query: { dictType: row.dictType } })`。
- **数据页初始化**：`onMounted` 读 `route.query.dictType`，若有则作为 `dictType` 筛选值并加载列表；顶部显示「当前字典类型：xxx」与返回按钮（`router.back()`）。
- **`list_class` 渲染**：数据页表格 `dict_label` 列用 `<a-tag :color="listClassMap[record.listClass]">{{ record.dictLabel }}</a-tag>` 配色，map 含 `primary`/`success`/`info`/`warning`/`danger`/`default`。
- **审计字段注入**：新增/编辑提交时 `createBy`/`updateBy` = `authStore.loginName`（与 `PostList.vue` 一致）。
- **删除确认**：`Modal.confirm`，类型删除提示「将同时删除该类型下所有字典数据」。

---

## 六、删除与唯一性约定

见 [sys-dict.md 第四节](../../database/sys-dict.md)。摘要：

- 两表 `del_flag` 逻辑删除，查询带 `WHERE del_flag='0'`。
- 删除类型**级联逻辑删除**其下数据。
- `dict_type` 应用层校验唯一，不建 DB 唯一索引。
- `dict_type` 变更同步更新 `sys_dict_data.dict_type`。

---

## 七、验证步骤

### 7.1 启动服务

```bash
cd demo && mvn spring-boot:run      # 后端，监听 8081
cd frontend && npm run dev          # 前端，/api 代理到 8081
```

> `JAVA_HOME` 必须指向 JDK（不能是 JRE），否则 Maven 编译报 `No compiler provided`。

### 7.2 建表（两表，幂等）

```bash
curl -X POST http://localhost:8081/api/dictType/createTable
# 期望：{"rtData":"字典类型表创建成功","rtState":true,"rtMsg":""}
curl -X POST http://localhost:8081/api/dictData/createTable
# 期望：{"rtData":"字典数据表创建成功","rtState":true,"rtMsg":""}
```

### 7.3 登录获取 token

```bash
curl -X POST http://localhost:8081/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"loginName":"admin","password":"admin123"}'
# 取响应 rtData.token 存入 TOKEN
```

### 7.4 接口联调

```bash
# 1) 新增字典类型
curl -X POST http://localhost:8081/api/dictType/create \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"dictName":"用户性别","dictType":"sys_user_sex","status":"0","remark":"测试","createBy":"admin","updateBy":"admin"}'
# 期望：rtState=true, rtData=null

# 2) 新增字典数据
curl -X POST http://localhost:8081/api/dictData/create \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"dictSort":1,"dictLabel":"男","dictValue":"0","dictType":"sys_user_sex","listClass":"primary","status":"0","remark":"","createBy":"admin","updateBy":"admin"}'

# 3) 数据列表（按类型筛）
curl -X POST "http://localhost:8081/api/dictData/list?page=1&pageSize=10&dictType=sys_user_sex" \
  -H "Authorization: Bearer $TOKEN"
# 期望：rtData.list 含上一步数据，rtData.total >= 1

# 4) 唯一性校验（重复 dictType）
curl -X POST http://localhost:8081/api/dictType/create \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"dictName":"性别","dictType":"sys_user_sex","status":"0","createBy":"admin","updateBy":"admin"}'
# 期望：rtState=false, rtMsg="字典类型已存在"

# 5) 删除类型（级联删数据）
curl -X POST http://localhost:8081/api/dictType/delete \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"dictId":1}'
# 期望：rtState=true；再次 list dictType=sys_user_sex 数据不再返回
```

### 7.5 前端页面验证

1. 登录后「系统管理 -> 字典管理」进入类型列表 `/sys/dict/type`。
2. 类型 CRUD 正常，分页 / 筛选生效。
3. 点类型行「数据」跳转 `/sys/dict/data?dictType=xxx`，数据页按类型筛选加载。
4. 数据 CRUD 正常，`list_class` Tag 配色生效。
5. 删除类型后，其下数据一并不可见。
6. 回归：`cd demo && mvn test`、`cd frontend && npm run build` 均通过。

---

## 八、与现有模式的关系

本模块完全遵循项目现有约定，不引入新机制：

| 维度 | 现有模块（post/role） | 本模块 |
|------|----------------------|--------|
| 分层 | entity -> dao -> service -> controller | 同 |
| 数据访问 | JdbcTemplate 手写 SQL | 同 |
| 响应体 | R（rtData/rtState/rtMsg） | 同 |
| 前缀 | @ApiPrefix + @RequestMapping | `/dictType`、`/dictData` |
| 逻辑删除 | del_flag（role），物理删除（post） | del_flag（两表） |
| 建表 | createTable IF NOT EXISTS（post/order） | 同，两表各一 |
| 前端 | api/xxx + views/sys/xxx + usePagination | 同，双页面 |

**差异点**：本模块是项目首个**双表 + 级联删除 + 跨表字段同步**的模块。级联删除的两条 `UPDATE` 与 `dictType` 变更同步的 `UPDATE` 在同一 service 方法内顺序执行即可（`JdbcTemplate` 自动提交下各自原子）；若需强一致可加 `@Transactional`，本期默认不加。

**文档独立**：本模块的表结构与接口契约分别独立成文（[sys-dict.md](../../database/sys-dict.md)、[dict-api.md](../../api/dict-api.md)），**不追加**到全局 `database-schema.md` 与 `api-contract.md`。
