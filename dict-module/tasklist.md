# 字典管理模块任务清单

> 关联文档：[设计总览](./design.md) · [数据库表结构](../../database/sys-dict.md) · [接口契约](../../api/dict-api.md)
> 状态：**待实施**

## 复杂度评估

| # | 任务 | 类型 | 复杂度 | 预估时间 |
|---|------|------|--------|---------|
| 1 | 字典类型后端 CRUD：DictType entity/dao/service/controller + createTable + dict_type 唯一性校验 | 后端 | 🟡 中等 | 2h |
| 2 | 字典数据后端 CRUD：DictData entity/dao/service/controller + createTable | 后端 | 🟢 简单 | 1.5h |
| 3 | 类型删除级联 + dictType 变更同步（DictTypeDao.delete/update 跨 sys_dict_data） | 后端 | 🔴 复杂 | 1h |
| 4 | TokenFilter 白名单放行 /api/dictType/createTable、/api/dictData/createTable | 后端 | 🟢 简单 | 0.2h |
| 5 | 字典类型列表页：api/dictType + DictTypeList.vue + 路由 + 菜单 | 前端 | 🟢 简单 | 1.5h |
| 6 | 字典数据列表页：api/dictData + DictDataList.vue + 路由 + list_class Tag 配色 | 前端 | 🟢 简单 | 1.5h |
| 7 | types/models.d.ts 追加 DictTypeInfo / DictDataInfo 等类型定义 | 前端 | 🟢 简单 | 0.3h |
| 8 | 端到端联调：建表 / 类型+数据 CRUD / 级联删除 / 唯一性 / dictType 变更同步 | 联调 | 🟡 中等 | 1.5h |
| 9 | 回归：cd demo && mvn test、cd frontend && npm run build | 联调 | 🟢 简单 | 0.5h |

合计预估：约 10h

## 分类标准

- 🟢 简单：CRUD、纯配置、模板化页面 -> 快速轨道
- 🟡 中等：涉及一定业务逻辑但边界清晰 -> 快速轨道（加强 review）
- 🔴 复杂：核心业务逻辑、算法、跨模块协作 -> 精细轨道

## 任务依赖

- 任务 1-4（后端）先行；任务 3（级联 / 变更同步）依赖任务 1、2 的表与字段；任务 4 与任务 1、2 共改 `TokenFilter`，建议顺序提交避免冲突。
- 任务 5、6、7（前端）依赖后端接口就绪；任务 6 依赖任务 5（共用 `router`、`types`）。
- 任务 8、9（联调回归）依赖全部任务完成。
- 实现细节见 [design.md](./design.md) 与 [dict-api.md](../../api/dict-api.md)，本文档不重复。
