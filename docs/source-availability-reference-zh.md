# 源 / 图床可用性参考（跨仓库实测汇总）

> 适用范围：本仓库（JavBoss-fork）以**图像功能增强**为主线（女优头像 / 厂牌高清艺术照 / 样品图修复 / 播放）。
> 样品图自愈脚本 `fix_sample_thumbs.py` 的判定逻辑是：**后端（NAS）能否抓到源 URL**——抓不到即 502 破图。
> 因此「在受限出口下，哪些源 / 图床实际抓不到图」直接决定了样品图修复的坏源黑名单。
>
> 本文件汇总本项目既有结论，并吸收跨仓库项目 `luckwalter/JavSP-fork` 在受限出口下的渠道实测
> （来源：`javsp/web/health.py` 的 `SOURCE_STATUS_OVERRIDE` 与 `DOMAIN_HINTS`，复核于 2026-10-11）。

## 一、图床可达性（本项目既有 + JavSP 印证）

| 图床 / 域名 | NAS 后端能否抓取 | 说明 |
|---|---|---|
| `pics.dmm` | ✅ 可抓 | DMM 图床 |
| `awsimgsrc.dmm` | ✅ 可抓 | DMM 图床 |
| `image.mgstage` | ✅ 可抓 | MGS 图床 |
| `c0.jdbstatic` | ❌ 不可抓 | Cloudflare 挡 NAS 两条路径之一 |
| `javbus` 图片 | ❌ 不可抓 | Cloudflare 挡；且站点已加驾照/年龄验证（见下） |

## 二、源 / 站点现状（实测，2026-10-11）

| 源 | 现状结论 | 对本项目图像增强的影响 | 处置 |
|---|---|---|---|
| **javbus** | 要求驾照/年龄验证（`driver-verify`），任意番号返回同一验证页；已配 Cookie 也过不去 | 该源图片 URL 必然 502 | 已列入坏源黑名单（PROVIDER_HOSTS） |
| **avsox** | 已改纯前端渲染，任意路径返回壳页，取不到任何静态数据；图床 `file.netcdn.space` | 无静态图可抓 | **新增**入黑名单（2026-10-11） |
| **avwiki** | 本就不是番号库（女优名字查询站），无影片详情页 | 对影片封面/样品图无意义 | **新增**入黑名单（2026-10-11） |
| **fc2fan** | 站点已关停（主站仅返回壳页） | 已废 | 本项目无此 provider，无需处理 |
| javdb / javmenu / javdatabase | 受限出口下可达 | 图床基本可用（javdb 的 jdbstatic 除外，见上） | 维持现状，由动态探测解封/保底 |

## 三、对样品图自愈脚本的影响

`fix_sample_thumbs.py` 的 `PROVIDER_HOSTS`（静态坏名单，保底永远参与判定）现已含：
`javbus` / `javdb`(+`jdbstatic`) / `javmoo` / `javmenu` / `xcity` / **`avsox`(+`file.netcdn.space`)** / **`avwiki`(+`av-wiki.net`)**。

动态判定（`GET /jav/providers` + 必要时的 `POST .../availability` 主动探测）只对名单内源做解封，
已废源探测必然失败 → 保持坏。任何 API 异常回退纯静态名单，行为只「少修」不「误删」，幂等不变。

## 四、参考源

- JavSP-fork：`javsp/web/health.py` 的 `SOURCE_STATUS_OVERRIDE`（javbus=驾照验证 / avsox=前端渲染 / avwiki=非番号库 / fc2fan=关停）
- 本项目：JavBoss 上游 #357 数据源检测（`GET /jav/providers`）提供动态可用性基础
- 注：JavSP 的「熔断器 / 状态细分」属主功能运维增强，**不在本项目借鉴范围**（项目边界：图像增强为主，不动主功能）
