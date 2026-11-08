# 项目记忆与开发约定

## 已确认的开发方向

- 本仓库为 DJI 上云 API 2，用户将与助手持续进行二次开发；默认使用中文沟通。
- 二次开发主工程为 `devkit/skeleton/`，后续未特别指定的业务开发以此为起点。
- SDK 暂时保持 `3.3.7`，Windows 本地使用 Mock 模式；待部署 Linux 服务器时再评估升级至 `3.4.0`，届时需验证接口兼容性。
- `devkit/fov-demo/` 是独立的培训示例，用于参考直播、OSD 遥测、SEI 视锥、Cesium 地图和手动操控实现。
- `devkit/fov-demo/backend/fhsdk-demo-server/` 是 FOV Demo 后端；定位主业务后端时应使用 skeleton 的启动模块。

## 项目记忆维护

- 根目录 `AGENTS.md` 维护开发方向、整体架构和跨项目约定；`devkit/skeleton/AGENTS.md` 维护 skeleton 的模块开发规则，具体规则在对应文件中更新。
- 项目文档中的仓库内路径使用相对路径，默认相对仓库根目录；使用其他基准时显式注明，方便在不同电脑和盘符下继续开发。
- 两份 `AGENTS.md` 随项目提交到 Git，通过拉取同一分支同步已确认的开发约定。

## Skeleton 架构与业务代码位置

Skeleton 是 Java 17 + Spring Boot 的 Maven 多模块单体应用，Vue 前端位于 `web/`。后端各模块由 `app/fh-skeleton-app/` 组装，运行在同一个 Java 进程中。

| 路径（相对 `devkit/skeleton/`） | 职责 |
| --- | --- |
| `pom.xml` | 聚合模块和共享构建配置 |
| `app/fh-skeleton-app/` | Spring Boot 启动入口、应用配置、模块依赖组装 |
| `feature/` | 业务功能；Controller、Service、DTO、实体与数据存取等按业务组织 |
| `kernel/kernel-spi/` | 公共接口、事件、查询等契约 |
| `kernel/kernel-impl/` | 事件总线等公共基础实现 |
| `kernel/kernel-mock-sdk/` | 模拟设备、遥测和事件，供本地开发与测试使用 |
| `kernel/kernel-real-sdk/` | 真实 SDK 初始化、能力适配和回调事件转换 |
| `fh-skeleton-bom/` | 集中管理一组 skeleton 模块的依赖版本 |
| `web/` | Vue 页面、组件、路由和前端接口请求 |
| `docker/`、`deploy/`、`deploy-init-sql/` | 容器、集群部署和数据库初始化配套 |

- 修改现有业务时进入对应 feature；新增独立业务时创建 `feature/feature-xxx/`，业务逻辑放在 feature 内，app 保持启动与组装职责。
- 现有 feature 包括 `device`（设备）、`wayline`（航线与任务）、`media`（媒体）、`terra`（重建）、`flight-area`（飞行区域）、`dashboard`（概览）、`user`（认证权限）、`chat`（AI 对话）。
- 开始 skeleton 内的工作前，阅读 `devkit/skeleton/AGENTS.md`，遵循其中的模块依赖、事件消费和自动配置注册规则；该文件是这些细节的维护位置。
- 新建 feature 可参考 `feature/feature-device/` 的 `DeviceRestController`、`DeviceStatusService`、`DeviceAutoConfiguration` 和 `.imports` 注册文件。

## 其他目录与运行关系

- `sdk/` 为客户端 SDK：`src/api/` 是 C++ 对外接口，`src/bll/` 是核心实现，`platform/java/` 是 Java 封装与 SWIG/JNI 绑定，`deps/` 和 `third_party/` 是支撑模块与第三方依赖。
- `devkit/mcp/` 将设备能力提供给 AI 工具调用，是独立工程。
- `devkit/libs/` 提供按 CPU 架构分发的预编译 SDK JAR；`devkit/skills/`、`devkit/knowledge/` 提供开发辅助资料。
- 根目录 `docs/` 提供平台部署、设备接入、鉴权和 SDK 等文档；skeleton 的 `docs/` 提供骨架工程说明。
- 自定义业务后端通过 SDK 接入上云 API 2 平台底座；底座有独立部署流程。
- 当前工作环境为 Windows。真实 SDK 原生库面向 Linux x86_64/aarch64；本地可使用 skeleton Mock 模式开发，真实接入需匹配 Linux 环境及 CPU 架构。
- README 与代码或配置有出入时，以当前源码和配置核实实际行为。

## DJI AI Plugin / Skills

- DJI 官方 `cloudapi2-*` Skills 已安装为本机 Codex 用户级技能；换电脑时需重新安装，项目中的 `AGENTS.md` 随 Git 同步不会同步用户技能目录。可在另一台电脑运行官方命令 `npx skills add https://github.com/dji-sdk/Cloud-API-2` 并选择 Codex。
- 处理 DJI SDK、前端组件、设备管理、遥测、地图、负载、AI、部署或排障任务时，按需加载对应 Skill；源码在 `devkit/skills/`，共享规则与路由在 `devkit/skills/_shared/`。
- DJI Skill 文档中以 `skills/`、`knowledge/`、`skeleton/`、`mcp/` 开头的项目路径以 `devkit/` 为基准；Skill 自身的 `references/`、`prompts/`、`commands/` 等路径相对该 Skill 目录。
- Skill 示例需结合本项目当前 SDK `3.3.7` 的真实接口验证；安装 AI Plugin 不改变已确认的 SDK 版本和 Windows Mock 开发方式。
- 设备操作用的 DJI MCP 服务接入需另行配置平台地址与凭证；安装 Skills 仅提供开发知识与操作指引。

<!-- CODEGRAPH_START -->
## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->
