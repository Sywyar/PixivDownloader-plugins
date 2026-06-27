# PixivDownloader-plugins

PixivDownloader 官方**外置插件**的分发仓库。这里存放两样东西:**① 由 CI 自动构建并发布的插件产物**(瘦身 PF4J
插件 jar,以 GitHub Release 逐版本发布);**② CI 生成并提交进 code 区的 `manifest.json`**(插件市场清单)。本仓库
**不含插件源码**——源码都在主仓库 [Sywyar/PixivDownloader](https://github.com/Sywyar/PixivDownloader)。

> English: distribution repo for PixivDownloader's official external plugins. It hosts the built thin PF4J
> jars as GitHub Releases, plus a CI-generated `manifest.json` (the market catalog) committed to the repo.
> No plugin source here. Consumed by the in-app plugin market — not for manual download.

清单的格式与安全契约见 [`PROTOCOL.md`](PROTOCOL.md)。

## 这是怎么被消费的

- **清单**:PixivDownloader 客户端读取本仓库 code 区的 `manifest.json`(经
  `https://raw.githubusercontent.com/Sywyar/PixivDownloader-plugins/master/manifest.json`,200 直出、无重定向),
  据此在「插件市场」里展示可装插件。
- **安装**:清单里的 `packageUrl` 直接指向本仓库的 **Release 资产链接**。客户端下载时**经它自己配置的代理**、并对
  GitHub 资产 CDN(`*.githubusercontent.com`)的重定向**按白名单跟随一跳**取字节,再用清单里钉死的 `sha256` / 大小
  **逐字节校验**后落盘、重启生效。
- 也就是说,**Release 链接确实是下载源,但请通过 PixivDownloader 内置的「插件市场」安装**(它负责代理、重定向白名单与
  完整性校验);不要手动下载瘦身 jar 直接丢进 `plugins/`(缺宿主以 `provided` 提供的依赖,无法独立运行)。

## Release 约定

- **一个 Release = 一个插件的一个版本。** 标签形如 `stats-v1.0.0`。
- 资产(assets):
  - `pixivdownload-plugin-<id>-<version>.jar` —— 瘦身插件包(根含 `plugin.properties`,不内嵌 PF4J / Spring / 契约类)。
  - `pixivdownload-plugin-<id>-<version>.jar.sha256` —— 对应 SHA-256 校验文件。
- **产物一经发布即不可变。** 升级请发新版本,绝不覆盖已发布资产——`manifest.json` 钉死了每个版本的 `sha256` / 字节数,
  覆盖会导致全网校验失败(`REJECTED_INTEGRITY`)。旧版本 Release 保留,清单里的历史版本仍可安装。

## manifest.json

由主仓库 CI 在发版时生成并**提交进 code 区**(请勿手改):安装关键字段(`version` / `packageUrl` / `sha256` /
`expectedSizeBytes` / `requiredCoreApi`)取自真实产物;`downloadCount` 取自 GitHub 资产 download_count 的生成时快照
(客户端打开市场时由后端再 best-effort 拉一次实时值、失败回退快照);展示字段(名称 / 简介 / 分类 / 标签 / 图标等)
取自主仓库的策展源。字段定义与硬约束详见 [`PROTOCOL.md`](PROTOCOL.md)。

## 发布流程(CI)

主仓库工作流(`publish-plugins`):构建 reactor → `assemble-plugin-distribution.ps1` 产出瘦身 jar + `sha256` → 把官方可选
插件逐版本发布到本仓库的 Release → `generate-market-manifest.ps1` 生成 `manifest.json` 并提交进本仓库 code 区。跨仓库
发布使用具备本仓库 **Contents: 读写** 权限的细粒度 PAT(存为主仓库 Actions secret `PLUGINS_REPO_TOKEN`)。

## 许可

各插件许可以其自身 `plugin.properties` / 主仓库声明为准。
