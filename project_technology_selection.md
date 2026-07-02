# Project Technology Selection

## 运行环境

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| Node.js | 运行时 | 前端构建链与工具生态的运行基座 | 驱动 Vite 构建、pnpm 脚本执行、前端本地开发服务器 | • 禁止在生产环境使用非 LTS 版本<br>• 版本须与 `packageManager` 字段及 CI 镜像一致 |
| Python | 运行时 | 后端服务与数据计算的运行基座 | 承载 Litestar ASGI 应用、Taskiq 异步任务、数据处理脚本 | • 禁止混用 Python 2 语法<br>• 类型注解覆盖率须 ≥ 80%<br>• 禁止绕过 uv 使用系统 Python |
| Docker | 容器编排 | 统一开发、测试与生产的交付标准 | 本地基础设施一键编排、CI/CD 镜像构建、消除环境差异 | • 禁止在容器内持久化业务数据<br>• 镜像须多阶段构建<br>• 禁止将生产密钥写入镜像或 Compose 文件 |
| 微信开发者工具 | 调试平台 | 小程序侧的真机预览与调试入口 | 编译产物预览、真机调试、体验版与正式版上传 | • 禁止绕过版本审核直接上传生产版本<br>• 禁止引入仅 Web 可用、未做小程序适配的依赖 |

## Monorepo 工具链

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| pnpm | 包管理 | 前端依赖的精准安装与 workspace 链接 | 硬链接节省磁盘、workspace 协议串联共享包、lockfile 锁定版本 | • 禁止混用 npm/yarn<br>• lockfile 变更须经过 Code Review<br>• 内部包禁止 `file:` 引用 |
| uv | 包管理 | Python 依赖的极速解析与虚拟环境管理 | 替代 pip/poetry、生成 uv.lock、支持多包 workspace | • 禁止混用 pip/poetry 破坏锁文件一致性<br>• `uv.lock` 须入库并与 CI 安装路径一致 |
| Turborepo | 任务编排 | 跨包构建任务的缓存与流水线编排 | 缓存 build/test/lint 产物、远程缓存加速 CI、管理包间依赖顺序 | • 禁止 `dev` 任务开启缓存<br>• `outputs` 须覆盖 Vite 产物与生成代码目录，否则破坏下游包缓存 |

## 共享包与 API 契约

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| packages/contracts | 契约包 | 前后端类型与接口规范的单一事实来源 | • 后端 Litestar 导出 openapi.json 至此<br>• 通过 CI 或本地脚本驱动 hey-api 生成前端类型与请求契约 | • 禁止在契约层引入框架专属代码<br>• 变更须先改后端模型/OpenAPI，再触发代码生成与版本同步 |
| packages/api-client | 客户端包 | 前后端 HTTP 交互的类型安全封装层 | • hey-api 读取 contracts 自动生成<br>• 供 Web 与小程序复用，统一请求与错误处理 | • 禁止手动修改生成代码<br>• 禁止绕过此包用 fetch/axios 直连接口<br>• 须与后端 API 版本严格对齐 |

## 质量与安全门禁

### 代码质量工具

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| pre-commit | Git 钩子 | 提交前的自动化质量拦截 gate | 统一触发 ESLint/Prettier/Ruff/pyright，阻止不合规代码入库 | • 禁止 `--no-verify` 跳过提交（须审批）<br>• 钩子配置须与 CI 任务集保持一致 |
| lint-staged | 增量检查 | 仅对变更文件执行检查以缩短反馈时间 | 对 staged 文件跑 lint 与格式化，避免全量扫描浪费 CI 资源 | • 禁止将 pyright 配置为仅检查 staged 文件<br>• staged 规则须与 CI 全量规则一致 |
| ESLint | 静态分析 | TS/Vue 代码的规范与缺陷检查 | 统一编码风格、发现未使用变量与潜在 bug、阻断不安全模式 | • 禁止引入与 Prettier 冲突的格式化规则<br>• 禁止对 `api-client` 生成代码关闭关键规则而不加 ignore 约定 |
| Prettier | 格式化 | 前端代码排版的自动化统一 | • 格式化 TS/Vue/CSS/JSON/Markdown<br>• 不覆盖 Python 文件（Python 格式化由 Ruff 负责） | • 禁止在同一项目混用其他格式化器<br>• 禁止提交未格式化代码导致 CI 与本地不一致 |
| Ruff | 静态分析/格式化 | 同时负责 Python 代码的静态分析与格式化 | 替代 flake8 + black + isort，统一风格与逻辑检查，极速执行 | • 禁止再引入 Black/Flake8/isort<br>• 禁止 Ruff 与 pyright 对同一问题重复且规则冲突 |
| pyright | 类型检查 | 专职强类型静态验证，与 Ruff 的语法检查互补 | • 验证 SQLAlchemy/Pydantic/Litestar 类型推断<br>• 覆盖 Ruff 无法处理的深度类型流分析 | • pre-commit 须 `pass_filenames: false` 全项目分析<br>• 禁止大面积 `type: ignore` 绕过 ORM/模型类型 |
| coverage.py | 测试覆盖率 | 测试范围的量化度量与门禁 | 生成覆盖率报告、设定门禁阈值、发现未测试的代码路径 | • 核心模块覆盖率须 ≥ 80%<br>• 禁止为刷覆盖率写无意义测试<br>• 禁止无测试的模块合入主干 |

