# 插件市场清单协议（manifest.json）

本仓库 code 区的 `manifest.json` 是 PixivDownloader 官方插件市场的**目录清单**：全网客户端来这里拉清单、浏览与安装插件。
本文是它的**格式与安全契约**，生产者（本仓库 CI）与消费者（PixivDownloader 客户端）都以此为准。

`schemaVersion` 当前为 `"1"`。签名对象属于同一开发周期内的协议收敛，**不单独提升 `schemaVersion`**。未知字段会被客户端忽略（可安全新增自定义字段、前向兼容）；已定义字段不得改变语义。

清单本体必须与 detached signature 分离发布：客户端按仓库配置或固定同名规则获取 `manifest.json.sig`，先验证下载到内存的 `manifest.json` 原始字节，再解析 JSON。

---

## 1. 硬约束（消费端强制校验，违反即拒绝）

客户端对清单与插件包做严格校验，下列每条**必须**满足：

1. **严格、合法 JSON（UTF-8）**：解析器**未开启注释**，绝不能有 `//` 或 `/* */`；key 必须**逐字 camelCase**。
2. **清单大小 ≤ 1 MB**。
3. **清单、清单签名、包 URL 与包签名 URL 必须 https + 解析到公网地址**。客户端拦截 loopback、内网、CGNAT、link-local、IPv6 ULA 等地址。
4. **默认直连档禁重定向**：清单、清单签名、包与包签名端点都应**直接 200**，不许 3xx。
    - **例外（仅官方受信仓库）**：客户端对本仓库标记为 `proxy-trusted`，会**经其出站代理**拉取，并对 GitHub 资产 CDN
      （`*.githubusercontent.com`）的 302**按主机白名单跟随至多一跳**。完整性与发布者认证仍由 `expectedSizeBytes`、`sha256` 与 Ed25519 签名兜底，放宽只关 SSRF。
5. **`packageUrl` 路径必须以 `.jar` 或 `.zip` 结尾**。客户端按后缀决定安装类型。
6. **`expectedSizeBytes` 与 `sha256` 必须等于实际下发字节**，逐字节一致；不符 → 拒装、零落盘。单包流式上限 = `min(expectedSizeBytes, 100MB)`。`sha256` 是被签名 envelope 绑定的内容摘要，**不单独承担发布者认证**。
7. **`manifest.json.sig` 必须存在且有效**。客户端必须验证 `manifest.json` 原始字节及 detached signature，禁止解析 JSON 后重新序列化再验签。
8. **不得把 manifest 内部字段当作首次信任入口**。尤其不能先信任 manifest 内写出的签名 URL 再决定 manifest 是否可信；manifest 签名只来自仓库配置或固定同名规则。
9. **本仓库所有包都必须有结构化 `signature` 对象并链到内置官方 trust root**。缺失、字符串旧格式、坏 Base64、未知算法、未知 key、撤销 key、hash mismatch、identity mismatch 或 invalid signature 均 fail-closed。
10. **首版唯一签名算法为 `Ed25519`**。`keyId` 只用于查找 trust root，不能因名称包含 `official` 等字样而自动受信。
11. **`requiredCoreApi`** 应等于该插件 `plugin.properties` 的 `plugin.requires`（取 `major.minor`）。客户端据此判兼容：**major 必须相等、minor ≤ 核心**。当前核心 API = `1.0`。
12. 安装语义：客户端只**下载 → 验 manifest → 校验包字节 → 验包签名 → 落盘 `plugins/` 与 provenance sidecar → 重启或后续生命周期入口生效**。验签失败的包不得进入 PF4J classloader 创建、插件主类实例化或插件 contribution 调用。

> `★` = **安装关键字段**（缺 / 错即拒）；其余字段仅用于展示、检索、排序或诊断，**不参与首次信任决策**。

---

## 2. 规范样例

`manifest.json`：

