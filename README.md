# PixivDownloader 官方插件仓库

本仓库为 PixivDownloader 插件市场提供官方外置插件包与已签名目录清单。插件源码、测试与发布脚本位于 [PixivDownloader 主仓库](https://github.com/Sywyar/PixivDownloader)；清单协议与消费端校验规则见 [PROTOCOL.md](PROTOCOL.md)。

## 1. 安装与更新插件

在 PixivDownloader 的「插件市场」中选择插件，查看所需 SDK 版本、依赖与更新说明后点击安装。客户端在提交安装事务前会依次校验清单 Ed25519 签名、插件包大小、SHA-256 摘要与插件包签名；界面提示重启时，请按插件生命周期要求重启后端或完整退出重开程序。

这些插件 JAR 依赖宿主提供的 SDK 与运行环境，不可作为独立 Java 程序直接运行。下载或校验异常请保留错误码与插件版本号，前往 [主仓库 Issues](https://github.com/Sywyar/PixivDownloader/issues) 反馈。

## 2. 仓库内容与网络策略

| 路径或位置 | 用途 |
| --- | --- |
| `manifest.json`、`manifest.json.sig` | 正式版应用使用的官方插件主清单及独立 Ed25519 签名 |
| `nightly-manifest.json`、`nightly-manifest.json.sig` | Nightly 应用使用的同批插件清单及独立签名 |
| `history/<pluginId>-<sha256>.json` | 主清单按摘要与大小绑定的历史版本分片（单文件上限 900 个版本 / 1 MiB） |
| GitHub Releases | 各插件版本的 `<jar>`、`<jar>.sha256`、`<jar>.sig` 与市场文档/截图附件 |

正式版默认清单地址为 `https://raw.githubusercontent.com/Sywyar/PixivDownloader-plugins/master/manifest.json`。官方仓库使用 `proxy-trusted` 策略：应用开启全局代理时走该代理，仅允许在受控 GitHub 资产主机范围内跟随最多 5 跳 HTTPS 重定向，并逐跳复核目标主机。

## 3. 版本发布与不可变性

- **版本与标签**：常规发布按插件独立编号，Git 标签格式为 `<pluginId>-v<version>`；Nightly 使用独立滚动清单。
- **生成与签名**：主仓库发布工作流构建并冻结插件包字节，生成 Ed25519 签名与 Release 附件，再由 `scripts/generate-market-manifest.ps1` 提取真实包描述符与策展信息生成清单及 `.sig`（通过主仓库 `release` Environment 的 `CROSS_REPO_RELEASE_TOKEN` 写入本仓库）。**严禁手动编辑 `manifest.json`、`history/` 或 `.sig` 文件。**
- **同版本修复例外**：日常更新必须提升插件版本号。仅主仓库显式手动 `-f` 修复入口允许同版本替换官方插件产物（须同步重算包、签名、清单与版本文档；该修复不会触发普通更新角标，用户需按说明重新安装，且该例外不适用于 SDK）。
- **许可协议**：本仓库目录与元数据见 [LICENSE](LICENSE)，各插件许可以其描述符及主仓库声明为准。

## English Summary

This repository distributes official external plugins for PixivDownloader via Ed25519-signed catalogs (`manifest.json`, `nightly-manifest.json`, and digest-bound `history/*.json` shards) and GitHub Release assets. Install and update plugins through the in-application Plugin Marketplace; these JARs require the host runtime and cannot run standalone.

Plugin sources and release scripts live in the [PixivDownloader main repository](https://github.com/Sywyar/PixivDownloader). Never edit `manifest.json` or `.sig` files manually. Normal updates increment the plugin version; the explicit manual `-f` repair workflow is the only same-version replacement path for official plugins and does not trigger a standard update badge. See [PROTOCOL.md](PROTOCOL.md) for the full catalog and verification contract.