### 安全扫描工具

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| Socket.dev | 供应链扫描 | 依赖树的安全风险与许可证审计 | 扫描 npm/PyPI 恶意包、拦截高风险依赖、PR 阶段自动检测 | • 禁止忽略 Socket 阻断的依赖<br>• 禁止未经审查引入带 install script 的高风险包 |
| Trivy | 漏洞扫描 | 镜像与文件系统的 CVE 漏洞发现 | 扫描 Docker 镜像漏洞、密钥泄露、文件系统安全风险 | • 禁止 HIGH/CRITICAL 未修复漏洞入主分支<br>• 禁止跳过对 Granian/Litestar 运行镜像的扫描 |

## 本地基础设施

> Docker Compose 本地启动；按需求选用子集，写入 `infrastructure/docker-compose.yml`

### 数据存储

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| PostgreSQL | 关系数据库 | 业务数据的持久化主存储 | ACID 事务、复杂查询、Alembic 迁移目标，与 pgvector 同实例共存 | • 禁止在生产环境使用 root 账户<br>• 密码复杂度须符合安全基线<br>• 禁止跳过 Alembic 直接改表 |
| Redis | 内存数据库 | 高速缓存与临时状态存储 | 热点数据缓存、分布式锁、会话管理 | • 禁止将 Redis 作为唯一持久化存储<br>• 敏感数据须加密<br>• 禁止业务代码依赖本地 Redis 特性在生产不可用 |
| MinIO | 对象存储 | S3 兼容的本地对象存储服务 | 文件/图片托管、附件上传、导出文件存储，统一存储抽象层 | • 禁止公开 Bucket 默认 ACL<br>• 上传文件须校验 MIME 类型<br>• 禁止将 MinIO 端点硬编码到业务代码 |
| pgvector | 向量扩展 | PostgreSQL 内的向量相似度检索 | Embedding 存储、语义搜索、RAG 场景下的向量近邻查询 | • 禁止用 pgvector 替代 Meilisearch 做拼写容错全文检索<br>• 禁止为向量检索单独引入第二套向量数据库 |

### 消息与搜索

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| RabbitMQ | 消息队列 | 异步任务的投递与系统解耦 | 承载 Taskiq 异步任务、削峰填谷、事件驱动架构的通信枢纽 | • 禁止消息无 TTL 无限堆积<br>• 生产环境须启用持久化与镜像队列<br>• 禁止生产使用默认 guest 账号 |
| Meilisearch | 搜索引擎 | 面向用户的全文检索服务 | 关键词搜索、拼写容错、排序与筛选、中文分词与实时索引 | • 禁止直接暴露搜索端口到公网<br>• 禁止将 Meilisearch 当作主库或唯一数据源 |

### 网络与身份

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| Caddy | 反向代理 | 统一流量入口与 TLS 终止 | 自动 HTTPS、HTTP/3、请求路由、负载均衡、后端端口隐藏 | • 禁止在生产环境使用自签名证书<br>• 配置变更须版本控制<br>• 禁止在 Litestar/Granian 重复实现 TLS 终止 |
| Logto | IAM | 身份认证与访问控制中枢 | OIDC/OAuth2 登录、JWT 签发、用户权限与角色管理 | • 禁止自建账号密码体系与 Logto 并存<br>• 禁止硬编码客户端密钥<br>• Token 过期策略须符合安全规范 |

