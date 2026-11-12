# singbox-adblock-rules

基于真机抓包整理的 sing-box 广告规则集。

**当前覆盖 App**：酷安 `com.coolapk.market`、懂球帝 `com.dongqiudi.news`

## 直接订阅

```json
{
  "route": {
    "rule_set": [
      {
        "type": "remote",
        "tag": "ads",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/wvv666/singbox-adblock-rules/main/rulesets/lite/all.srs",
        "update_interval": "1d"
      }
    ],
    "rules": [
      { "action": "sniff" },
      { "protocol": "dns", "action": "hijack-dns" },
      { "rule_set": "ads", "action": "reject", "method": "default" }
    ]
  },
  "dns": {
    "rules": [
      { "rule_set": "ads", "action": "reject", "method": "default" }
    ]
  },
  "experimental": {
    "cache_file": { "enabled": true, "path": "cache.db" }
  }
}
```

> 远程规则集需要 `experimental.cache_file.enabled: true` 才会缓存。

## 本地使用

下载 `.srs` 到本地，改用 local 类型：

```json
{
  "type": "local",
  "tag": "ads",
  "format": "binary",
  "path": "/etc/sing-box/ruleset/ads.srs"
}
```

```bash
curl -L -o /etc/sing-box/ruleset/ads.srs \
  https://raw.githubusercontent.com/wvv666/singbox-adblock-rules/main/rulesets/lite/all.srs
```

## ⚠ 必须加 `{"action": "sniff"}`

不加的话，mixed/socks 入站的 UDP DNS 不会被 `hijack-dns` 接管，
`dns.rules` 整段失效，查询直奔上游 —— 表现为"规则写对了但完全不管用"。

另外 `route.default_domain_resolver` 必须配置，sing-box 1.14 起缺失会直接 FATAL。

## 规则集一览

| 文件 | 条数 | 说明 |
|---|---|---|
| `rulesets/lite/all.{json,srs}` | 33 | 全库纯广告域名，**推荐** |
| `rulesets/full/all.{json,srs}` | 51 | 加 SDK 底座、埋点上报 |
| `rulesets/lite/coolapk.{json,srs}` | 20 | 仅酷安 |
| `rulesets/lite/dongqiudi.{json,srs}` | 26 | 仅懂球帝 |

**lite** — 纯广告 SDK、竞价接口、广告素材 CDN。不影响 App 任何功能。

**full** — 额外含聚合底座、崩溃统计、埋点上报。拦了不影响使用，但 App 的统计会静默失败。

## 内容说明

规则集以 `domain_suffix` 为主，语义是"该域名及其所有子域"。
例如 `pangolin-sdk-toutiao.com` 会覆盖 `api-access.pangolin-sdk-toutiao.com`。

已尽量只收广告域名，业务域名（API、图片、头像、视频、支付、推送）均未收录。

## 这些规则拦了什么

以酷安为例，绝大部分广告流量来自穿山甲（字节）聚合 SDK：

| 域名 | 说明 |
|---|---|
| `api-access.pangolin-sdk-toutiao.com` | 拉取广告 `/get_ads/`、聚合竞价 `/mediation/exchange/` |
| `api-access.pangolin-sdk-toutiao1.com` | 统计上报 `/stats/batch/` |
| `log-api.pangolin-sdk-toutiao*.com` | 日志上报 |
| `pglstatp-toutiao.com` | CSJ 策略中心、广告模板、广告视频分片 |
| `douyinpic.com` / `ecombdimg.com` | 广告素材图 |
| `analytics.oceanengine.com` | 巨量引擎 |

酷安没有自有广告平台，拦掉穿山甲后广告位基本空白，不会出现加载等待。


## 域名归类说明

### 广点通 / 腾讯优量汇

| 域名 | 用途 |
|---|---|
| `gdt.qq.com` | 竞价回调 `win_notice.fcg`，含 `win.gdt.qq.com` |
| `e.qq.com` | 曝光/点击上报 |
| `pgdt.gtimg.cn` | 广点通素材 CDN |

### Sigmob

`_sigmob.cn` / `_sigmob.com`，含 `tm.sigmob.cn`（`/waterfall/v1`、`/strategy/v6`）与 `rtbcallback.sigmob.cn`

### 穿山甲 / 字节

| 域名 | 用途 |
|---|---|
| `pangolin-sdk-toutiao.com` | 主接口，`/get_ads/`、`/mediation/exchange/` |
| `pangolin-sdk-toutiao1.com` | `/stats/batch/`、`/callstack/batch/` |
| `pangolin-sdk-toutiao-b.com` | 日志 |
| `pglstatp-toutiao.com` | CSJ 策略中心、广告模板、素材分片 |
| `webcast-open.douyin.com` | 穿山甲配置接口 |
| `apps.bytesfield.com` | 字节 SDK 包信息 |

### 懂球帝自有广告平台

| 域名 | 用途 |
|---|---|
| `ap.dongqiudi.com` | `/plat/v3` `/plat/v4` 广告决策 |
| `dab.dongqiudi.com` | `/track/v3` `/imp/v3` `/click/v3` 广告埋点 |

### 其他

| 域名 | 用途 |
|---|---|
| `mobads.baidu.com` / `cpro.baidustatic.com` | 百度 mobads |
| `api.touch-moblie.com` | 触漫广告 SDK |
| `adprolink.xwpbj.com` | 聚合广告 |
| `dmp-collect.xdmssp.com` | 数据采集 |
| `xdrun.com` | `fapi.xdrun.com` 上报上游 |
| `g.cn.miaozhen.com` | 秒针监测 |
| `gdfp.gifshow.com` | 快手广告 |
| `yunxish.com` | 云曦聚合底座 |
| `iggrowth.cn` | 聚合配置 |
| `sdklog.zhangyuyidong.cn` | SDK 日志 |
| `js.a.kspkg.com` | 快手 SDK |

## 编译

想自己修改 JSON 后重新编译：

```bash
sing-box rule-set compile --output ads.srs ads.json
```

验证规则是否按预期命中：

```bash
# 注意：命中信息输出到 stderr，且退出码恒为 0
sing-box rule-set match -f binary ads.srs api-access.pangolin-sdk-toutiao.com
```

## 已知陷阱

整理过程中踩过的坑，自建规则集时同样适用：

1. **`rule-set match` 必须加 `-f binary`** 才能读 `.srs`，否则报 `invalid character 'S'`
2. **`rule-set match` 的命中信息走 stderr**，退出码恒为 0，不能只看 stdout 或 `$?`
3. **`domain_suffix` 是后缀匹配**：`gdt.qq.com` 自动覆盖 `win.gdt.qq.com`，不要重复列
4. **`domain_suffix` 不要写前导点**：`.a.com` 不匹配 `a.com` 本身
5. **`domain_regex` 不加锚点**：`ad` 会匹配一切含 ad 的域名，等于 keyword
6. **空规则编译能过但启动 FATAL**：`{"type":"logical","rules":[]}` 报 `missing conditions`，
   写完必须跑 `sing-box check`
7. **字段生效范围取决于引用方**：规则集被路由规则引用时不认 `query_type`（直接 FATAL），
   被 DNS 规则引用时 `port` 条件静默失效

## 免责

规则集基于公开的广告 SDK 域名特征整理，仅用于个人网络环境减少广告流量。
不同 App 版本、不同时间的广告域名会变化，规则可能失效或误伤，请自行验证。

## License

MIT
