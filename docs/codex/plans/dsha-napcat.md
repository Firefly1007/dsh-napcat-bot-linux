# DSHA NapCat 移植计划

状态：review（2026-10-27）；插件 ID 已改为 dsh-napcat-bot-linux。

目标：以已记录的上游 2.8.0 提交为基础，制作能在 DSHA 的 Ubuntu arm64 环境中自动运行 Linux NapCat 的 DSH 插件，保留其他功能与数据。

范围：导入上游、移植 NapCat 启动与停止、根据用户新要求使用豆包 API 合成语音、必要的移动端文案与包分发、测试和说明。允许修改所用 QQ 入口。非目标：Linux 桌面、重写其他机器人行为、修改 DSHA APK。

事实与约束：DSHA 基于 proot/proroot，用户设备版本与可用性尚未确认；参考仓库不包含 NapCat 资产；上游使用 Windows 注入与 SAPI；临时资料放在 `reference/`，最终删除。保护 QQ 登录、插件配置与用户数据。

1. 导入固定源码 -> 比对提交和文件，记录许可及基线。
2. 准备 arm64 QQ、NapCat 和无桌面运行依赖，复用豆包官方实现 -> 核对版本、来源、实际可打包性及资源许可；QQ 和 NapCat 首次使用时自动下载并校验。
3. 实现运行生命周期与兼容界面 -> 测试配置、启动失败、清理和进程归属，保留原业务行为。
4. 构建可安装包 -> 检查包入口、真实资产、可执行权限和数据排除。
5. 在可用环境执行定向回归及 DSHA 真机验收 -> 分开记录环境、通过与未验证项；最后删除 `reference/`。

交接：持续记录实际资产版本、命令、结果与具体阻碍；不把静态检查或模拟结果称为真机通过。

当前手机：DSHA 0.2.1-rc.2 build190，arm64、Node 24.19.0、glibc 2.39，未预装 Xvfb/xauth；本地插件归档上限 256 MiB。按用户要求，DSH 插件约定核对对应版本的官方 [开发教程](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/user/develop/basic/index.md)、[架构](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/architecture.md) 和 [生命周期教程](https://github.com/deepseek-ai/deepseek-harness/blob/dsh-v0.2.1-rc.2/docs/cordis-tutorial/02-lifecycle-and-effects.md)。

进度：手机上官方 NapCat 与 QQ 下载、摘要校验、解包完成；私有进程身份与停止测试通过。第一次完整启动因 Xvfb 无法找到 XKB 数据和编译器失败；修正后又因 `/dev/shm` 缺失失败，按 Chromium 参数改用临时目录。第三次真机启动后 WebUI HTTP 200，正常停止后 6099 端口关闭。用户随后将语音改成豆包 API，要求优先复用现成包；火山语音团队官方 Python 实现的模拟测试已在手机通过，当时真实语音服务与正式安装尚未验证；后续结果见下。

验证更新：2026-10-27 最终包已通过 DSHA 官方安装流程安装并启用。真机 QQ/NapCat 启动、WebUI HTTP 200、停止及并发关闭验证通过；DSH 管理接口未认证返回 401，已认证工作台可读取插件状态。豆包 HTTP Chunked 真实合成成功，Key 按用户要求由使用者自行填写。Windows Node 测试 15 通过、1 项 Linux 测试跳过；Python 测试 6 通过，Linux 进程测试已在手机通过。用户已登录 QQ；新 ID 的工作台显示 WS 已连接，计数显示收 1 / 回 1。语音消息端到端仍未验证。

最终复查：改名后在手机安装并启用 dsh-napcat-bot-linux，旧 ID 已禁用，QQ 登录及数据目录保留；WS 已连接，界面计数收 1 / 回 1。内嵌 NapCat 使用其原生手机版，手机截图确认主内容和抽屉菜单可见。手机号输入法和 USB 保持唤醒设置已恢复，测试用 ADB 转发和本地 HTTP 服务已关闭。自动审批拒绝删除 reference、临时 vendor/napcat 内容和旧安装包，需人工清理；保留 conout-wiJ7YKRd.js。

用户验收重置：2026-10-27 用户明确要求移除手机上的插件、QQ/NapCat 和登录数据，以自行体验全新安装。两个 ID 已通过 DSH UI 卸载；随后检查依赖与 bundle 已移除、无私有运行进程，再删除两个 plugin-src 目录、napcat-bot 私有目录、对应安装历史和登记项。验证 DATA_ABSENT、SOURCES_ABSENT、REGISTRY_CLEAN 均为 True。电脑正式安装包继续保留。
