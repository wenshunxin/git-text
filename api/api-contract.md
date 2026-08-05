# 前后端开发契约

> 创建日期：2026-08-04
> 适用范围：`demo/`（Spring Boot 后端）+ `frontend/`（Vue 3 前端）
> 维护方：全栈开发组
> 状态：**现行**

本契约是前后端协作的**单一事实来源（Single Source of Truth）**。所有新增接口、字段、状态码均以本文档为准；任何一方修改涉及契约的内容，必须同步更新本文档并通知另一方。

---

## 目录

- [一、技术栈约定](#一技术栈约定)
- [二、通用约定](#二通用约定)
- [三、认证与授权契约](#三认证与授权契约)
- [四、CORS 与代理约定](#四cors-与代理约定)
- [五、分页契约](#五分页契约)
- [六、文件上传契约](#六文件上传契约)
- [七、接口清单](#七接口清单)
- [八、数据模型契约](#八数据模型契约)
- [九、前端开发约定](#九前端开发约定)
- [十、SSE 流式契约（聊天）](#十sse-流式契约聊天)
- [十一、错误处理契约](#十一错误处理契约)
- [十二、协作流程与规范](#十二协作流程与规范)
- [十三、已知问题与改进建议](#十三已知问题与改进建议)

---

## 一、技术栈约定

### 1.1 后端（`demo/`）

| 项目 | 版本/说明 |
|------|-----------|
| Spring Boot | 1.2.3.RELEASE（基于 Spring 4.1，**无内置 CORS 支持**，需手动实现） |
| Java | 1.8（JDK 8） |
| 构建工具 | Maven |
| 数据库 | MySQL（驱动 `mysql-connector-java` 5.1.49，旧驱动类 `com.mysql.jdbc.Driver`） |
| 数据库连接 | `jdbc:mysql://192.168.0.106:3306/wsx` |
| 数据访问 | **无 ORM**，全部使用 `JdbcTemplate` 手写 SQL |
| 鉴权 | jjwt 0.9.1（HS256 签名） |
| AI 能力 | DeepSeek API（`deepseek-chat` 模型，流式 SSE） |
| 代码增强 | Lombok（provided scope，IDE 注解处理器可能不可用，**优先手写 getter/setter**） |

### 1.2 前端（`frontend/`）

| 项目 | 版本/说明 |
|------|-----------|
| Vue | ^3.5.39 |
| 构建工具 | Vite ^8.1.1 |
| UI 库 | Ant Design Vue ^4.2.6 + @ant-design/icons-vue |
| HTTP 库 | axios ^1.18.1 |
| 状态管理 | Pinia ^3.0.4 |
| 路由 | vue-router ^5.1.0（Hash 模式） |
| 图表 | echarts ^6.1.0 + vue-echarts |
| 图像裁剪 | cropperjs ^1.6.2 |
| 样式 | Tailwind CSS ^3.4.19 |
| 语言 | TypeScript |

---

## 二、通用约定

### 2.1 API 前缀机制

后端通过自定义注解 `@ApiPrefix`（默认前缀 `/api`）+ `ApiPrefixRequestMappingHandlerMapping` 自动为 Controller 类拼接 `/api` 前缀。

- 标注 `@ApiPrefix` 的 Controller，其 `@RequestMapping` 路径自动加 `/api`。例：`@ApiPrefix` + 类级 `@RequestMapping("/user")` + 方法 `@RequestMapping("/list")` → 实际路径 `/api/user/list`。
- **例外**：`HelloController` 未标注 `@ApiPrefix`，其 `/` 与 `/api/hello` 为硬编码原样路径。
- 前端 axios 实例 `baseURL = '/api'`，因此前端调用时路径**不带** `/api` 前缀（如 `request.get('/user/list')`）。

### 2.2 统一响应体 `R`

**所有 `/api/**` 接口**（除 SSE 流式接口外）必须返回统一响应体 `R`：

```json
{
  "rtData": {},
  "rtState": true,
  "rtMsg": ""
}
```

| 字段 | 类型 | 含义 |
|------|------|------|
| `rtData` | Object | 返回数据。成功时有值（无数据时为 `null`），失败时为 `null` |
| `rtState` | boolean | `true` 成功，`false` 失败 |
| `rtMsg` | String | 提示信息。成功时为 `""`，失败时为错误描述 |

**构造方式**：
- 成功带数据：`R.ok(data)` → `rtState=true, rtData=data, rtMsg=""`
- 成功无数据：`R.ok(null)` → `rtState=true, rtData=null, rtMsg=""`
- 失败：`R.fail("错误信息")` → `rtState=false, rtData=null, rtMsg="错误信息"`

### 2.3 HTTP 方法约定

后端所有接口均使用 `@RequestMapping` **未指定 method**，理论上接受所有 HTTP 方法。为统一行为，约定：

| 参数位置 | HTTP 方法 | 前端调用 |
|----------|-----------|----------|
| 查询参数 `@RequestParam` | GET | `request.get(url, { params })` |
| 请求体 `@RequestBody` | POST | `request.post(url, data)` |
| 文件上传 `MultipartFile` | POST（multipart/form-data） | `FormData` 提交 |

### 2.4 命名约定

- **数据库字段**：蛇形命名（snake_case），如 `user_id`、`login_name`、`del_flag`。
- **Java 实体字段**：驼峰命名（camelCase），如 `userId`、`loginName`、`delFlag`（由 `BeanPropertyRowMapper` 自动映射）。
- **前端 TS 类型字段**：与 Java 实体一致，驼峰命名。
- **JSON 传输字段**：与 Java 实体一致，驼峰命名。
- **逻辑删除标志**：`del_flag` 字符串 `"0"`=存在 / `"2"`=删除；`del_status` 整数 `1`=正常 / `0`=已删除（order 表特例）。

### 2.5 时间字段约定

所有时间字段（`createTime`/`updateTime`/`addTime`/`createdAt`/`updatedAt`）在 Java 实体中均为 **String 类型**，由 MySQL `DATETIME` 转换，格式通常为 `yyyy-MM-dd HH:mm:ss`。前端按字符串处理，不做 Date 转换。

---

## 三、认证与授权契约

### 3.1 登录流程

```
前端 LoginView → POST /api/auth/login {loginName, password}
                              ↓
后端 AuthController.login:
  1. 按 loginName 查询 sys_user（del_flag='0'）
  2. 校验 status，"1"=停用则拒绝
  3. 密码校验：MD5(loginName + password + salt) 与库中 password 比对
  4. 生成 JWT token 返回
                              ↓
前端存储 token 到 localStorage，后续请求注入 Authorization 头
```

**登录请求**：`POST /api/auth/login`（白名单，无需 token）

请求体：
```json
{ "loginName": "admin", "password": "admin123" }
```

成功响应 `rtData`：
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "loginName": "admin",
  "userName": "若依",
  "avatar": "/upload/xxx.png"
}
```

失败响应：`rtState=false`，`rtMsg` 为 `"账号不存在"` / `"账号已停用"` / `"密码错误"` / `"账号或密码不能为空"`。

### 3.2 JWT Token 约定

| 属性 | 值 |
|------|-----|
| 签名算法 | HS256 |
| 密钥 | 硬编码 `SpringBootDemo2024SecretKeyForJWT` |
| 过期时间 | 8 小时（8 × 3600 × 1000 ms） |
| 载荷 subject | `loginName`（登录账号） |
| 标准字段 | `iat`（签发时间）、`exp`（过期时间） |

> **安全提示**：密钥硬编码在源码中，生产环境应迁移到配置或环境变量（见[改进建议](#十三已知问题与改进建议)）。

### 3.3 Token 校验（TokenFilter）

`TokenFilter`（`OncePerRequestFilter`）拦截所有请求，校验逻辑：

1. 取 `request.getServletPath()` 判断是否在白名单，是则放行。
2. OPTIONS 请求直接放行（CORS 预检）。
3. 取请求头 `Authorization`，必须以 `"Bearer "` 开头。
4. 截取 token（第 7 个字符起），调 `JwtTokenUtil.validateToken(token)` 验证签名与过期。
5. **验证通过**：`request.setAttribute("currentUser", loginName)`，继续过滤器链。
6. **验证失败/无 token**：返回 HTTP 401 + 响应体 `R.fail("未登录，请先登录")` 或 `R.fail("token 已过期或无效")`。

**Controller 获取当前用户**：
```java
// currentUser 是 loginName（登录账号），不是 userId 也不是 userName
String loginName = (String) request.getAttribute("currentUser");
User user = userDao.findByAccount(loginName);
```

### 3.4 白名单（无需 token）

| 路径 | 匹配方式 | 说明 |
|------|----------|------|
| `/` | 精确匹配 | HelloController 根路径 |
| `/api/auth/login` | 精确匹配 | 登录接口 |
| `/upload/**` | 前缀匹配 | 静态资源（图片访问） |
| `/api/order/createTable` | 前缀匹配 | 订单建表（临时） |
| `/api/chat/createTables` | 前缀匹配 | 聊天建表 |
| `/api/post/createTable` | 前缀匹配 | 岗位建表 |
| 所有 OPTIONS 请求 | 方法匹配 | CORS 预检 |

**除上述外，所有 `/api/**` 路径均需要 token**（含 `/api/hello`）。

### 3.5 密码加密算法

- **加密**：`MD5(loginName + password + salt)`，使用 Spring 的 `DigestUtils.md5DigestAsHex`。
- **盐生成**：6 位随机数字（`SecureRandom`，0-9）。
- **校验**：`encrypt(loginName, password, salt).equals(dbPassword)`。
- **新增用户默认密码**：未传 `password` 时默认 `"123456"`。
- **新增用户默认状态**：`"0"`（正常）。

### 3.6 前端 401 处理

前端 axios 响应拦截器收到 HTTP 401 时：
1. 调用 `authStore.redirectToLogin({ expired: true })`。
2. 该方法**并发去重**（同一时间窗口多个 401 只处理一次），弹 `message.warning('登录已过期，请重新登录')`。
3. 清除认证状态、重置聊天 store。
4. 跳转 `/login?redirect=<原页面路径>`，重登后回到原处。

---

## 四、CORS 与代理约定

### 4.1 后端 CORS 配置

后端 `CorsFilter` 手动实现（Spring Boot 1.2.x 无内置 CORS 支持）：

| 配置项 | 值 |
|--------|-----|
| `Access-Control-Allow-Origin` | `*`（允许所有源） |
| `Access-Control-Allow-Methods` | `GET, POST, PUT, DELETE, OPTIONS` |
| `Access-Control-Allow-Headers` | `Content-Type, Authorization, X-Requested-With` |
| `Access-Control-Expose-Headers` | `Authorization` |
| `Access-Control-Max-Age` | `3600`（秒） |
| `Access-Control-Allow-Credentials` | **未设置**（因 Origin 为 `*`，不可带凭证） |

OPTIONS 预检请求在 CorsFilter 中直接返回 200。

### 4.2 前端开发代理（Vite）

`vite.config.ts` 配置开发服务器代理，解决开发环境跨域：

| 前缀 | 目标 | 说明 |
|------|------|------|
| `/api` | `http://localhost:8081`（可由 `VITE_API_BASE_URL` 覆盖） | API 请求代理 |
| `/upload` | `http://localhost:8081` | 静态资源代理 |

生产环境由 nginx 等反向代理处理，前端 `baseURL='/api'` 无需修改。

---

## 五、分页契约

### 5.1 入参字段

| 模块 | 页码字段 | 页大小字段 | 默认页码 | 默认页大小 |
|------|----------|------------|----------|------------|
| User | `page` | `pageSize` | 1 | **3** |
| Role | `page` | `pageSize` | 1 | 10 |
| Order | `page` | `pageSize` | 1 | 10 |

**统一使用 `page` + `pageSize`**（非 `page`/`size`，非 `pageNum`/`pageSize`）。均为查询参数，`page` 从 1 开始。

### 5.2 返回结构

统一返回 `R.ok({ list, total })`：

```json
{
  "rtData": {
    "list": [ /* 当前页数据数组 */ ],
    "total": 100
  },
  "rtState": true,
  "rtMsg": ""
}
```

**统一使用 `list` + `total`**（非 `rows`/`total`，非 `records`/`total`）。无 `totalPages`、`currentPage` 等额外字段，前端自行计算总页数 `Math.ceil(total / pageSize)`。

### 5.3 SQL 分页

`LIMIT ? OFFSET ?`，其中 `offset = (page - 1) * pageSize`。

---

## 六、文件上传契约

### 6.1 上传接口

`POST /api/file/upload`（需认证，multipart/form-data）

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `file` | MultipartFile | 是 | 图片文件 |
| `module` | String | 否 | 模块分类，默认 `"common"` |

**格式限制**：仅允许 `jpg, jpeg, png, gif, bmp, webp, svg`，其他格式返回 `R.fail("不支持的图片格式: xxx")`。

**大小限制**：单文件 10MB（`upload.maxFileSize`），超限由 `GlobalExceptionHandler` 捕获返回 `R.fail("文件大小超出限制")`。

成功响应 `rtData`：
```json
{
  "fileName": "202608041200001234_xxx.png",
  "filePath": "common/2026/202608041200001234_xxx.png",
  "url": "/upload/common/2026/202608041200001234_xxx.png"
}
```

### 6.2 存储与访问映射

- **存储路径**：`{upload.path}/{module}/{year}/{唯一文件名}`
  - `upload.path` = `/Users/wsx/spring-boot-demo/images`
  - 文件名规则：`yyyyMMddHHmmssSSS` + 随机数 + `_` + 原始文件名（去空格）
- **访问 URL**：`/upload/{module}/{year}/{fileName}`
- **静态资源映射**：`/upload/**` → 本地目录 `/Users/wsx/spring-boot-demo/images/`（`AppConfig` 注册）。
- `/upload/**` 在 TokenFilter 白名单中，**无需认证即可访问图片**。

### 6.3 前端调用约定

前端上传时构造 `FormData`：
```ts
const formData = new FormData()
formData.append('file', file)
formData.append('module', 'avatar') // 可选
// 单独用 axios 实例或覆盖 Content-Type 为 multipart/form-data
```

> 注意：axios 实例默认 `Content-Type: application/json`，上传时需让浏览器自动设置 `multipart/form-data` 边界，不要手动写死该头。

---

## 七、接口清单

> 以下所有路径均为完整路径（含 `/api` 前缀）。认证列「是」表示需要 token。除特别说明，所有接口返回 `R` 包装。

### 7.1 Hello 模块

| 方法 | 路径 | 认证 | 参数 | rtData | 说明 |
|------|------|------|------|--------|------|
| ANY | `/` | 否 | 无 | 纯文本 `Hello World`（**非 R 格式**） | 根路径测试 |
| ANY | `/api/hello` | 是 | 无 | `"你好 世界！"`（String） | JSON 问候接口 |

### 7.2 Auth 模块（`/api/auth`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/auth/login` | 否 | Body: `{loginName, password}` | `{token, loginName, userName, avatar}` | 登录 |
| ANY | `/api/auth/updateAvatar` | 是 | Body: `{avatar: String}` | `{avatar: String}` | 更新当前用户头像 |

### 7.3 User 模块（`/api/user`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/user/list` | 是 | Query: `page`(默认1), `pageSize`(默认3), `userName?`, `sex?`, `loginName?` | `{list: UserInfo[], total: number}` | 分页查询用户 |
| ANY | `/api/user/update` | 是 | Body: `UpdateUserParams`（`userId` 必填，仅更新 `userName`/`sex`/`avatar`） | `null` | 编辑用户 |
| ANY | `/api/user/create` | 是 | Body: `CreateUserParams`（`password` 可选，默认 `123456`） | `null` | 新增用户 |
| ANY | `/api/user/delete` | 是 | Body: `{userId: number}` | `null` | 逻辑删除（`del_flag='2'`） |

### 7.4 Role 模块（`/api/role`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/role/list` | 是 | Query: `page`(默认1), `pageSize`(默认10), `roleKey?`, `roleName?` | `{list: RoleInfo[], total: number}` | 分页查询角色 |
| ANY | `/api/role/create` | 是 | Body: `CreateRoleParams` | `null` | 新增角色 |
| ANY | `/api/role/update` | 是 | Body: `UpdateRoleParams`（`roleId` 必填） | `null` | 编辑角色 |
| ANY | `/api/role/delete` | 是 | Body: `{roleId: number}` | `null` | 逻辑删除（`del_flag='2'`） |

### 7.5 Menu 模块（`/api/menu`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/menu/list` | 是 | 无 | `MenuInfo[]`（全部菜单，**非分页**） | 查询所有菜单 |
| ANY | `/api/menu/getById` | 是 | Body: `{menuId: number}` | `MenuInfo` | 查询单条菜单 |
| ANY | `/api/menu/create` | 是 | Body: `CreateMenuParams` | `null` | 新增菜单 |
| ANY | `/api/menu/update` | 是 | Body: `UpdateMenuParams`（`menuId` 必填） | `null` | 编辑菜单 |
| ANY | `/api/menu/delete` | 是 | Body: `{menuId: number}` | `null`（有子菜单时返回 fail） | 删除菜单（**物理删除**） |

> **注意**：Menu 删除为物理删除（`DELETE FROM`），且删除前检查子菜单，有子菜单时返回 `R.fail("存在子菜单，无法删除")`。

### 7.6 Order 模块（`/api/order`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/order/createTable` | **否** | 无 | `"订单表创建成功"`（String） | 临时建表 DDL |
| ANY | `/api/order/list` | 是 | Query: `page`(默认1), `pageSize`(默认10), `orderStatus?`, `keyword?` | `{list: OrderInfo[], total: number}` | 分页查询订单 |
| ANY | `/api/order/stats` | 是 | 无 | `OrderStats` | 订单统计 |
| ANY | `/api/order/create` | 是 | Body: `CreateOrderParams`（无需传 `orderNo`，后端生成） | `null` | 新增订单 |
| ANY | `/api/order/update` | 是 | Body: `UpdateOrderParams`（`id` 必填） | `null` | 编辑订单 |
| ANY | `/api/order/delete` | 是 | Body: `{id: number}` | `null` | 逻辑删除（`del_status=0`） |

**订单号生成规则**：`yyyyMMdd` + 4 位随机数字。

**`OrderStats` 字段**：`all`(全部)、`pending`=待付款(status=0)、`inProgress`=进行中(status=1)、`completed`=已完成(status=2)。**注意：未统计 status=3（已取消）**。

### 7.7 Chat 模块（`/api/chat`）

| 方法 | 路径 | 认证 | 请求参数 | rtData / 响应 | 说明 |
|------|------|------|----------|----------------|------|
| ANY | `/api/chat/createTables` | **否** | 无 | `"聊天表创建成功"`（String） | 建表 |
| ANY | `/api/chat/sessions` | 是 | 无（从 token 取当前用户） | `ChatSession[]` | 获取当前用户会话列表 |
| ANY | `/api/chat/session/create` | 是 | Body: `{title?: string}` | `ChatSession` | 创建新会话 |
| ANY | `/api/chat/session/delete` | 是 | Body: `{sessionId: number}` | `null` | 删除会话（验证归属，级联删消息） |
| ANY | `/api/chat/session/updateTitle` | 是 | Body: `{sessionId, title}` | `null` | 更新会话标题 |
| ANY | `/api/chat/session/updateIcon` | 是 | Body: `{sessionId, iconUrl}` | `null` | 更新会话头像 |
| ANY | `/api/chat/messages` | 是 | Query: `sessionId`(必填) | `ChatMessage[]` | 获取会话消息列表 |
| ANY | `/api/chat/send` | 是 | Body: `{sessionId, message}` | **SSE 流**（非 R 格式，见[第十章](#十sse-流式契约聊天)） | 发送消息并流式返回 AI 回复 |

### 7.8 Dashboard 模块（`/api/dashboard`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/dashboard/stats` | 是 | 无 | `DashboardStats` | Dashboard 统计数据 |

`DashboardStats` 结构：
```json
{
  "userCount": 100,              // 真实统计：sys_user WHERE del_flag='0'
  "orderCount": 50,              // 真实统计：order WHERE del_status=1
  "visitTrend": [                // 模拟：近7天访问量
    {"date": "2021/4/25", "value": 30000}
  ],
  "dailyAvgVisit": 57044,        // 模拟：日均访问量
  "authTrend": [                 // 模拟：24小时授权数分布
    {"hour": "0时", "value": 8}
  ],
  "authCount": 9970,             // 模拟：授权总数
  "techStack": [                 // 固定：技术栈版本
    {"name": "vue", "version": "^3.5.29", "upgrade": "up"}
  ],
  "openProjects": [              // 固定：开源项目列表
    {"name": "vue-admin-better", "desc": "...", "color": "teal"}
  ],
  "quickActions": [              // 固定：快捷功能
    {"label": "随机换肤", "key": "random-skin", "color": "#f56c6c", "badge": "true"}
  ]
}
```

> **注意**：仅 `userCount` 和 `orderCount` 为真实数据库查询（表不存在时返回 0），其余为硬编码模拟数据。

### 7.9 File 模块（`/api/file`）

见[第六章 文件上传契约](#六文件上传契约)。

### 7.10 Post 模块（`/api/post`）

| 方法 | 路径 | 认证 | 请求参数 | rtData | 说明 |
|------|------|------|----------|--------|------|
| ANY | `/api/post/list` | 是 | Query: `page`(默认1), `pageSize`(默认10), `postCode?`, `postName?`, `status?` | `{list: PostInfo[], total: number}` | 分页查询岗位 |
| ANY | `/api/post/create` | 是 | Body: `CreatePostParams` | `null` | 新增岗位 |
| ANY | `/api/post/update` | 是 | Body: `UpdatePostParams`（`postId` 必填） | `null` | 编辑岗位 |
| ANY | `/api/post/delete` | 是 | Body: `{postId: number}` | `null` | 物理删除（`DELETE FROM`） |
| ANY | `/api/post/createTable` | **否** | 无 | `"岗位表创建成功"`（String） | 建表 DDL（`IF NOT EXISTS`） |

> `postCode`/`postName` 用 LIKE 模糊查询，`status` 精确匹配；排序 `post_sort ASC, post_id DESC`。前端 `PostInfo`/`CreatePostParams`/`UpdatePostParams` 定义在 `types/models.d.ts`。

---

## 八、数据模型契约

数据库：`wsx`（MySQL `192.168.0.106:3306`）。下方每张表同时给出「数据库列」与「对应 TS 类型」。

### 8.1 `sys_user` 表（用户）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `user_id` | BIGINT PK AUTO_INCREMENT | `userId: number` | 主键 |
| `login_name` | VARCHAR | `loginName: string` | 登录账号 |
| `user_name` | VARCHAR | `userName: string` | 姓名 |
| `sex` | VARCHAR | `sex: string` | 性别 |
| `avatar` | VARCHAR | `avatar: string` | 头像 URL |
| `password` | VARCHAR | `password: string` | 密码 `MD5(loginName+password+salt)` |
| `salt` | VARCHAR | `salt: string` | 盐（6 位随机数字） |
| `status` | CHAR | `status: string` | 状态：`0`=正常 `1`=停用 |
| `del_flag` | CHAR | `delFlag: string` | 逻辑删除：`0`=存在 `2`=删除 |
| `create_time` | DATETIME | `createTime: string` | 创建时间 |
| `update_time` | DATETIME | `updateTime: string` | 更新时间 |

查询条件 `WHERE del_flag = '0'`，排序 `ORDER BY user_id DESC`。

### 8.2 `sys_role` 表（角色）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `role_id` | BIGINT PK | `roleId: number` | 主键 |
| `role_name` | VARCHAR | `roleName: string` | 角色名称 |
| `role_key` | VARCHAR | `roleKey: string` | 角色权限字符 |
| `role_sort` | INT | `roleSort: number` | 显示顺序 |
| `data_scope` | CHAR | `dataScope: string` | 数据范围：`1`=全部 `2`=自定义 |
| `status` | CHAR | `status: string` | 状态：`0`=正常 `1`=停用 |
| `del_flag` | CHAR | `delFlag: string` | 逻辑删除：`0`=正常 `2`=已删除 |
| `create_by` | VARCHAR | `createBy: string` | 创建者 |
| `create_time` | DATETIME | `createTime: string` | 创建时间 |
| `update_by` | VARCHAR | `updateBy: string` | 更新者 |
| `update_time` | DATETIME | `updateTime: string` | 更新时间 |
| `remark` | VARCHAR | `remark: string` | 备注 |

查询条件 `WHERE del_flag = '0'`，排序 `ORDER BY role_sort ASC, role_id DESC`。

### 8.3 `sys_menu` 表（菜单）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `menu_id` | BIGINT PK | `menuId: number` | 主键 |
| `menu_name` | VARCHAR | `menuName: string` | 菜单名称 |
| `parent_id` | BIGINT | `parentId: number` | 父菜单 ID（`0`=顶级） |
| `order_num` | INT | `orderNum: number` | 显示顺序 |
| `url` | VARCHAR | `url: string` | 路由地址 |
| `target` | VARCHAR | `target: string` | 打开方式 |
| `menu_type` | CHAR | `menuType: 'M'\|'C'\|'F'` | 类型：`M`=目录 `C`=菜单 `F`=按钮 |
| `visible` | CHAR | `visible: string` | 是否显示：`0`=显示 `1`=隐藏 |
| `is_refresh` | CHAR | `isRefresh: string` | 是否刷新：`1`=是 |
| `perms` | VARCHAR | `perms: string` | 权限标识 |
| `icon` | VARCHAR | `icon: string` | 图标 |
| `create_by` / `create_time` / `update_by` / `update_time` / `remark` | — | 同名驼峰 | 审计字段 |

**无 `del_flag`**，全量查询，排序 `ORDER BY parent_id ASC, order_num ASC`。删除为**物理删除**。前端 `MenuInfo` 含可选 `children?: MenuInfo[]`（树形结构由前端组装）。

### 8.4 `order` 表（订单）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `id` | INT PK AUTO_INCREMENT | `id: number` | 主键 |
| `order_no` | VARCHAR(50) NOT NULL | `orderNo: string` | 订单号（后端生成） |
| `customer_name` | VARCHAR(100) NOT NULL | `customerName: string` | 客户名称 |
| `quantity` | INT DEFAULT 0 | `quantity: number` | 数量 |
| `unit_price` | DECIMAL(10,2) | `unitPrice: number` | 单价 |
| `total_amount` | DECIMAL(10,2) | `totalAmount: number` | 金额 |
| `order_status` | TINYINT DEFAULT 0 | `orderStatus: number` | 状态：`0`=待付款 `1`=进行中 `2`=已完成 `3`=已取消 |
| `payment_method` | VARCHAR(50) | `paymentMethod: string` | 付款方式 |
| `remark` | VARCHAR(500) | `remark: string` | 备注 |
| `add_time` | DATETIME | `addTime: string` | 创建时间 |
| `update_time` | DATETIME | `updateTime: string` | 更新时间 |
| `del_status` | TINYINT DEFAULT 1 | `delStatus: number` | 逻辑删除：`1`=正常 `0`=已删除 |

查询条件 `WHERE del_status = 1`，`keyword` 搜索 `order_no LIKE ? OR customer_name LIKE ?`，排序 `ORDER BY id DESC`。

### 8.5 `chat_session` 表（聊天会话）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `id` | INT PK AUTO_INCREMENT | `id: number` | 主键 |
| `user_sid` | INT NOT NULL | `userSid: number` | 用户 ID（= `sys_user.user_id`） |
| `title` | VARCHAR(100) DEFAULT '新的聊天' | `title: string` | 会话标题 |
| `ai_model` | VARCHAR(50) DEFAULT 'deepseek-chat' | `aiModel: string` | AI 模型 |
| `icon_url` | VARCHAR(500) | `iconUrl?: string` | 会话头像 URL |
| `created_at` | DATETIME | `createdAt: string` | 创建时间 |
| `updated_at` | DATETIME | `updatedAt: string` | 更新时间 |

按 `user_sid` 查询，排序 `ORDER BY updated_at DESC`。删除为**物理删除**（级联删除消息）。

> **遗留字段**：`user_sid` 值存储 `sys_user.user_id`，字段名保留旧称（未随重命名改动）。

### 8.6 `chat_message` 表（聊天消息）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `id` | INT PK AUTO_INCREMENT | `id: number` | 主键 |
| `session_id` | INT NOT NULL | `sessionId: number` | 会话 ID |
| `role` | VARCHAR(20) NOT NULL | `role: 'user'\|'assistant'` | 角色 |
| `content` | TEXT NOT NULL | `content: string` | 消息内容 |
| `created_at` | DATETIME | `createdAt: string` | 创建时间 |

字符集 `utf8mb4`（支持 Emoji）。按 `session_id` 查询，排序 `ORDER BY created_at ASC`（时间正序）。前端 `ChatMessage` 含可选 `_streaming?: boolean`（流式标记，不传后端）。

### 8.7 `sys_post` 表（岗位）

| 列名 | SQL 类型 | Java/TS 字段 | 含义 |
|------|----------|--------------|------|
| `post_id` | BIGINT PK AUTO_INCREMENT | `postId: number` | 主键 |
| `post_code` | VARCHAR(64) NOT NULL | `postCode: string` | 岗位编码 |
| `post_name` | VARCHAR(50) NOT NULL | `postName: string` | 岗位名称 |
| `post_sort` | INT DEFAULT 0 | `postSort: number` | 显示顺序 |
| `status` | CHAR(1) DEFAULT '0' | `status: string` | 状态：`0`=正常 `1`=停用 |
| `create_by` | VARCHAR(64) | `createBy: string` | 创建者 |
| `create_time` | DATETIME | `createTime: string` | 创建时间 |
| `update_by` | VARCHAR(64) | `updateBy: string` | 更新者 |
| `update_time` | DATETIME | `updateTime: string` | 更新时间 |
| `remark` | VARCHAR(500) | `remark: string` | 备注 |

排序 `ORDER BY post_sort ASC, post_id DESC`；删除为物理删除（`DELETE FROM`，RuoYi 原版 `sys_post` 无 `del_flag`）。建表 DDL 由 `/api/post/createTable` 接口提供（`IF NOT EXISTS`，与 RuoYi 原版一致）。

---

## 九、前端开发约定

### 9.1 axios 实例（`api/request.ts`）

| 配置 | 值 |
|------|-----|
| `baseURL` | `/api` |
| `timeout` | 10000（10 秒，超时触发 `ECONNABORTED`） |
| 默认 `Content-Type` | `application/json` |

**请求拦截器**：从 `localStorage` 读取 token，注入 `Authorization: Bearer <token>` 头。

**响应拦截器**（核心解包逻辑）：
- HTTP 2xx：取 `response.data` 为 `ApiResponse`。
  - `rtState === true` → **返回整个 `ApiResponse` 对象**（调用方用 `res.rtData` 取数据），**不解包 rtData**。
  - `rtState === false` → `message.error(rtMsg)` 并 `reject`。
- HTTP 非 2xx：按状态码分类提示。

| HTTP 状态码 | 前端处理 |
|-------------|----------|
| 400 | `message.error('请求参数错误')` |
| 401 | `authStore.redirectToLogin({ expired: true })`（去重 + 跳登录） |
| 403 | `message.error('没有权限访问')` |
| 404 | `message.error('请求的资源不存在')` |
| 500 | `message.error('服务器内部错误，请稍后重试')` |
| 其他 | `message.error('请求失败，状态码：${status}')` |
| `ECONNABORTED` | `message.error('请求超时，请重试')` |
| 无 response | `message.error('网络连接异常，请检查网络')` |

> **关键约定**：前端 API 函数返回 `Promise<ApiResponse<T>>`，调用方通过 `res.rtData` 取实际数据。拦截器不做 `rtData` 解包，保留 `rtState`/`rtMsg` 供调用方判断。

### 9.2 API 封装规范

所有接口封装在 `src/api/<模块>/index.ts`，统一从 `@/api/request` 引入实例。规范：
- 查询类用 `request.get(url, { params })`。
- 写入类用 `request.post(url, data)`。
- 函数必须有 JSDoc 注释说明用途。
- 返回类型必须显式标注 `Promise<ApiResponse<T>>`。

示例：
```ts
/** 获取用户分页列表，支持姓名、性别和账号过滤 */
export function getUserList(
    page: number = 1,
    pageSize: number = 3,
    userName?: string,
    sex?: string | null,
    loginName?: string
): Promise<ApiResponse<PaginatedData<UserInfo>>> {
    const params: Record<string, unknown> = { page, pageSize }
    if (userName) params.userName = userName
    if (sex != null) params.sex = sex
    if (loginName) params.loginName = loginName
    return request.get('/user/list', { params })
}
```

### 9.3 类型声明

- 通用类型（`ApiResponse<T>`、`PaginatedData<T>`）定义在 `src/types/api.d.ts`，**全局 ambient 声明，无需 import**。
- 领域模型（`UserInfo`、`OrderInfo`、`RoleInfo`、`MenuInfo`、`ChatSession` 等）定义在 `src/types/models.d.ts`，全局可用。
- 新增实体须同步在 `models.d.ts` 增加对应 interface，字段与后端 Java 实体一一对应。

### 9.4 状态管理（Pinia）

**认证 store（`stores/auth`）**：
- 状态字段：`token`、`loginName`、`userName`、`avatar`。
- 持久化：`token`、`avatar` 存 `localStorage`（键名见 `STORAGE_KEYS`），`loginName`/`userName` 仅内存（刷新后需重登或重新拉取）。
- 计算属性：`isLoggedIn = !!token`。
- 核心方法：`login()`、`logout()`、`setAvatar()`、`redirectToLogin()`、`clearAuth()`。
- **401 并发去重**：`handlingAuthError` 标志保证同一窗口多个 401 只处理一次，`login` 成功或手动退出时复位。

**localStorage 键名常量（`constants/storageKeys.ts`）**：

| 键 | 值 | 说明 |
|----|----|------|
| `TOKEN` | `'token'` | JWT token |
| `AVATAR` | `'avatar'` | 用户头像 URL |
| `THEME_DARK` | `'theme-dark'` | 暗黑模式开关 |
| `THEME_PRIMARY` | `'theme-primary'` | 主题主色 |
| `SIDEBAR_COLLAPSED` | `'sidebar-collapsed'` | 侧边栏折叠状态 |

### 9.5 路由守卫（`router/index.ts`）

- 路由模式：`createWebHashHistory`（Hash 模式）。
- 受保护路由在父路由 `meta.requiresAuth = true`（默认受保护）。
- `beforeEach` 守卫逻辑：
  - 目标路由 `requiresAuth !== false` 且无 token → 跳 `/login?redirect=<原路径>`。
  - 已登录访问 `/login` → 跳 `/`。
  - 其他放行。

| 路由 path | name | 说明 |
|-----------|------|------|
| `/` | Dashboard | 仪表盘 |
| `/users` | Users | 用户管理 |
| `/orders` | Orders | 订单管理 |
| `/chat` | Chat | AI 聊天 |
| `/sys/role` | SysRole | 角色管理 |
| `/sys/menu` | SysMenu | 菜单管理 |
| `/about` | About | 关于 |
| `/login` | Login | 登录（`requiresAuth: false`） |

### 9.6 Composables

**`useRequest`**：封装 loading 状态。`loading` 初始为 `true`（适配首屏即请求），`execute(asyncFn)` 自动管理 loading 开关，错误向上抛出由调用方处理。

**`usePagination`**：封装分页状态 `page`/`pageSize`/`total`，生成 `paginationConfig` 直接绑定 `a-table :pagination`。默认 `pageSize=10`，`showSizeChanger=false`，`showTotal` 文本 `共 ${t} 条`。`resetPage()` 重置到第一页（搜索/筛选变化时调用）。

### 9.7 缩进与代码风格

- 前端统一 **4 空格缩进**。
- 使用有意义的变量名和函数名。
- 新增函数必须有注释（JSDoc `/** */` 格式）。

---

## 十、SSE 流式契约（聊天）

聊天发送消息接口 `/api/chat/send` 是**唯一不返回 `R` 格式**的接口，改为 SSE 事件流。

### 10.1 响应头

```
Content-Type: text/event-stream;charset=UTF-8
Cache-Control: no-cache
Connection: keep-alive
X-Accel-Buffering: no
```

### 10.2 事件格式

每个事件：`data: {JSON}\n\n`，JSON 结构：

| `type` | 载荷 | 说明 |
|--------|------|------|
| `start` | `{"type":"start","sessionId":<id>}` | 流开始 |
| `chunk` | `{"type":"chunk","content":"<片段>"}` | AI 回复片段（多次推送） |
| `done` | `{"type":"done","sessionId":<id>}` | 流正常结束 |
| `error` | `{"type":"error","message":"<错误信息>"}` | 错误 |

### 10.3 前端调用约定

前端 **不使用 axios**（axios 不支持流式读取），改用原生 `fetch` + `ReadableStream`：

```ts
fetch('/api/chat/send', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}` // 手动注入 token，绕过 axios 拦截器
    },
    body: JSON.stringify({ sessionId, message })
})
```

逐行读取 `data: ` 前缀，`JSON.parse` 后按 `type` 分发到 `callbacks.onStart/onChunk/onDone/onError`。

### 10.4 后端业务逻辑

1. 保存用户消息到 `chat_message`。
2. 查询会话历史消息。
3. 组装 OpenAI messages 数组（含 system prompt：`"你是一个有帮助的AI助手，请用简洁、自然的中文回复用户。"`）。
4. 调用 DeepSeek 流式 API，逐 chunk 回传前端。
5. 流结束后保存完整 AI 回复。
6. 首次对话（`history.size() <= 2`）自动更新会话标题为用户消息前 20 字。

> **依赖**：DeepSeek API Key 从环境变量 `DEEPSEEK_API_KEY` 读取，未配置时 `chat/send` 调用失败。

---

## 十一、错误处理契约

### 11.1 后端全局异常处理（`GlobalExceptionHandler`）

| 捕获异常 | 响应 |
|----------|------|
| `MultipartException`（含 `FileSizeLimitExceededException`） | `R.fail("文件大小超出限制")` |
| `MultipartException`（其他） | `R.fail("文件上传失败: " + 根因消息)` |

> **缺口**：全局异常处理**仅捕获 `MultipartException`**，未捕获 `NullPointerException`、SQL 异常等。其他异常会返回 Spring Boot 默认错误页（HTTP 500）。改进建议见[第十三章](#十三已知问题与改进建议)。

### 11.2 业务错误

业务校验失败统一用 `R.fail(msg)` 返回（HTTP 200，`rtState=false`），前端响应拦截器弹 `message.error(rtMsg)`。例如：
- 登录失败：`R.fail("账号不存在")` / `R.fail("密码错误")`
- 删除菜单有子菜单：`R.fail("存在子菜单，无法删除")`
- 文件格式不支持：`R.fail("不支持的图片格式: xxx")`

### 11.3 鉴权错误

- 未登录/token 无效：HTTP **401** + `R.fail("未登录，请先登录")` / `R.fail("token 已过期或无效")`。
- 前端 401 统一走 `redirectToLogin({ expired: true })`。

### 11.4 前端错误处理分层

1. **拦截器层**（`request.ts`）：统一处理 HTTP 状态码、业务 `rtState=false`、超时、网络异常，弹 `message`。
2. **调用方层**（页面/composable）：`try/catch` 捕获 reject，做页面级补充提示（如「获取列表失败」）。`useRequest` 不吞错误。

---

## 十二、协作流程与规范

### 12.1 新增接口协作流程

1. **后端先行**：编写 Controller + Service + DAO，确定路径、方法、入参、出参。
2. **更新契约**：在本文件「接口清单」与「数据模型」章节补全接口与字段定义。
3. **通知前端**：将接口路径、请求/响应字段同步给前端。
4. **前端实现**：在 `api/<模块>/index.ts` 增加封装函数，在 `types/models.d.ts` 增加类型，返回类型标注 `Promise<ApiResponse<T>>`。
5. **联调**：前端按约定调用，验证 `rtState`/`rtData`/`rtMsg` 解包正常。
6. **回归**：`cd demo && mvn test`、`cd frontend && npm run build` 均通过。

### 12.2 字段变更规范

- 后端实体字段变更（增删改）→ 同步更新 `models.d.ts` 对应 interface → 同步更新本契约数据模型章节。
- 数据库表结构变更（DDL）→ 在本契约标注，并更新 `application.yml`/建表接口说明。
- **禁止**前端硬编码后端字段名以外的别名；**禁止**后端返回与契约不一致的字段。

### 12.3 编码规范

- **Java**：新增方法用 `//` 行注释说明用途；优先手写 getter/setter（Lombok 注解处理器不可靠）；接口统一返回 `R`。
- **JavaScript/TypeScript**：新增函数用 JSDoc `/** */` 注释；4 空格缩进；有意义命名。
- **SQL**：表名蛇形，逻辑删除用 `del_flag`/`del_status`，分页用 `LIMIT ? OFFSET ?`。

### 12.4 Git 提交规范

- 使用 Conventional Commits 格式（如 `feat(user): 新增用户导入接口`、`fix(auth): 修复 token 校验空指针`）。
- 提交信息末尾**不追加** `Co-Authored-By` 行。

---

## 十三、已知问题与改进建议

| # | 问题 | 现状 | 建议 |
|---|------|------|------|
| 1 | 用户列表返回 `password`/`salt` | 沿用原行为，敏感字段外泄 | 后续用 DTO 过滤敏感字段，列表接口不返回密码相关字段 |
| 2 | JWT 密钥硬编码 | `SpringBootDemo2024SecretKeyForJWT` 写死在源码 | 迁移到 `application.yml` 或环境变量 |
| 3 | 全局异常处理不全 | 仅捕获 `MultipartException`，其他异常返回默认 500 页 | 增加 `@ExceptionHandler(Exception.class)` 兜底，统一返回 `R.fail` |
| 4 | Order stats 缺 status=3 | 仅统计 0/1/2，未统计已取消(3) | 增加 `cancelled` 字段统计 status=3 |
| 5 | Menu 物理删除 | 与其他模块逻辑删除不一致，删除不可恢复 | 评估改为逻辑删除（加 `del_flag`）或保留但明确文档警示 |
| 6 | HTTP 方法未限制 | `@RequestMapping` 未指定 method，接受所有方法 | 按语义明确指定 `@GetMapping`/`@PostMapping`，避免安全风险 |
| 7 | CORS 允许所有源 | `Allow-Origin: *`，不带凭证 | 生产环境收敛为具体前端域名 |
| 8 | 临时建表接口暴露 | `/api/order/createTable`、`/api/chat/createTables` 在白名单 | 生产环境移除或加权限保护 |
| 9 | `user_sid` 字段名遗留 | `chat_session.user_sid` 实存 `user_id` | 评估改名为 `user_id` 并同步 DAO（见 `rename-user-to-sys-user.md` 决策 D3） |
| 10 | DeepSeek Key 配置 | 依赖环境变量，未配置即失败 | 启动时检测并给出明确日志提示 |

---

## 附录：核心文件索引

### 后端（`demo/src/main/java/com/example/`）

| 职责 | 文件 |
|------|------|
| 启动入口 | `App.java` |
| 统一响应体 | `common/R.java` |
| API 前缀机制 | `common/ApiPrefix.java`、`common/ApiPrefixRequestMappingHandlerMapping.java` |
| 鉴权过滤器 | `filter/TokenFilter.java` |
| 跨域过滤器 | `filter/CorsFilter.java` |
| 全局异常 | `exception/GlobalExceptionHandler.java` |
| JWT 工具 | `utils/JwtTokenUtil.java` |
| 密码工具 | `utils/PasswordUtils.java` |
| 文件上传 | `utils/FileUploadUtil.java`、`controller/FileController.java` |
| 静态资源映射 | `config/AppConfig.java`、`config/WebMvcConfig.java`、`config/UploadProperties.java` |
| 控制器 | `controller/*.java`（Hello/Auth/User/Role/Menu/Order/Chat/Dashboard/File） |
| 实体 | `entity/*.java`（User/Role/Menu/Order/ChatSession/ChatMessage） |
| 数据访问 | `dao/*.java`（JdbcTemplate 手写 SQL） |

### 前端（`frontend/src/`）

| 职责 | 文件 |
|------|------|
| axios 实例 | `api/request.ts` |
| API 封装 | `api/<模块>/index.ts`、`api/sys/menu.ts` |
| 通用类型 | `types/api.d.ts` |
| 领域模型 | `types/models.d.ts` |
| 认证状态 | `stores/auth/index.ts` |
| 聊天状态 | `stores/chat/index.ts` |
| 路由 | `router/index.ts` |
| 请求封装 | `composables/useRequest.ts`、`composables/usePagination.ts` |
| 存储键常量 | `constants/storageKeys.ts` |
| Vite 配置 | `vite.config.ts` |
