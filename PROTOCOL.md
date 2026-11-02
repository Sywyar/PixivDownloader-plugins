# 插件市场清单协议（`manifest.json`）

本文规定官方插件清单 `manifest.json`（及 `nightly-manifest.json`）、独立签名 `.sig` 与历史分片 `history/<pluginId>-<sha256>.json` 的数据结构及客户端强制校验规则。用户安装说明见 [README.md](README.md)。

清单由主仓库发布流程（`scripts/generate-market-manifest.ps1`）自动生成并签名，**禁止手工修改 `manifest.json`、`history/*.json` 或 `.sig` 文件**。

## 1. 客户端强制校验规则（Fail-Closed）

消费端在解析与安装前强制执行以下 12 项检查，任一不满足即拒绝：

1. **严格 UTF-8 JSON**：禁止包含 `//` 或 `/* */` 注释，字段名严格区分大小写（camelCase）。
2. **大小预算**：主清单与单份历史分片默认拉取上限均为 `1 MiB`（`1,048,576` 字节）；单个插件包默认上限为 `192 MiB`（`201,326,592` 字节）。
3. **强制公网 HTTPS**：清单、清单签名、历史分片、插件包与包签名 URL 均须使用 `https://`。直连模式在本地解析 DNS 并拦截回环、私网、CGNAT、link-local 与 IPv6 ULA 地址；`proxy-trusted` 模式在启用应用代理时交由代理解析。
4. **重定向边界**：`direct-strict` 模式禁止任何 `3xx` 重定向；本仓库使用的官方 `proxy-trusted` 策略仅允许在受控 GitHub 资产主机（如 `release-assets.githubusercontent.com`）范围内跟随最多 5 跳 HTTPS 重定向，且每一跳均重新校验。
5. **包后缀约束**：`packageUrl` 路径必须以 `.jar` 或 `.zip` 结尾。
6. **精确字节与摘要匹配**：实际下载字节数必须等于 `expectedSizeBytes`，小写十六进制 SHA-256 必须等于 `sha256`。
7. **先验原始字节签名**：客户端按仓库配置或固定同名规则获取 `manifest.json.sig`，在解析 JSON 前对下载到内存的 `manifest.json` 原始字节验证 Ed25519 签名（严禁先解析 JSON 再重序列化验签，也严禁使用清单内部字段作为清单自身的首次信任入口）。
8. **结构化包签名**：每个包条目必须包含结构化 `signature` 对象并链到客户端内置的官方信任根；缺失、旧字符串格式、非法 Base64、未知/撤销 `keyId` 或签名不匹配均拒绝。
9. **固定签名算法**：`algorithm` 唯一合法值为 `"Ed25519"`，`formatVersion` 为 `1`；`keyId` 仅用于索引信任根，不按名称字面判断可信度。
10. **SDK 兼容声明**：`requiredSdk`（及同值兼容别名 `requiredCoreApi`）取自包内 `plugin.requires`。稳定 SDK 填 `major.minor`（要求宿主 major 相同且 minor 不低于声明值）；预发布 SDK 填 `=完整SDK版本`（要求精确匹配）。
11. **包内描述符复核**：下载并验签冻结字节后，安装器还会核对包内 `plugin.properties` 的真实 ID、版本、SDK 要求与依赖，不一致即拒绝。
12. **事务激活顺序**：下载 → 验清单签名 → 解析清单 → 下载包并校验大小与 SHA-256 → 验包签名 → 校验包内描述符 → 原子写入 `plugins/` 与 `plugins/provenance/` → 按生命周期策略生效。任何一步失败均不会创建 PF4J ClassLoader 或执行插件代码。

## 2. 结构示例

以下示例中的时间、下载量、SHA-256 与签名均为演示占位符：

`manifest.json`：

