# RuoYi-Vue-Plus 6.X 新人学习指南

> 面向初次接触本项目的新手，以"从入门到能自己写功能"为线索，一步步带你读懂这套后台管理系统。

---

## 目录

1. [这个项目是什么](#一这个项目是什么)
2. [技术栈全景图](#二技术栈全景图)
3. [目录结构到底在讲什么](#三目录结构到底在讲什么)
4. [推荐的学习路径](#四推荐的学习路径)
5. [如何把项目跑起来](#五如何把项目跑起来)
6. [核心机制拆解](#六核心机制拆解)
7. [一个完整功能的代码是怎么流转的](#七一个完整功能的代码是怎么流转的)
8. [如何新增一个自己的业务模块](#八如何新增一个自己的业务模块)
9. [高频注解与工具速查表](#九高频注解与工具速查表)
10. [学习资源与避坑提醒](#十学习资源与避坑提醒)

---

## 一、这个项目是什么

**RuoYi-Vue-Plus** 是 Dromara 社区（`org.dromara`）基于经典 RuoYi 二次开发的后台管理系统框架，专门为 **分布式集群** 场景做了全方位升级，**不兼容**原版 RuoYi。

它的本质是一套 **"开箱即用的企业级 Web 后台脚手架"**。你不需要从零搭项目，而是基于它已有的权限、用户、部门、菜单、日志、文件、代码生成等能力，直接往上加你自己的业务。

**当前版本信息**（来自根 [pom.xml](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/pom.xml#L15-L26)）：

| 关键项 | 版本/值 |
|--------|---------|
| 模块版本 `revision` | 6.0.0 |
| 编程语言 / JDK | Java，要求 **JDK 21** 及以上（支持 25） |
| 核心框架 | Spring Boot **4.1** |
| 开发环境 | 默认 `dev`（可用 `-P local / dev / prod` 切换） |

> 与老 RuoYi 最大的几个区别（详见 [README.md](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/README.md)）：
> - 权限认证从 Spring Security 换成 **Sa-Token**
> - ORM 从手写 MyBatis XML 换成 **MyBatis-Plus**（基本不写 SQL）
> - 单体模块注入耦合 → **插件化多模块**解耦
> - 内置了大量开箱能力：工作流、代码生成、文件/云存储、短信、邮件、定时任务、链路追踪等

---

## 二、技术栈全景图

先建立一张"用到什么"的地图，遇到时能对上号即可，不必一次全记。

### 2.1 基础与构建

| 技术 | 作用 |
|------|------|
| Maven 多模块 | 整个项目拆成若干 jar 子模块，统一版本管理 |
| JDK 21 | 支持虚拟线程、switch 新模式、sealed 等新特性 |

### 2.2 Web 与接口层

| 技术 | 作用 |
|------|------|
| Spring Boot 4.1 | 主框架，自动装配<br>默认容器是 **Jetty**（而非 Tomcat） |
| `R<T>` 统一返回体 | 所有接口返回都会包一层 `{code, msg, data}` |
| SpringDoc + javadoc | 接口文档零注解生成（只要把注释写好） |
| GlobalExceptionHandler | 全局异常统一兜底 |

### 2.3 安全与认证

| 技术 | 作用 |
|------|------|
| **Sa-Token** | 登录凭证、权限/角色校验（`@SaCheckPermission` 等） |
| JWT | 无状态令牌 |
| JustAuth | 微信、钉钉等第三方登录 |
| Sa-Token 注解 | `@SaIgnore`(放行)、`@SaCheckLogin`、`@SaCheckRole`、`@SaCheckPermission` |

### 2.4 数据层

| 技术 | 作用 |
|------|------|
| **MyBatis-Plus** | 对象化增删改查，几乎不用写 SQL；主键默认雪花 ID（`ASSIGN_ID`） |
| MyBatis-Plus 分页插件 | `PageQuery` + `PageResult` 对象化分页 |
| dynamic-datasource | 多数据源、异构切换（可连 MySQL/Oracle/PostgreSQL/SQLServer） |
| 数据权限插件 | 注解 + SQL 拼接，无感数据隔离 |
| 数据加密/脱敏插件 | 存取时自动加解密、序列化时脱敏 |

### 2.5 缓存 / 锁 / 任务

| 技术 | 作用 |
|------|------|
| Redis + **Redisson** | 缓存、分布式限流、分布式队列 |
| Spring-Cache | 缓存注解（`@Cacheable` 等，可扩展过期时间） |
| **Lock4j** | 分布式锁（基于 Redisson） |
| **SnailJob** | 分布式定时任务调度 |
| Warm-Flow | 工作流引擎（审批流转） |

### 2.6 基础设施组件

| 常见组件 | 简记作用 |
|---------|---------|
| MinIO / AWS S3 | 文件与云存储 |
| sms4j | 短信融合 |
| mail-api | 邮件 |
| netty / WebSocket / SSE | 实时消息推送 |
| Spring Boot Admin | 服务监控，在线日志 |
| SkyWalking / actuator | 链路追踪、探针 |
| Easy-ES | Elasticsearch 封装 |
| LiteFlow | 规则/流程编排 |
| Spring AI + MCP | AI 应用与 MCP 服务端 |

> 前端是**另外独立**的项目（不需要本项目代码），官方是 Vue3 + TS + ElementPlus 的 `plus-ui`。本仓库只含有**后端**。

---

## 三、目录结构到底在讲什么

顶层是一个 Maven 聚合工程，用 `groupId = org.dromara` 统一管理。根目录下的几个大模块：

```
RuoYi-Vue-Plus-6.X/
├── pom.xml                    # 根 POM：统一版本、环境 profile
├── ruoyi-admin/               # 启动主程序 + 认证/注册（Spring Boot 入口）
├── ruoyi-api/                 # 对外开放的接口/DTO 定义（供业务模块互相调用）
├── ruoyi-common/              # 通用工具与组件库（被业务模块依赖）
├── ruoyi-modules/             # 业务模块（真正放业务代码的地方）
├── ruoyi-extend/              # 扩展服务（监控后台、任务调度后台）
├── script/                    # 初始化 SQL、部署脚本、docker 编排
└── .run/                      # 各服务的 IDEA 启动配置
```

### 3.1 ruoyi-common —— 通用组件库（按功能拆分，最稳定的基石）

里面是一堆 `ruoyi-common-*` 小 jar，**按职责拆得很细**：

- `ruoyi-common-core`：核心工具类、常量、`R`/`R<T>` 返回体、`PageResult`、通用异常
- `ruoyi-common-mybatis`：MyBatis-Plus 封装、分页 `PageQuery`、数据权限 `DataPermissionHelper`
- `ruoyi-common-redis`：Redis/Redisson 封装、`RedisUtils`、`@RepeatSubmit`
- `ruoyi-common-satoken`：Sa-Token 封装、`LoginHelper`（当前登录用户工具）
- `ruoyi-common-security`：加解密、脱敏、接口传输加密
- `ruoyi-common-log`：操作日志 `@Log`
- `ruoyi-common-excel`：Excel 导入导出
- `ruoyi-common-doc`：接口文档自动配置
- 其他：`json`、`oss`、`sms`、`mail`、`job`、`mqtt`、`websocket`、`translation`、`encrypt`、`sensitive`、`social`、`mcp`、`ai`、`elasticsearch`、`liteflow`、`log` 等

> 读代码顺序建议：**先 `common-core`，再看别的**，因为它被所有模块依赖，是所有共用概念（R、异常、工具）的家。

### 3.2 ruoyi-modules —— 业务模块（学习重点）

[ruoyi-modules/pom.xml](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/ruoyi-modules/pom.xml#L18-L25) 里列出了全部业务模块：

| 模块 | 作用 |
|------|------|
| `ruoyi-system` | 系统管理（用户/角色/部门/菜单/字典/日志/OSS 等），**必看核心** |
| `ruoyi-demo` | 各功能的演示用例，**学代码的最佳教材** |
| `ruoyi-gen` | 代码生成器（设计表结构即可生成 CRUD） |
| `ruoyi-job` | 定时任务模块 |
| `ruoyi-workflow` | 工作流/审批模块 |
| `ruoyi-ai` | AI 相关能力 |

> 学习路径上，`ruoyi-demo` 和 `ruoyi-system` 是最该研读的。前者教你"每个能力怎么用"，后者教你"一个正经业务模块长什么样"。

### 3.3 ruoyi-admin —— 启动入口与认证接口

- 启动类：[DromaraApplication.java](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/ruoyi-admin/src/main/java/org/dromara/DromaraApplication.java)
- 认证/注册 controller（`org.dromara.web`）：登录、退出、注册、第三方登录
- 配置文件：`application.yml`（通用）、`application-dev/prod.yml`（分环境）

> `ruoyi-admin` 只负责"组装"，它依赖所有业务模块。业务模块之间通过 `ruoyi-api` 暴露接口互相调用，避免直接 import 耦合。

---

## 四、推荐的学习路径

新手最容易犯的错是一头扎进细节直接读某个 service。建议按下面**四个阶段**递进：

### 阶段 0：先把项目跑起来（建立信心）
先让程序能在本地启动，再谈读代码。见下一节。

### 阶段 1：理解"一个请求是怎么走通的"（建立全局感）
目标：搞清楚 `Controller → Service → Mapper → 数据库 → 返回` 这条主线。
入口建议：
- [AuthController.login()](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/ruoyi-admin/src/main/java/org/dromara/web/controller/AuthController.java#L75-L107)
- [SysUserController](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/ruoyi-modules/ruoyi-system/src/main/java/org/dromara/system/controller/system/SysUserController.java)

### 阶段 2：看懂"一个业务模块"内部怎么组织（结构感）
读懂 `ruoyi-system` 的一个领域（比如用户），弄清 `domain / bo / vo / mapper / service / controller` 各放什么。

### 阶段 3：搞懂公共机制（深度）
- Sa-Token 登录与权限校验如何生效
- 数据权限 `DataPermissionHelper` 怎么做到无感过滤
- 分页 `PageQuery`/`PageResult` 如何工作
- 缓存、日志、Excel、接口加密如何通过注解+切面实现

### 阶段 4：动手写一个自己的模块（输出）
用代码生成器（`ruoyi-gen`）或手写一个极简 CRUD，贯彻阶段 1~3 的理解。

---

## 五、如何把项目跑起来

### 5.1 必要的环境

| 依赖 | 说明 |
|------|------|
| JDK 21+ | 必须 |
| Maven 3.x+ | 可用 mvnw 自带 wrapper |
| MySQL | 主数据库（需导入 SQL） |
| Redis 6+ | 缓存/登录令牌都依赖 |
| Node.js（可选，前端用） | plus-ui 项目需要；纯后端调试可忽略 |

### 5.2 初始化数据库

SQL 脚本在 [script/sql](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/script/sql)：

| 脚本 | 作用 |
|------|------|
| `ry_vue.sql` | 系统核心表（user/dept/role/menu/dict/oper_log 等） |
| `ry_job.sql` | 定时任务表 |
| `ry_workflow.sql` | 工作相关表 |
| `ry_ai.sql` | AI 相关表 |

先创建空数据库（如 `ry-vue`），然后依次导入上述 `.sql`，数据库连接信息在 `application-dev.yml`。

> 也可以直接 `script/docker/docker-compose.yml` 一键拉起数据库/Redis/中间件。

### 5.3 启动

IDEA 打开 `ruoyi-admin/src/main/java/org/dromara/DromaraApplication.java`，运行 `main` 即可。
或命令行：

```bash
mvnw spring-boot:run -pl ruoyi-admin
```

启动成功后控制台会打印“RuoYi-Vue-Plus启动成功”，默认端口 `8080`（见 `application.yml`）。

> 注意：若开了 `api-decrypt.enabled=true`（接口加密），直接用 Postman 测接口会比较麻烦，学习阶段可在 `application.yml` 临时改为 `false` 便于调试（正式环境再打开）。

---

## 六、核心机制拆解

### 6.1 统一返回体 `R<T>` 与分页 `PageResult`

几乎所有接口都返回 `R.ok(...)` / `R.fail(...)`，结构固定：

```json
{ "code": 200, "msg": "操作成功", "data": { ... } }
```

`PageResult<T>` 是对 MyBatis-Plus 分页结果的封装，含 `total` 总记录数、`rows` 数据列表、`pageNum`、`pageSize` 等。

### 6.2 MyBatis-Plus 对象化操作（少写 SQL）

在 `ruoyi-common-mybatis` 中封装了基类。业务 mapper 继承 MP 的 `BaseMapper<T>`，单表增删改查直接调方法，只有复杂查询才配 XML（mapper 目录里 `*Mapper.xml`）。

- 主键策略：`application.yml` 中 `id-type: ASSIGN_ID`（雪花 ID），新增时自动生成。
- 逻辑删除：`enableLogicDelete` 全局开启，配置了 `@TableLogic` 的字段在删除时走"软删除"。

### 6.3 权限认证 Sa-Token

登录成功后 `LoginHelper.getLoginUser()` 取出当前用户，里面带 `userId`、`menuPermission`（权限码）、`rolePermission`。接口上用注解控制：

```java
@SaCheckPermission("system:user:list")   // 需要权限码才能访问
@SaIgnore                                // 放行，不校验登录
@SaCheckLogin / @SaCheckRole(...)
```

### 6.4 分层与对象命名规范（看懂包的关键）

一个业务领域（如「用户」）在 `ruoyi-system` 里是这样组织的（**这套规范全项目通用**）：

```
controller/SysUserController.java    # 接收 HTTP 请求、做参数校验、返回 R
service/ISysUserService.java         # 接口（业务逻辑抽象）
service/impl/SysUserServiceImpl.java # 实现（真正的业务逻辑）
domain/SysUser.java                  # 实体/DO，对应数据库表
domain/bo/SysUserBo.java             # Business Object，用于接收前端传入参数
domain/vo/SysUserVo.java             # View Object，用于返回给前端展示
mapper/SysUserMapper.java            # 数据库访问
mapper/*.xml                         # 复杂 SQL
```

约定速记：
- **DO（domain）**：和表字段一一对应
- **Bo**：请求入参（输入），Controller 里 `@RequestBody SysUserBo`
- **Vo**：响应出参（输出），接口返回 `R<SysUserVo>` / `R<PageResult<SysUserVo>>`
- 命名统一：接口 `IxxxService`，实现 `IxxxServiceImpl`，Controller `XxxController`

> **为什么分 Bo/Vo**：避免把数据库字段裸传给用户，且前后端参数结构可各自演进，天然解耦 + 更安全。

### 6.5 登录流程串讲（把上面串起来）

看 [AuthController.login()](file:///c:/springwork/RuoYi-Vue-Plus-6.X/RuoYi-Vue-Plus-6.X/ruoyi-admin/src/main/java/org/dromara/web/controller/AuthController.java#L75-L107)：

1. 前端 POST 加密的登录信息 → `@ApiEncrypt` 解密。
2. 根据 `clientId` 找客户端配置 `SysClientVo`，校验授权类型 `grantType`。
3. `IAuthStrategy.login(...)` 走到对应策略（密码/短信/邮箱/三方/小程序），见 `org.dromara.web.service.impl` 下的 `PasswordAuthStrategy` 等。
4. 校验账号密码，生成 JWT 令牌，缓存用户信息到 Redis。
5. 返回 `LoginVo`（令牌 + 用户信息）。

之后每次带令牌访问，Sa-Token 过滤器校验有效性，`LoginHelper` 拿当前用户。

---

## 七、一个完整功能的代码是怎么流转的

以 **「分页查询用户列表」** 为例，串起完整链路（照这个思路去读，任何接口都能看懂了）：

```
浏览器 / plus-ui
   │   GET /system/user/list?pageNum=1&pageSize=10
   ▼
SysUserController.list(SysUserBo, PageQuery)          // ① 控制层
   │   校验权限 @SaCheckPermission("system:user:list")
   ▼
ISysUserService.selectPageUserList(user, pageQuery)   // ② 服务接口
   ▼
SysUserServiceImpl.selectPageUserList()               // ③ 服务实现：组装查询 + 分页
   │   内部通过 MyBatis-Plus 的 lambdaQuery / 分页插件查库
   ▼
SysUserMapper（继承 BaseMapper<SysUser>）              // ④ 数据访问
   │   解析成 SQL 执行，必要时走数据权限过滤
   ▼
MySQL
   │
   ▼（返回向上逐层封装）
PageResult<SysUserVo>  →  R.ok(...)                   // ⑤ 统一外包返回
   ▼
前端拿到 { code, msg, data:{total, rows} } 渲染表格
```

**关键点**：
- 每一层只负责自己的事：Controller 管"接参数 + 校验 + 返回"，Service 管"业务逻辑"，Mapper 管"数据"。
- 入参 `SysUserBo`（输入模型）、出参 `SysUserVo`（输出模型）分离，避免把数据库实体直接暴露。

> 看完这一个，再看新增/修改/删除就顺了——无非是 `@PostMapping`/`@PutMapping`/`@DeleteMapping` + 对应 Service 方法，加 `@Log` `@RepeatSubmit` 记录日志和防重复提交。

---

## 八、如何新增一个自己的业务模块

核心原则：**新代码只依赖通用组件（ruoyi-common-*）和 ruoyi-api，不互相 import 其它业务模块的具体实现类。**

### 方式 A：用代码生成器（推荐，最快）
1. 在数据库先建好业务表。
2. 在 `ruoyi-gen`（代码生成）界面导入表。
3. 配置后一键生成 Controller/Service/Mapper/DO/Bo/Vo 及前端页面，下载解压放进对应模块即可。

### 方式 B：手写一个极简模块（理解原理）
在某个业务模块下（例如就建在 `ruoyi-demo` 里练手，或新建 `ruoyi-xxx`），照规矩建：

```
xxx/controller/XxxController.java     # REST 接口
xxx/service/IXxxService.java          # 接口
xxx/service/impl/XxxServiceImpl.java  # 实现
xxx/domain/Xxx.java                   # 实体，对应表
xxx/domain/bo/XxxBo.java              # 入参
xxx/domain/vo/XxxVo.java              # 出参
xxx/mapper/XxxMapper.java             # 继承 BaseMapper<Xxx>
```

Controller 示例（一个查 + 一个增即可跑通）：

```java
@RestController
@RequestMapping("/demo/goods")
@RequiredArgsConstructor
public class XxxController extends BaseController {

    private final IXxxService service;

    @SaCheckPermission("demo:goods:list")
    @GetMapping("/list")
    public R<PageResult<XxxVo>> list(XxxBo bo, PageQuery pageQuery) {
        return R.ok(service.queryPageList(bo, pageQuery));
    }

    @Log(title = "商品管理", businessType = BusinessType.INSERT)
    @SaCheckPermission("demo:goods:add")
    @PostMapping
    public R<Void> add(@Validated @RequestBody XxxBo bo) {
        return toAjax(service.insertByBo(bo));
    }
}
```

在 Service 实现里，用 MyBatis-Plus 的对象化操作即可，例如：

```java
public PageResult<XxxVo> queryPageList(XxxBo bo, PageQuery pageQuery) {
    LambdaQueryWrapper<Xxx> wrapper = Wrappers.lambdaQuery(Xxx.class)
        .eq(Xxx::getStatus, bo.getStatus());            // 加条件
    Page<Xxx> page = mapper.selectPage(pageQuery.build(), wrapper);  // 分页
    return PageResult.build(page, XxxVo::new);          // 转 Vo
}
```

> 新建模块后，记得在对应的聚合 `pom.xml`（如 `ruoyi-modules/pom.xml`）的 `<modules>` 里加上，并把模块依赖（`ruoyi-admin`）一并声明。

---

## 九、高频注解与工具速查表

| 注解 / 工具 | 作用 |
|------------|------|
| `R.ok() / R.fail()` | 统一响应封装 |
| `@SaCheckPermission("x:y:z")` | 需要某权限码 |
| `@SaCheckRole(...)` | 需要某角色 |
| `@SaCheckLogin` | 需要登录 |
| `@SaIgnore` | 跳过登录/权限校验 |
| `@Log(title=, businessType=)` | 记录操作日志 |
| `@RepeatSubmit` | 防止重复提交 |
| `@ApiEncrypt` | 接口加解密 |
| `@Validated` / `@RequestBody` | 参数校验 / JSON 实体接收 |
| `LoginHelper.getLoginUser()` | 当前登录用户 |
| `PermissionHelper/StreamUtils/StringUtils` | 常用工具（来自 hutool/自封装） |
| `PageQuery` / `PageResult` | 分页入参 / 分页结果 |
| `DataPermissionHelper.ignore()` | 临时忽略数据权限 |
| `RedisUtils` | Redis 快捷工具 |

---

## 十、学习资源与避坑提醒

### 10.1 官方资料
- 官方文档：[plus-doc.dromara.org](https://plus-doc.dromara.org)（**初始化、部署必看**）
- 前端项目：`plus-ui`（Vue3 + TS + ElementPlus）
- 专栏/视频（入门必看）：文档站的 `common/column`

### 10.2 避坑提醒（新手高发）
1. **JDK 必须是 21+**，版本过低会编译失败。
2. **缺 Redis / 数据库** 时启动即报错——先把环境起好，或用 docker-compose 一键编排。
3. **改环境用 `-P dev/prod/local`**，不要硬编码环境名。
4. **别改通用组件（ruoyi-common）的公共行为**去适配单个业务，会全局受影响；自定义能力优先新建业务模块。
5. **读接口文档不用写注解**——把方法/字段的 javadoc 注释写规范，SpringDoc 会自动生成，所以**代码注释很重要**。
6. **新增自己的模块**不要直接 import 别的业务模块的具体类，通过 `ruoyi-api` 调用，保持解耦。
7. 学习阶段若被"接口加密"挡住调试，可临时关 `api-decrypt.enabled`（`application.yml`），理解后再开。

### 10.3 推荐的研读顺序（读源码）
```
1. common-core 的 R、异常、常量、PageResult
2. ruoyi-demo 里跑一个接口的完整链路
3. ruoyi-system 的 SysUserController → SysUserServiceImpl → SysUserMapper
4. satoken 过滤器 + LoginHelper（认证怎么生效）
5. mybatis 的分页插件 + 数据权限（高级）
```

---

**一句话总结**：这是一个"分层清晰、插件化多模块、开箱即用的企业级后台"，学习它的关键是抓住 **`Controller → Service → Mapper`** 这条主线 + **Bo/Vo/Do 分层的命名规范**，再由浅入深去理解 Sa-Token 权限、MyBatis-Plus 数据层和各类注解切面。把 `ruoyi-system` 的「用户管理」完整读懂，你就已经超过 80% 的新手了。