### 可观测性

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| Netdata | 指标监控 | 系统与容器资源的实时监控 | CPU/内存/磁盘/网络指标采集、容器健康度监控、告警触发 | • 禁止关闭关键告警通道<br>• 监控数据保留期须 ≥ 30 天<br>• 禁止采集含 PII 的业务指标 |
| Dozzle | 日志查看 | 容器日志的实时聚合与查看 | 本地排障时快速查看多容器日志、辅助定位问题根因 | • 禁止在日志中输出明文密码、Token 等敏感信息<br>• 禁止依赖 Dozzle 做日志持久化与审计 |

## 前端技术栈

### 双端通用

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| TypeScript | 编程语言 | 前端代码的静态类型安全保障 | 类型检查、接口契约对齐、IDE 智能提示、编译期错误发现 | • 禁止显式使用 `any`<br>• `strict` 模式须开启<br>• 禁止在 TS 包中混写无类型 JS |
| Vue 3 | UI 框架 | 响应式用户界面的组件化构建 | Composition API、响应式数据驱动、组件复用与性能优化 | • 禁止混用 Options API 与 Composition API<br>• 须使用 `<script setup>`<br>• 禁止引入 Vue 2 生态（Vuex 等） |
| Vite | 构建工具 | Web 侧工程化支撑；小程序侧由 UniApp 接管构建 | • Web 侧极速 dev server、代码分割与生产打包<br>• 小程序侧须遵循 UniApp 条件编译与分包机制，部分插件需适配 UniApp 构建钩子 | • 禁止回退 Webpack<br>• 禁止各应用各自维护不兼容的 Vite 配置<br>• 禁止在小程序侧直接套用 Web 专属插件 |
| Pinia | 状态管理 | 客户端全局状态的集中管理 | 跨组件状态共享、模块化 Store、支持持久化插件扩展 | • 禁止在 Pinia 外直接修改全局状态<br>• Store 须按业务域拆分<br>• 禁止用 Vuex |
| pinia-plugin-persistedstate | 状态持久化 | 客户端状态到浏览器存储的持久化 | 将 Pinia 状态同步到 localStorage/sessionStorage，刷新后恢复 | • 禁止持久化敏感信息（Token 除外，须加密存储）<br>• 持久化 key 须按端（Web/小程序）隔离 |
| UnoCSS | CSS 引擎 | 原子化样式的按需生成与体积优化 | 原子类样式、按需生成、与 Reka UI 组合实现定制主题 | • 禁止与 Tailwind 并存<br>• 禁止在 UniApp 工程引入 UnoCSS 增加包体积 |
| hey-api | 代码生成 | 基于 OpenAPI 生成类型定义与接口契约 | • 读取 contracts 生成 TS Types 与接口路径契约，消除手写类型<br>• 实际请求执行由 alova 统一接管 | • 禁止手动修改生成产物<br>• 禁止跳过生成直接维护与 Pydantic 漂移的类型 |
| alova | 请求层 | HTTP 请求的策略化封装与多端适配 | • Web 侧用标准 fetch/axios<br>• 小程序侧通过 @alova/adapter-uniapp 调用 uni.request<br>• 两端统一拦截、缓存、防抖、分页、Token 刷新 | • 禁止裸用 fetch/xhr 调业务 API<br>• 禁止 Web 与小程序各写一套请求拦截逻辑 |
| Valibot | 数据校验 | 前端数据的轻量级 schema 校验 | 表单校验、API 响应校验、Tree-shaking 友好的类型验证 | • 禁止引入 Zod 并存增加体积<br>• 禁止手写与 Pydantic/OpenAPI 冲突的接口 schema |
| Vitest | 测试框架 | 前端单元与集成测试的执行框架 | 测试 composable、工具函数、组件，与 Vite 生态同构 | • 核心工具函数覆盖率须 ≥ 80%<br>• 禁止引入 Jest 形成双测试运行时 |
| consola | 日志工具 | 前端日志的统一输出与级别控制 | 开发调试、生产日志降级、结构化字段与后端 structlog 对齐 | • 禁止 `console.log` 散落<br>• 禁止输出 token、PII 到浏览器控制台 |
| unplugin-auto-import | 构建插件 | 框架 API 的自动导入与样板消除 | 自动导入 Vue/Pinia/Vue Router API，减少重复 import | • 禁止自动导入业务模块或 api-client<br>• 白名单仅含框架与已约定库 |
| unplugin-vue-components | 构建插件 | Vue 组件的按需自动注册 | 按目录自动引入组件、避免手动全局注册、支持 Tree-shaking | • 禁止自动注册跨应用共享的业务容器组件<br>• 组件目录须与 Monorepo 包边界一致 |
| rollup-plugin-visualizer | 构建分析 | 打包产物体积的可视化分析 | 生成体积报告、定位大依赖、指导代码拆分与懒加载 | • 禁止对未经分析的体积回归进行合入<br>• 禁止未分析就上调体积阈值 |

