# GitHub 发布

状态：in_progress（2026-10-27）

目标：Fork `9178159/dsh-napcat-bot` 到 `Firefly1007/dsh-napcat-bot-linux`，支持在 DSHA 原生插件市场直接粘贴仓库 URL 安装，并提供 Release tgz。

范围：保留上游提交关系；提交安装必需的源码、210 个固定系统依赖包与豆包客户端；更新安装说明。QQ/NapCat 仍按已有运行逻辑下载。非目标：再次给用户手机安装、修改机器人业务或发布 npm。

约束：仓库归档必须自包含安装所需文件，不依赖 Git LFS 或本地未跟踪资产；保留各资产来源和许可；排除用户数据、密钥、临时下载、Python 缓存及本地 dist。

1. Fork 与本地历史 -> 验证 parent 为原仓库，迁移提交以上游固定提交为父。
2. 仓库内容 -> 验证 210 个 deb、Python 客户端、manifest 与 patch 可用，逐包大小和 SHA256 对应清单。
3. 发布包 -> Node/Python 定向测试通过，真实打包检查通过，归档低于 DSHA 256 MiB 限额。
4. 推送与发布 -> 验证远端 main 对应提交、GitHub archive 内容和 Release 附件；交付可粘贴的仓库 URL。

手机验收证据沿用 dsha-napcat.md；本轮仅验证发布内容，不把远端文件校验称为新的手机安装测试。
