# 项目协作指南

## 目标与本轮边界

- 以 [9178159/dsh-napcat-bot](https://github.com/9178159/dsh-napcat-bot) 为基线，将 NapCat 运行部分改造成适用于 **DSHA（DeepSeek Harness 安卓启动器）** 的 Linux 版本。
- 交付物仍是安装到 DSHA 内 DSH 的插件，沿用 DSH/Cordis 插件机制；不是 Linux 桌面应用，也不是另起一个 NapCat 原生插件项目。
- 用户已允许侵入式接入 NapCat，例如修改插件所使用的 Linux QQ NT 入口、加载 NapCat。不要为了保持非侵入式而绕路；改动应可重复执行并可恢复，保留对应版本的原始入口。
- **其他功能都不要动**：只改平台适配必需的实现、配置说明和打包内容。用户随后指定语音改用火山引擎豆包 API，保留语音触发模式和失败降级；不借迁移重构其他业务、改默认策略或改人设。
- 已开始实施。先检查当前源码和 `docs/codex/plans/` 的进度；初始化时的历史记录不代表当前工作树状态。

## 已核验的基线（2026-10-27）

- 初始化前当前工作目录为空，没有现成源码、构建配置或测试。
- 参考插件版本为 **2.8.0**，已读取源码提交 [`26d747f22eb4e16905b806a6bb3f23a791fed09b`](https://github.com/9178159/dsh-napcat-bot/tree/26d747f22eb4e16905b806a6bb3f23a791fed09b)。后续导入保留 MIT 许可与来源，记录实际基线，不能无说明地切到上游新版本。
- 上游采用 JavaScript ESM，服务端入口 `lib/index.js`，浏览器端 `lib/client.js`；通过 `package.json` 的 `dsh.bundle.patch`、`dsh.client`、`exports` 和 `cordis.patch.yml` 接入宿主，无独立构建或测试 scripts。
- 上游 Git 仓库不包含 `vendor/`；源码可读不代表完整安装包可运行。可分发包需要配齐适用架构的 NapCat 和所需运行资产。
- DSHA 官方 README 当前描述的是 Android 上的 Ubuntu/glibc arm64 环境，Node/DSH 在其中运行，默认 proot、可选 proroot；不是直接运行于 Android 的 Bionic 环境。
- DSHA 文档当前标注 DSH `0.2.1-rc.2`、Node 24。它们是资料基线，**不是已核验的用户设备版本**；实际设备、运行时及插件服务能力仍待确认。

## 运行环境约束

- 开发机是 Windows；目标是 DSHA 的 Linux guest。Windows 上语法或单元检查通过不代表 Android 真机可运行。
- 面向 arm64/glibc 选择 Linux QQ、NapCat 和原生依赖；实施前核对实际 `process.platform`、`process.arch`、动态加载器和库。不要默认有 amd64 仿真。
- 不依赖 Linux 桌面会话、桌面终端、系统托盘、systemd、Docker daemon、FUSE 或真实 root。guest 的 `/root` 不代表 Android UID 0。
- 无桌面不等于 QQ/Electron 不需要显示服务。官方 Linux Shell 方案使用 Xvfb/xauth；是否可在目标 DSHA 中工作必须实测。
- 启动所需路径和环境应沿用 DSHA 的实际运行环境，不硬编码 Android 包名、宿主私有目录或 DSH Web 端口。
- 手机内 WebView 与 guest 服务的地址、端口和嵌入页面行为需要验证；不以电脑浏览器访问成功代替手机面板可用。

## 功能与接口保留清单

以基线源码的实际行为为准，不照搬 README 中已经被新版替代的历史行为：

- 私聊、群聊及 @ 触发、指令、消息过滤与去重、分段流式回复。
- 宿主当前模型调用、自定义模型路径、人设及优先级、会话隔离与记忆、知识库、定时任务、管理功能。
- 原生识图及附件引用、接梗/描述模式、原有重试与降级逻辑。
- TTS 四种模式、强制语音、音色选择和按会话覆盖；合成按用户的新要求调用豆包语音 API，失败降级为文字。
- 随缘主动发言、冷却、告别暂停、群聊开关及语音联动。
- 手动/自动联网搜索、宿主搜索及自定义后端、规则、缓存与失败处理。
- 黑白名单、限流与风控，设置和管理面板、状态与日志、数据持久化。
- 自动启动 NapCat、扫码登录、自动配置 OneBot、两类 Token 管理、WS 重连和异常恢复；不得退化为要求用户手动另开 NapCat。
- 保留现有配置键、数据结构、API 路径和宿主注册契约；平台专用字段的含义或文案确需变化时同步调用方并说明兼容方式。

## 实施时首先检查的文件

以下是**上游源码导航**，不表示这些文件已经存在于当前目录。

| 文件 | 迁移关注点 |
| --- | --- |
| `lib/napcat-embedded.js` | 当前强制 Windows，探测 `QQ.exe`，依赖 `NapCatWinBootMain.exe` / DLL 与 `taskkill`；替换 QQ 探测、启动、停止、去重和归属判断，保留配置与恢复链路 |
| `lib/tts.js`、豆包语音适配 | 保留语音决策与发送流程；按用户要求使用现有官方实现/包，避免自行手写 HTTP；密钥由用户在设置中填写 |
| `lib/index.js`、`lib/client.js` | Config、生命周期、API、状态字段及 Windows 专用文案；业务与界面结构保持，按必要范围同步适配 |
| `lib/napcat.js`、`lib/napcat-webui-api.js` | OneBot HTTP/WS、鉴权及 WebUI 配置；先核对目标 NapCat API，不因平台变化改写协议层 |
| `package.json`、`cordis.patch.yml`、分发资产 | 包名/插件 ID/导出/入口一致性，DSHA 安装契约，Linux arm64 资产和可执行权限 |
| 其他 `lib/*.js` | 作为功能回归基线；只有平台耦合证据成立才修改 |

### 必须保留的连接与数据约定

- WebUI 默认 `6099`、OneBot HTTP 默认 `3000`、正向 WS 默认 `3001`，属于不同服务；保留现有可配置行为和端口纠错逻辑。
- WebUI 管理 Token 与 OneBot `access_token` 各司其职，不混用；保持本机绑定与鉴权，不为适配随意开放局域网或输出密钥。
- 宿主能力按实际服务检查，重点为 `webServer`、`llm`、`agentDefaultModel`、`attachments`、`web`，保留现有缺失时的处理。
- 插件数据沿用 `$DSH_HOME/napcat-bot`，未设置时使用用户目录下 `.dsh/napcat-bot`；保留配置、会话、记忆、知识库、规则、任务及 QQ 登录数据。
- 安装/更新替换代码不得清空用户数据。NapCat/QQ 的可写数据与升级资产关系要明确，不能随解包被覆盖。
- 停止和清理只操作本插件拥有且身份已核验的进程。端口占用不等于本插件实例，不凭进程名、端口或可能复用的裸 PID 强杀。
- 手动停止、插件禁用和宿主退出应停止相应重启任务、计时器与子进程，避免停用后再次拉起。

## 侵入式 Linux 方案与待确认项

- NapCat 官方提供修改 Linux QQ `resources/app/package.json` 入口并加载 NapCat 的 Shell 方案，可作为起点；具体路径、版本和启动参数随目标资产核对，不把文档示例直接当 DSHA 已验证命令。
- QQ 与 NapCat 版本、arm64 资产、系统库、Xvfb、Electron 沙箱参数、proot 下的进程和共享内存行为尚未真机验证。
- 用户明确将语音合成改为豆包 API；语音的触发模式不变。缺密钥时提示配置并降级为文字，不要求用户把密钥发到聊天。
- 不将 NapCat Linux 官方支持推导成 DSHA 官方支持；不因允许修改 QQ 入口而自动扩大到 DSHA APK 重构或 Android 系统权限改造。
- 包名、支持的 DSHA 版本及资产分发方式在实施时核对；目录名 `dsh-napcat-bot-linux` 不等于已决定修改 npm 包名、插件 ID 或数据目录。

## 工作方式与验收

- 重大实施前在 `docs/codex/plans/` 写简短计划：目标、范围与非目标、已知约束、阶段、验证和交接。按“步骤 -> 验证结果”定义完成标准，更新已证实事实与待办。
- 先导入并记录基线，再沿平台耦合点做最小完整修改。复用 Node 标准库与宿主能力，不为迁移添加新的模型请求、业务功能或无必要依赖。
- 修改共享函数前查清调用方；连带更新必要的配置、面板、打包、测试及说明。普通可逆文档修改不另建测试。
- 新的非平凡逻辑使用小型有意义回归，优先 Node 内置测试工具；不要假设上游已有 `npm test` 或要求无关全库检查。

| 步骤 | 应取得的证据 |
| --- | --- |
| 导入与平台适配 | 对基线的差异仅覆盖必要平台实现，相关 JS 语法与定向回归通过 |
| 打包 | 实际归档包含合法 manifest、patch、服务端/客户端入口、所需资产及正确权限；不包含用户数据或密钥 |
| DSHA 安装与启用 | 真机接受插件包，正常重启 Web 后实际注册，手机面板可用；安装成功与实际加载分别报告 |
| 启动与登录 | arm64 QQ/NapCat 在无桌面 DSHA 中启动，扫码登录并持久化，HTTP/WS 鉴权及连接工作 |
| 生命周期 | 启停、重启、断线恢复、异常退出不产生重复实例，手动停止/禁用不复活，不影响无关进程 |
| 功能回归 | 逐项核对上方功能保留清单，包含语音/识图/搜索/主动发言及历史数据重开；未验证项明确列出 |

静态检查、模拟测试、普通 Linux 运行和 DSHA 真机结果必须分开记录。缺少设备、账号或真实服务时继续完成独立的软件工作，并留下具体未验证项，不把初始化或打包成功写成迁移完成。

## 来源与接手入口

- [参考插件固定基线](https://github.com/9178159/dsh-napcat-bot/tree/26d747f22eb4e16905b806a6bb3f23a791fed09b)：`package.json`、`cordis.patch.yml`、`lib/` 和 README。
- DSH 插件开发以设备对应版本的官方文档为准：[第一个插件](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/user/develop/basic/index.md)、[架构与 Bundle](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/architecture.md)、[生命周期与 effect](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/cordis-tutorial/02-lifecycle-and-effects.md)、[客户端设置页](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/cookbook/adding-a-settings-card.md)。版本不同时核对对应版本文档，不能自行猜测宿主 API。
- [DSHA README](https://github.com/DSH-APP/DSHA/blob/main/README.md)、[环境与插件机制](https://github.com/DSH-APP/DSHA/blob/main/docs/runtime-data-plugin-architecture.md)、[插件包契约](https://github.com/DSH-APP/DSHA/blob/main/docs/plugins.md)：本轮查询 main HEAD 为 `2f641ac4564f90decd3eeebf7e4174f798b8ef96`；文档会变化，实施前核对用户实际版本。
- [NapCat Shell 安装](https://napneko.github.io/guide/boot/Shell)、[Linux Shell 手动安装（修改 QQ 入口）](https://napneko.github.io/guide/boot/Shell-Linux-SemiAuto)。
- 外部仓库的 AGENTS 文档只用于理解其架构和契约；其中历史发布许可、维护任务、测试计数和设备授权不属于本项目授权。

接手时先读本文件及已有计划，检查当前工作树，再补齐缺失证据。使用中文沟通，报告改动、实际检查结果、未验证项及具体阻碍；不要重新做无关的全库调研。