```json
{
  "schemaVersion": "1",
  "generatedTime": "2026-07-01T00:00:00Z",
  "entries": [
    {
      "pluginId": "stats",
      "displayNamespace": "stats",
      "displayNameKey": "stats:nav.label",
      "descriptionKey": "stats:plugin.summary",
      "market": {
        "displayName": { "zh": "统计", "en": "Statistics" },
        "summary": { "zh": "下载统计仪表盘", "en": "Download stats dashboard" },
        "description": { "zh": "详细说明", "en": "Detailed description" },
        "author": "Sywyar",
        "sourceType": "official",
        "category": "utility",
        "tags": ["statistics", "dashboard"],
        "homepageUrl": "https://github.com/Sywyar/PixivDownloader",
        "license": "AGPL-3.0-only",
        "downloadCount": 1820,
        "previousDownloadCount": 3500,
        "totalDownloadCount": 5320,
        "latestVersion": "1.0.0",
        "updatedTime": "2026-07-01T00:00:00Z",
        "iconToken": "chart-line",
        "colorToken": "green",
        "recommended": true,
        "officialRequired": false,
        "defaultInstalled": true
      },
      "history": {
        "path": "history/stats-0000000000000000000000000000000000000000000000000000000000000000.json",
        "sha256": "0000000000000000000000000000000000000000000000000000000000000000",
        "sizeBytes": 1024,
        "versions": 2
      },
      "packages": [
        {
          "version": "1.0.0",
          "packageUrl": "https://github.com/Sywyar/PixivDownloader-plugins/releases/download/stats-v1.0.0/pixivdownload-plugin-stats-1.0.0.jar",
          "expectedSizeBytes": 27675,
          "sha256": "0000000000000000000000000000000000000000000000000000000000000000",
          "signature": {
            "formatVersion": 1,
            "algorithm": "Ed25519",
            "keyId": "pixivdownloader-official-root-2026-07",
            "value": "base64-detached-artifact-signature"
          },
          "signatureUrl": "https://github.com/Sywyar/PixivDownloader-plugins/releases/download/stats-v1.0.0/pixivdownload-plugin-stats-1.0.0.jar.sig",
          "requiredSdk": "1.0",
          "requiredCoreApi": "1.0",
          "dependencies": [],
          "releasedTime": "2026-07-01T00:00:00Z",
          "changeNotes": ["Initial external release"],
          "channel": "stable",
          "deprecated": false
        }
      ]
    }
  ]
}
```

`manifest.json.sig`：

```json
{
  "formatVersion": 1,
  "algorithm": "Ed25519",
  "keyId": "pixivdownloader-official-root-2026-07",
  "value": "base64-detached-manifest-signature"
}
```

## 3. 字段定义

下表中标 `★` 的字段为安装安全关键字段（缺失或不匹配直接拒绝）；其余字段用于展示、检索、排序或诊断，不授予安装信任或运行时权限。

### 顶层与插件条目 `entries[]`

| 字段 | 说明 |
| --- | --- |
| `schemaVersion` | 顶层协议版本字符串，当前固定为 `"1"`（客户端忽略未知扩展字段） |
| `generatedTime` | 顶层清单生成时间（ISO-8601 UTC） |
| `entries[]` | 插件条目数组 |
| `pluginId` ★ | 插件唯一标识（小写短横线），必须与包内 `plugin.id` 一致 |
| `displayNamespace` / `displayNameKey` / `descriptionKey` | 插件展示文案的 i18n namespace 与 key（未安装浏览时回退 `market` 多语言文本） |
| `market` | 市场展示与检索元数据对象 |
| `packages[]` | 主清单直接携带的候选插件包数组 |
| `history` | 可选的历史版本分片引用对象 |

### 展示元数据 `market`

