# 插件市场清单协议（manifest.json）

本仓库 code 区的 `manifest.json` 是 PixivDownloader 插件市场的**目录清单**:全网客户端来这里拉清单、浏览与安装插件。
本文是它的**格式与安全契约**——生产者(本仓库 CI)与消费者(PixivDownloader 客户端)都以此为准。

`schemaVersion` 当前为 `"1"`。**未知字段被忽略**(可安全新增自定义字段、前向兼容);已定义字段不得改变语义。

---

## 1. 硬约束(消费端强制校验,违反即拒绝)

客户端对清单与插件包做严格校验,下列每条**必须**满足:

1. **严格、合法 JSON(UTF-8)**:解析器**未开启注释**——绝不能有 `//` 或 `/* */`;key 必须**逐字 camelCase**。
2. **清单大小 ≤ 1 MB**。
3. **清单与包 URL 必须 https + 解析到公网地址**(客户端拦截 loopback / 内网 / CGNAT / link-local / IPv6 ULA 等)。
4. **默认直连档禁重定向**:清单与包端点都应**直接 200**,不许 3xx。
    - **例外(仅官方受信仓库)**:客户端对本仓库标记为 `proxy-trusted`,会**经其出站代理**拉取,并对 GitHub 资产 CDN
      (`*.githubusercontent.com`)的 302**按主机白名单跟随至多一跳**——这正是 `packageUrl` 用 GitHub Release 链接的前提
      (见 §4)。完整性仍由 §1.6 的 sha256/size 兜底,放宽只关 SSRF。
5. **`packageUrl` 路径必须以 `.jar` 或 `.zip` 结尾**(客户端按后缀决定安装类型)。
6. **`expectedSizeBytes` 与 `sha256` 必须等于实际下发字节**,逐字节一致;不符 → 拒装(`REJECTED_INTEGRITY`)、零落盘。
   单包流式上限 = `min(expectedSizeBytes, 100MB)`。
7. **`signature` 必须省略 / 留空**:客户端当前无签名校验器,**非空即 fail-closed 拒装**。
8. **`requiredCoreApi`** 应等于该插件 `plugin.properties` 的 `plugin.requires`(取 `major.minor`)。客户端据此判兼容:
   **major 必须相等、minor ≤ 核心**。当前核心 API = `1.0`。
9. 安装语义:客户端只**下载 → 校验 → 落盘 `plugins/` → 重启生效**,**不热加载**。

> `★` = **安装关键字段**(缺 / 错即拒);其余字段仅用于展示 / 检索 / 排序,**不参与安全决策**。

---

## 2. 规范样例