### Web SPA

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| Vue Router | 路由 | SPA 页面导航与权限控制 | 路由配置、代码分割懒加载、导航守卫、与后端鉴权联动 | • 禁止整页刷新跳转破坏 SPA 体验<br>• 禁止在组件内散落鉴权逻辑而不经守卫 |
| Reka UI | 无头组件 | 可访问性优先的 UI 基础原语 | 无样式组件、键盘可访问、ARIA 支持，与 UnoCSS 组合定制外观 | • 禁止引入 Element Plus/Ant Design 等与 Reka UI 并存<br>• 禁止修改组件内部 ARIA 属性 |
| ECharts | 图表引擎 | 数据可视化图表的渲染引擎 | 柱状图、折线图、饼图等数据图表，支持大数据量渲染 | • 禁止为简单展示再引入第二套图表库<br>• 禁止未经按需导入全量引入 ECharts |
| vue-echarts | Vue 集成 | ECharts 的 Vue 3 响应式封装 | 响应式 option 绑定、自动销毁、与 Vue 生命周期集成 | • 禁止绕过 vue-echarts 直接 `echarts.init` 散落各页<br>• 禁止在组件卸载后保留图表实例 |
| GSAP | 动画引擎 | 复杂动画与时间轴的精确控制 | 页面过渡、复杂时间轴动画、高性能动画效果，优先 CSS 简单过渡 | • 禁止用 GSAP 做可用 CSS 完成的简单动效<br>• 禁止在 prefers-reduced-motion 环境下播放动画 |
| Playwright | E2E 测试 | 端到端用户旅程的自动化验证 | 跨浏览器测试、核心路径自动化、录制追踪与回归防护 | • 核心用户旅程须覆盖 E2E 测试<br>• 禁止跳过失败测试直接合入<br>• 禁止引入 Cypress 双 E2E 栈 |
| VueUse | Composable 库 | 浏览器与 Vue 的组合式工具集 | 封装浏览器 API、响应式工具函数，减少重复造轮子 | • 禁止重复实现 VueUse 已有 composable<br>• 禁止在 UniApp 中引用仅 Web 可用的 VueUse 模块 |
| unplugin-icons | 构建插件 | SVG 图标的按需组件化引入 | 按需图标组件、支持多图标集、避免全量打包图标字体 | • 禁止全量打包图标集<br>• 禁止与 @lucide/vue 重复引入同一图标两套实现 |
| @lucide/vue | 图标组件 | 统一线条风格图标库 | 按钮/菜单/导航图标、主题变量跟随、Tree-shaking 优化 | • 禁止混用多种风格图标集<br>• 禁止在小程序侧引用 @lucide/vue（用 Wot Design 图标） |
| size-limit | 体积门禁 | 产物体积增长的自动化门禁 | CI 中限制 JS/CSS 体积、防止依赖膨胀、性能回归拦截 | • 禁止突破既定体积预算<br>• 上调限额须附带 visualizer 报告与拆分说明 |