| 字段 | 说明 |
| --- | --- |
| `displayName` / `summary` / `description` | `locale -> 文本` 映射（未安装时的本地化展示兜底） |
| `author` / `sourceType` / `license` | 作者名称、来源类型（本仓库为 `"official"`）、SPDX 许可证标识 |
| `category` | `translate` / `download` / `convert` / `notify` / `backup` / `security` / `ui` / `utility`（未知值回退 `utility`） |
| `tags` / `homepageUrl` | 标签数组、HTTP(S) 主页地址 |
| `downloadCount` / `previousDownloadCount` / `totalDownloadCount` | 当前版本 Release 资产下载快照、历史版本累计下载量、总下载量 |
| `latestVersion` / `updatedTime` | 最新版本号与最近更新时间 |
| `iconToken` / `colorToken` | 受控图标与颜色 token（`[a-z][a-z0-9-]{0,39}`） |
| `recommended` / `officialRequired` / `defaultInstalled` | 布尔展示标记（不替代宿主的运行时必选策略） |
| `defaultLocale` / `links` / `icon` / `screenshots` | 可选市场扩展展示资源 |

### 插件包 `packages[]` 与历史分片 `history`

| 字段 | 说明 |
| --- | --- |
| `version` ★ | 插件版本号，必须与包内 `plugin.version` 一致 |
| `packageUrl` ★ | 插件包稳定 HTTPS 下载地址（`https://github.com/Sywyar/PixivDownloader-plugins/releases/download/<id>-v<version>/<jar>`） |
| `expectedSizeBytes` ★ | 插件包精确字节数（正整数） |
| `sha256` ★ | 插件包原始字节的 64 位小写十六进制 SHA-256 |
| `signature` ★ | 结构化 Ed25519 签名对象（`formatVersion: 1`、`algorithm: "Ed25519"`、`keyId`、Base64 `value`） |
| `signatureUrl` | 独立签名附件地址（`<packageUrl>.sig`），供定位与诊断；安装验签使用已验签清单内的 `signature` |
| `requiredSdk` / `requiredCoreApi` | 所需 SDK 版本声明（两者同值） |
| `dependencies` | 从 `plugin.dependencies` 提取的依赖表达式数组 |
| `releasedTime` / `changeNotes` / `channel` / `deprecated` | 发布时间、更新说明数组、渠道（`stable` / `alpha` / `beta` / `rc` / `nightly`）与弃用展示标记 |
| `identityMigrationSignatures` / `content` | 可选身份迁移授权签名、版本级静态文档与截图附件引用（带独立大小与 SHA-256 约束） |
| `history.path` / `sha256` / `sizeBytes` / `versions` | 历史分片同源相对路径 `history/<pluginId>-<sha256>.json`、64 位小写 SHA-256、字节数（≤ 1 MiB）与版本数（≤ 900） |

主清单仅保留各插件的最新版本、最新稳定版本，以及固定旧客户端集合（SDK `1.0.0`、`1.1.0`）各自的最新兼容稳定版本；其余历史版本写入不嵌套 `history` 的 `history/<pluginId>-<sha256>.json` 分片并由主清单签名绑定。

## 4. 签名 Envelope 与绑定域

签名只能由主仓库 `pixivdownload-plugin-signature` CLI 生成，各签名域使用独立的 domain separator：

- **插件包签名（`artifact`）**：绑定固定 artifact 域标识、`formatVersion`、`algorithm`、`keyId`、`pluginId`、`version`、`artifactSize` 与 32 字节原始 SHA-256。
- **清单签名（`manifest`）**：绑定固定 manifest 域标识、`repositoryId`（本仓库固定为 `"official"`）、清单原始字节长度与清单原始字节 SHA-256。

## 5. 发布与版本不可变性

- 每个插件版本对应一个 `<pluginId>-v<version>` Release，至少包含 `<jar>`、`<jar>.sha256` 和 `<jar>.sig`。
- 常规更新必须递增 `plugin.version`。仅主仓库显式手动 `-f` 修复入口允许同版本更新官方插件，且必须同步重算包、签名、清单与版本文档（此例外不适用于 SDK）。
- 生成与发布流程强制执行私钥泄露扫描，私钥绝不进入仓库、清单、签名、日志或 Release 附件。