```json
{
  "schemaVersion": "1",
  "generatedTime": "2026-07-01T00:00:00Z",
  "entries": [
    {
      "pluginId": "stats",
      "displayNameKey": "stats:nav.label",
      "descriptionKey": "stats:plugin.summary",
      "market": {
        "displayName": {
          "zh": "统计",
          "en": "Statistics"
        },
        "summary": {
          "zh": "下载统计仪表盘",
          "en": "Download stats dashboard"
        },
        "description": {
          "zh": "更详细的说明",
          "en": "Longer description"
        },
        "author": "Sywyar",
        "sourceType": "official",
        "category": "utility",
        "tags": [
          "statistics",
          "dashboard"
        ],
        "homepageUrl": "https://github.com/Sywyar/PixivDownloader",
        "license": "MIT",
        "downloadCount": 1820,
        "previousDownloadCount": 3500,
        "totalDownloadCount": 5320,
        "latestVersion": "1.0.0",
        "updatedTime": "2026-07-01T00:00:00Z",
        "iconToken": "chart-line",
        "colorToken": "green",
        "recommended": true,
        "officialRequired": false
      },
      "packages": [
        {
          "version": "1.0.0",
          "packageUrl": "https://github.com/Sywyar/PixivDownloader-plugins/releases/download/stats-v1.0.0/pixivdownload-plugin-stats-1.0.0.jar",
          "expectedSizeBytes": 27675,
          "sha256": "e7cd66fe0123456789abcdef0123456789abcdef0123456789abcdefd397b8",
          "signature": {
            "formatVersion": 1,
            "algorithm": "Ed25519",
            "keyId": "pixivdownloader-official-root-2026-07",
            "value": "base64-detached-artifact-signature"
          },
          "signatureUrl": "https://github.com/Sywyar/PixivDownloader-plugins/releases/download/stats-v1.0.0/pixivdownload-plugin-stats-1.0.0.jar.sig",
          "requiredCoreApi": "1.0",
          "dependencies": [],
          "releasedTime": "2026-07-01T00:00:00Z",
          "changeNotes": [
            "首个外置版本"
          ],
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

---

## 3. 字段参考

### 顶层

| 字段              | 说明                                      |
|-----------------|-----------------------------------------|
| `schemaVersion` | 协议版本，当前 `"1"`。签名对象化不单独提升版本。            |
| `generatedTime` | 清单生成时间（ISO-8601 UTC），用于新鲜度 / 诊断。        |
| `entries[]`     | 插件条目数组。                                 |

### 条目 `entries[]`

| 字段                                  | 说明                                               |
|-------------------------------------|--------------------------------------------------|
| `pluginId` ★                        | 插件唯一 id（小写短横线），安装按它选条目。                         |
| `displayNameKey` / `descriptionKey` | i18n key（已安装 / 官方插件用；未安装浏览时解析不出，回退 `market` 字面）。 |
| `market`                            | 展示 / 检索元数据（见下）。                                  |
| `packages[]`                        | 该插件的版本包数组（见下）。                                   |

### 展示元数据 `market`

| 字段                                        | 说明                                                                                                              |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| `displayName` / `summary` / `description` | `locale→文本` map（未安装浏览时的字面兜底）。                                                                                   |
| `author`                                  | 作者。                                                                                                             |
| `sourceType`                              | `official` / `community`。本仓库发布的官方包应为 `official`。                                                                 |
| `category`                                | `translate` / `download` / `convert` / `notify` / `backup` / `security` / `ui` / `utility`，未知回退 `utility`（`all` 是聚合项、不可用作分类）。 |
| `tags`                                    | 标签数组。                                                                                                           |
| `homepageUrl`                             | 仅 http/https，客户端渲染前净化。                                                                                          |
| `license`                                 | SPDX 许可标识。                                                                                                      |
| `downloadCount`                           | 当前版本的 GitHub Release 资产下载量（生成时快照）。                                                                              |
| `previousDownloadCount`                   | 历史版本累积下载量（跨版本累加，首次生成为 0；由 CI 在版本变化时从旧清单累加得出）。                                                                   |
| `totalDownloadCount`                      | 累积总下载量（`downloadCount + previousDownloadCount`，客户端展示 / 排序用）。                                                    |
| `latestVersion`                           | 最新版本号。                                                                                                          |
| `updatedTime`                             | 最近更新时间。                                                                                                         |
| `iconToken` / `colorToken`                | 受控 token `[a-z][a-z0-9-]{0,39}`，越界回退。                                                                           |
| `recommended` / `officialRequired`        | 布尔展示标记。                                                                                                         |
| `rating` / `ratingCount`                  | **省略**（无评分采集渠道，不假造）。                                                                                            |

### 版本包 `packages[]`

| 字段                    | 说明                                                            |
|-----------------------|---------------------------------------------------------------|
| `version` ★           | 版本号。                                                          |
| `packageUrl` ★        | 包下载地址：https + 末段 `.jar` / `.zip`（见 §6）。                    |
| `expectedSizeBytes` ★ | 实际下发字节数（>0、≤100MB）。                                           |
| `sha256` ★            | 对实际下发字节算的 SHA-256（小写十六进制）；同时被 artifact 签名 envelope 绑定。       |
| `signature` ★         | 结构化包签名对象（见下）。本仓库官方包必须存在且有效；旧字符串格式拒绝。                       |
| `signatureUrl`        | 可选 detached artifact signature URL，当前官方生成规则为 `<packageUrl>.sig`；安装使用已验证 manifest 内的结构化 `signature`。 |
| `requiredCoreApi` ★   | 对应 `plugin.requires` 的 `major.minor`。                              |
| `dependencies`        | 依赖数组（暂为空）。                                                    |
| `releasedTime`        | 发布时间。                                                         |
| `changeNotes`         | 更新说明字符串数组。                                                    |
| `channel`             | `stable` / `beta`（null=stable）。                                |
| `deprecated`          | `true`=下架 / 置灰。                                               |

### 签名对象 `signature` / `manifest.json.sig`

| 字段              | 说明                                                      |
|-----------------|---------------------------------------------------------|
| `formatVersion` | 签名 envelope 格式版本，当前为数字 `1`。                            |
| `algorithm`     | 签名算法，当前唯一合法值为 `Ed25519`。未知算法 fail-closed。              |
| `keyId`         | 信任根 key 标识，只用于查找 trust root；不能根据名称文本推断可信。              |
| `value`         | Base64 编码的 detached Ed25519 签名字节。坏 Base64 或空值均拒绝。       |

---

## 4. 签名 envelope 语义

签名消息只能由 PixivDownloader 主程序内置的签名模块生成，调用方、脚本或仓库文档都不得自行拼接签名消息。

artifact 签名 envelope 绑定：

- 固定 artifact domain separator。
- `formatVersion`。
- `algorithm`。
- `keyId`。
- `pluginId`。
- `version`。
- `artifactSize`。
- 32 字节 SHA-256。

manifest 签名 envelope 使用独立 domain separator，绑定：

- `repositoryId`，本仓库为 `official`。
- manifest 原始字节长度。
- manifest 原始字节 SHA-256。

任意一位被改，包括 artifact bytes、manifest raw bytes、pluginId、version、size、sha256、algorithm、keyId 或 signature value，客户端都必须拒绝。

---

## 5. manifest detached signature 获取与验签顺序

客户端消费官方仓库时按以下顺序执行：

1. 按仓库配置下载 `manifest.json` 到内存。
2. 按固定同名规则下载 `manifest.json.sig`。
3. 使用内置官方 trust root 验证 `manifest.json` 原始字节。
4. 验签成功后才解析 JSON。
5. 校验每个官方包的结构化 `signature` 对象。
6. 下载包字节后，比对 `expectedSizeBytes`、`sha256`，再验证 artifact signature。
7. 验证成功才允许进入安装事务和 PF4J 加载链路。

manifest 内可保留展示或诊断字段，但不得把 manifest 内部读出的 URL 当作 manifest 自身的首次信任入口。

---

## 6. `packageUrl` 与 GitHub Release 重定向

`packageUrl` 直接指向本仓库的 Release 资产稳定链接，形如：

```text
https://github.com/Sywyar/PixivDownloader-plugins/releases/download/<id>-v<version>/<jar>
```

GitHub 对该链接的响应是：`GET 稳定链接` → **302** → `https://release-assets.githubusercontent.com/...`（签名、约 5 分钟过期）→ **200** 字节。稳定链接可入清单；跳转目标短时效、不可入清单。客户端因此对**官方受信仓库**启用“经代理 + 按 `*.githubusercontent.com` 白名单跟随一跳重定向”来取字节（见 §1.4）；取到的字节再由 `expectedSizeBytes`、`sha256` 与 Ed25519 artifact signature 验证。