### 微信小程序

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| UniApp | 跨端框架 | Vue 语法到小程序的编译适配层 | • 一套代码编译到微信小程序、条件编译处理平台差异<br>• 须利用 subPackages 分包加载控制单包 ≤ 2MB | • 禁止混写微信原生 WXML 破坏跨端一致性<br>• 禁止引入仅 H5 可用的 Vite/Web API |
| @uni-helper/uni-use | Composable 库 | 小程序侧的组合式工具封装 | 封装 uni API、与 alova 配合、对应 Web 侧 VueUse 角色 | • 禁止重复封装 uni 已有能力<br>• 禁止在此实现应用级业务状态（用 Pinia） |
| Wot Design Uni | UI 组件库 | 小程序 UI 层的高阶组件集 | 表单、列表、弹窗等组件、适配微信小程序设计规范 | • 禁止再引入 uView 等第二套 UI 库<br>• 禁止把 Web 的 UnoCSS/Reka 组件照搬到小程序 |

## 后端技术栈

| 技术 | 类别 | 职责 | 用途 | 红线 |
| ---- | ---- | ---- | ---- | ---- |
| Litestar | Web 框架 | ASGI 应用的核心框架与路由中枢 | HTTP 请求处理、DI 容器、OpenAPI 生成、中间件链与异常处理 | • 禁止与 FastAPI/Flask 并存<br>• 禁止手写路由而不生成 OpenAPI 破坏契约链<br>• 禁止在 handler 内写业务逻辑 |
| Pydantic v2 | 数据建模 | 请求/响应数据的校验与序列化 | 模型定义、自动校验、OpenAPI Schema 驱动、前后端类型契约 | • 禁止 v1 语法与配置<br>• 禁止 handler 用裸 dict 替代 Pydantic 模型<br>• 禁止在模型中使用 `Any` |
| Granian | ASGI 服务器 | 高性能 HTTP 服务的运行时承载 | Rust I/O 层、高并发处理、生产环境替代 Uvicorn | • 禁止依赖 trio/gevent 路由<br>• 禁止在生产环境使用开发服务器（`litestar run`） |
| Taskiq | 任务队列 | 异步任务的调度与执行框架 | 后台任务投递、定时任务、延迟执行、与 RabbitMQ 集成 | • 禁止用 Celery 双队列栈<br>• 禁止在 Granian worker 内同步阻塞长任务<br>• 失败任务须配置重试与死信队列 |
| SQLAlchemy 2.0 | ORM | 数据库对象关系映射与查询抽象 | async ORM 查询、模型定义、与 PostgreSQL 交互、2.0 风格 API | • 禁止 1.x `query()` 风格<br>• 禁止裸写 SQL（复杂报表除外）<br>• 须使用 2.0 风格 API |
| Scalar | API 文档 UI | OpenAPI 文档的现代化渲染界面 | 开发联调文档、与 Litestar OpenAPI 集成、接口可视化浏览 | • 禁止生产对公网暴露未鉴权文档<br>• 禁止维护与 OpenAPI 不一致的手写文档 |
| asyncpg | 数据库驱动 | PostgreSQL 的异步高性能连接 | async 数据库访问、连接池管理、与 SQLAlchemy async engine 配合 | • 禁止在 async 路径使用 psycopg2 等同步驱动<br>• 禁止连接池配置与 Compose 中 PG 实例不匹配 |
| httpx | HTTP 客户端 | 异步外部 HTTP 调用的统一客户端 | 服务间调用、第三方 API 集成、webhook 推送、超时与重试配置 | • 禁止 async 代码路径使用 requests<br>• 外部调用须配置超时，必要时走 Taskiq 异步化 |
| Alembic | 数据库迁移 | 数据库 schema 的版本控制与管理 | DDL 版本控制、升级/回滚脚本、与 SQLAlchemy 模型同步 | • 禁止手工改库不生成 revision<br>• 禁止生产执行未在 CI 验证过的 migration |
| structlog | 结构化日志 | 后端日志的统一结构化输出 | JSON 格式日志、聚合检索、字段约定对齐前端 consola | • 禁止 `print` 调试<br>• 禁止输出 JWT、数据库 URL 等敏感字段<br>• 须使用 key-value 绑定 |
| pytest | 测试框架 | Python 代码的自动化测试与验证 | 路由测试、仓储测试、任务测试、async 支持、fixture 管理 | • 核心逻辑覆盖率须 ≥ 80%<br>• 禁止提交失败的测试<br>• 禁止测试依赖未在 Compose 定义的不可复现外部服务 |