```json
{
  "schemaVersion": "1",
  "generatedTime": "2026-06-27T00:00:00Z",
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
        "updatedTime": "2026-06-20T10:00:00Z",
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
          "sha256": "e7cd66fe…d397b8",
          "requiredCoreApi": "1.0",
          "dependencies": [],
          "releasedTime": "2026-06-20T10:00:00Z",
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

---

## 3. 字段参考

### 顶层

| 字段              | 说明                               |
|-----------------|----------------------------------|
| `schemaVersion` | 协议版本,当前 `"1"`。                   |
| `generatedTime` | 清单生成时间(ISO-8601 UTC),用于新鲜度 / 诊断。 |
| `entries[]`     | 插件条目数组。                          |

### 条目 `entries[]`

| 字段                                  | 说明                                               |
|-------------------------------------|--------------------------------------------------|
| `pluginId` ★                        | 插件唯一 id(小写短横线),安装按它选条目。                          |
| `displayNameKey` / `descriptionKey` | i18n key(已安装 / 官方插件用;未安装浏览时解析不出,回退 `market` 字面)。 |
| `market`                            | 展示 / 检索元数据(见下)。                                  |
| `packages[]`                        | 该插件的版本包数组(见下)。                                   |

### 展示元数据 `market`

| 字段                                        | 说明                                                                                                              |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| `displayName` / `summary` / `description` | `locale→文本` map(未安装浏览时的字面兜底)。                                                                                   |
| `author`                                  | 作者。                                                                                                             |
| `sourceType`                              | `official` / `community`。                                                                                       |
| `category`                                | `translate`/`download`/`convert`/`notify`/`backup`/`security`/`ui`/`utility`,未知回退 `utility`(`all` 是聚合项、不可用作分类)。 |
| `tags`                                    | 标签数组。                                                                                                           |
| `homepageUrl`                             | 仅 http/https,客户端渲染前净化。                                                                                          |
| `license`                                 | SPDX 许可标识。                                                                                                      |
| `downloadCount`                           | 当前版本的 GitHub Release 资产下载量（生成时快照）。                                                                              |
| `previousDownloadCount`                   | 历史版本累积下载量（跨版本累加，首次生成为 0；由 CI 在版本变化时从旧清单累加得出）。                                                                   |
| `totalDownloadCount`                      | 累积总下载量（`downloadCount + previousDownloadCount`，客户端展示 / 排序用）。                                                    |
| `latestVersion`                           | 最新版本号。                                                                                                          |
| `updatedTime`                             | 最近更新时间。                                                                                                         |
| `iconToken` / `colorToken`                | 受控 token `[a-z][a-z0-9-]{0,39}`,越界回退。                                                                           |
| `recommended` / `officialRequired`        | 布尔展示标记。                                                                                                         |
| `rating` / `ratingCount`                  | **省略**(无评分采集渠道,不假造)。                                                                                            |

### 版本包 `packages[]`

| 字段                    | 说明                                    |
|-----------------------|---------------------------------------|
| `version` ★           | 版本号。                                  |
| `packageUrl` ★        | 包下载地址:https + 末段 `.jar`/`.zip`(见 §4)。 |
| `expectedSizeBytes` ★ | 实际下发字节数(>0、≤100MB)。                   |
| `sha256` ★            | 对实际下发字节算的 SHA-256(小写十六进制)。            |
| `requiredCoreApi` ★   | 对应 `plugin.requires` 的 `major.minor`。 |
| `signature`           | **省略**(非空即 fail-closed 拒装)。           |
| `dependencies`        | 依赖数组(暂为空)。                            |
| `releasedTime`        | 发布时间。                                 |
| `changeNotes`         | 更新说明字符串数组。                            |
| `channel`             | `stable` / `beta`(null=stable)。       |
| `deprecated`          | `true`=下架 / 置灰。                       |

---

## 4. `packageUrl` 与 GitHub Release 重定向

`packageUrl` 直接指向本仓库的 Release 资产稳定链接,形如:

```
https://github.com/Sywyar/PixivDownloader-plugins/releases/download/<id>-v<version>/<jar>
```

GitHub 对该链接的响应是:`GET 稳定链接` → **302** → `https://release-assets.githubusercontent.com/...`(签名、约 5 分钟
过期)→ **200** 字节。稳定链接可入清单;跳转目标短时效、不可入清单。客户端因此对**官方受信仓库**启用「经代理 + 按
`*.githubusercontent.com` 白名单跟随一跳重定向」来取字节(见 §1.4);取到的字节再由 `sha256`/`expectedSizeBytes`
**逐字节校验**——完整性不依赖传输方,即便重定向目标被篡改也会被拒。

---

## 5. 版本与不可变性

- **一个 Release = 一个插件的一个版本**;标签 `<id>-v<version>`,资产 = `<jar>` + `<jar>.sha256`。
- **已发布资产不可变**:升级请发新版本,绝不覆盖——`sha256` 钉死在清单里,覆盖会让全网校验失败。
- 旧版本 Release 保留,`packages[]` 中的历史版本仍可安装。

---

## 6. 清单如何生成

由主仓库 CI(`generate-market-manifest.ps1`)产出:**安装关键字段**从真实产物推导(`sha256`/大小实算、`requiredCoreApi`
读 jar 内 `plugin.requires`)、`downloadCount` 读 GitHub 资产计数快照、`previousDownloadCount` 与 `totalDownloadCount`
由 CI 从仓库中已发布的旧清单跨版本累加得出（版本变化时累加旧 `downloadCount + previousDownloadCount`，版本未变则保持旧值）；
**展示字段**取自主仓库的策展源;产出严格 JSON、
UTF-8、≤1MB,提交进本仓库 code 区。**请勿手改 `manifest.json`**。
