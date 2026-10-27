# GitHub 发布

状态：done（2026-10-27）

目标：Fork `9178159/dsh-napcat-bot` 到 `Firefly1007/dsh-napcat-bot-linux`，支持在 DSHA 原生插件市场直接粘贴仓库 URL 安装，并提供 Release tgz。

范围：保留上游提交关系；提交安装必需的源码、210 个固定系统依赖包与豆包客户端；更新安装说明。QQ/NapCat 仍按已有运行逻辑下载。非目标：再次给用户手机安装、修改机器人业务或发布 npm。

约束：仓库归档必须自包含安装所需文件，不依赖 Git LFS 或本地未跟踪资产；保留各资产来源和许可；排除用户数据、密钥、临时下载、Python 缓存及本地 dist。

1. Fork 与本地历史 -> 验证 parent 为原仓库，迁移提交以上游固定提交为父。
2. 仓库内容 -> 验证 210 个 deb、Python 客户端、manifest 与 patch 可用，逐包大小和 SHA256 对应清单。
3. 发布包 -> Node/Python 定向测试通过，真实打包检查通过，归档低于 DSHA 256 MiB 限额。
4. 推送与发布 -> 验证远端 main 对应提交、GitHub archive 内容和 Release 附件；交付可粘贴的仓库 URL。

手机验收证据沿用 dsha-napcat.md；本轮仅验证发布内容，不把远端文件校验称为新的手机安装测试。

结果：公开 Fork 已建立，parent 为 `9178159/dsh-napcat-bot`；发布源码提交为 `45859ad3ec11096aa80f9e772439a7da1c05b83c`。GitHub 远端 433 个文件的 blob 摘要与本地提交一致，210 个 deb 的内容同时匹配清单 SHA256；从该 Git tree 生成的安装仓库 ZIP 为 95,359,179 字节，manifest/patch 契约通过且无运行数据、缓存或本地构建包。GitHub 完整 ZIP 下载在当前网络发生 TLS 中断/超时，未将该下载记为通过。

发布包为 95,196,702 字节；GitHub Release `v2.8.0-dsha.1` 已公开，附件 SHA256 与本地均为 `f3768025d1b003226533b7215625b72ebb20453af73576d6611e41359345b92c`。Node 18 通过/1 个 Linux 专用项在 Windows 跳过；Python 6 通过。用户清空后的手机未重装，首次安装体验由用户自行执行。