---

## 7. 版本与不可变性

- **一个 Release = 一个插件的一个版本**；标签 `<id>-v<version>`。
- 每个 Release 至少包含：`<jar>`、`<jar>.sha256`、`<jar>.sig`。
- **已发布资产不可变**：升级请发新版本，绝不覆盖。`sha256` 与签名 envelope 都钉死在清单里，覆盖会让全网校验失败。
- 旧版本 Release 保留，`packages[]` 中的历史版本仍可安装。

---

## 8. 清单如何生成

由主仓库 CI（`generate-market-manifest.ps1`）产出：

- **安装关键字段**从真实产物推导：`sha256` / 大小实算、`requiredCoreApi` 读 jar 内 `plugin.requires`。
- 包签名由签名 CLI 在构建产物冻结后生成，写入 `packages[].signature`，并写出 Release 资产 `<jar>.sig`。
- `signatureUrl` 当前按 `<packageUrl>.sig` 生成，仅作为 detached artifact signature 的定位与诊断字段。
- `downloadCount` 读 GitHub 资产计数快照。
- `previousDownloadCount` 与 `totalDownloadCount` 由 CI 从仓库中已发布的旧清单跨版本累加得出：版本变化时累加旧 `downloadCount + previousDownloadCount`，版本未变则保持旧值。
- **展示字段**取自主仓库的策展源。
- 产出严格 JSON、UTF-8、≤1MB，提交进本仓库 code 区。
- 生成 `manifest.json` 后，必须对其原始字节生成 `manifest.json.sig`，并与清单分离发布。
- 生成与发布流程必须扫描私钥材料；私钥不得进入源码、清单、签名文件、测试报告、日志、构建缓存或 Release 附件。

**请勿手改 `manifest.json` 或 `manifest.json.sig`。**
