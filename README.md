# singbox-adblock-rules

基于真机抓包整理的 sing-box 广告规则集。

**已覆盖 7 个 App**：酷安、懂球帝、QQ邮箱、哔哩哔哩、一刻相册、虎扑、知乎

## 直接订阅

```json
{
  "route": {
    "rule_set": [{
      "type": "remote",
      "tag": "ads",
      "format": "binary",
      "url": "https://raw.githubusercontent.com/wvv666/singbox-adblock-rules/main/rulesets/lite/all.srs",
      "update_interval": "1d"
    }],
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

> 远程规则集必须开 `experimental.cache_file.enabled`，否则不缓存、每次启动重下。

## 本地使用

```bash
curl -L -o ads.srs \
  https://raw.githubusercontent.com/wvv666/singbox-adblock-rules/main/rulesets/lite/all.srs
```

```json
{
  "type": "local",
  "tag": "ads",
  "format": "binary",
  "path": "/etc/sing-box/ruleset/ads.srs"
}
```

## ⚠ 两个必配项

**① `{"action": "sniff"}` 不能少**

不加的话，mixed/socks 入站的 UDP DNS 不会被 `hijack-dns` 接管，
`dns.rules` 整段失效，查询直奔上游 —— 表现为"规则写对了但完全不管用"。

**② `route.default_domain_resolver` 必写**

sing-box 1.14 起缺失会直接 FATAL 退出。

## 规则集一览

| 文件 | 条数 | 说明 |
|---|---|---|
| `rulesets/lite/all.{json,srs}` | 57 | 全部纯广告域名，**推荐** |
| `rulesets/full/all.{json,srs}` | 104 | 加 SDK 底座与埋点上报 |
| `rulesets/lite/coolapk.{json,srs}` | 31 | 酷安 |
| `rulesets/lite/yike.{json,srs}` | 22 | 一刻相册 |
| `rulesets/lite/dongqiudi.{json,srs}` | 20 | 懂球帝 |
| `rulesets/lite/qqmail.{json,srs}` | 8 | QQ邮箱 |
| `rulesets/lite/zhihu.{json,srs}` | 4 | 知乎 |
| `rulesets/lite/hupu.{json,srs}` | 4 | 虎扑 |
| `rulesets/lite/bilibili.{json,srs}` | 2 | 哔哩哔哩 |

**lite** — 纯广告域名（含 App 自有广告平台）。不影响 App 任何功能。

**full** — 额外含聚合底座、崩溃统计、行为埋点。拦了不影响使用，但 App 的统计会静默失败。

只需要某个 App 就下对应的单文件，比全库更精准。

## 各 App 的广告结构

抓包分析后发现，能否用域名规则拦截，取决于广告是否跑在独立域名上：

| App | 广告载体 | 拦截效果 |
|---|---|---|
| **一刻相册** | 8 家第三方 SDK（倍孜/阿里妈妈/穿山甲/快手等） | ✓✓ 21 个独立域 |
| **酷安** | 穿山甲为主 | ✓✓ 占 92% 流量 |
| **QQ邮箱** | 广点通（UA 里 `GDTADNetClient`） | ✓✓ |
| **懂球帝** | 第三方 + 自有 `ap.`/`dab.` | ✓✓ |
| **虎扑** | 自有 `goblin.hupu.com`（独立子域） | ✓✓ |
| **知乎** | 业务接口内联（`ad_interval`、`X-AD` 头） | △ 只能断追踪 |
| **哔哩哔哩** | 自有 `cm.bilibili.com` | △ 只能断归因 |

**规律**：广告只要在独立子域，哪怕是自有的也能拦；广告和业务挤在同一个子域就没法用域名规则区分（网易邮箱的 `appconf.mail.163.com/mmad` 就是这种情况，故未收录）。

## 域名归类说明

### 穿山甲 / 字节

| 域名 | 用途 |
|---|---|
| `pangolin-sdk-toutiao.com` | `/get_ads/`、`/mediation/exchange/` |
| `pangolin-sdk-toutiao1.com` | `/stats/batch/`、`/callstack/batch/` |
| `pglstatp-toutiao.com` | CSJ 策略中心、广告模板、素材分片 |
| `douyinpic.com` / `ecombdimg.com` | 广告素材图 |
| `analytics.oceanengine.com` | 巨量引擎 |

### 广点通 / 腾讯优量汇

| 域名 | 用途 |
|---|---|
| `gdt.qq.com` | 竞价回调 `win_notice.fcg`、样式工厂 `style_factory`、`gdt_click`/`gdt_stats` |
| `e.qq.com` / `sdk.e.qq.com` | 曝光点击上报 |
| `sdkreport.e.qq.com` | `GDTADNetClient` SDK 事件 |
| `tangram.e.qq.com` | 广告配置下发 |

### 百度

| 域名 | 用途 |
|---|---|
| `tcbox.baidu.com` | 百度联盟 `/ztbox?action=zubc` |
| `mbd.baidu.com`、`mobads.baidu.com` | 百度 mobads |
| `nsclick.baidu.com` | 点击追踪 |

### 其他 SDK

| 域名 | 厂商 |
|---|---|
| `sdk.beizi.biz` | 倍孜 |
| `ad-api.adn-plus.com.cn` | 阿里妈妈 |
| `sdk-track.66mobi.com` | 快手 |
| `sdk.aqyad.com` / `trace.aqyad.com` | 爱奇艺 |
| `ad.shunchangzhixing.com` | 顺昌知行 |
| `ad.baihemob.com` | 白禾 |
| `sdk.adx.adwangmai.com` | 旺脉 |
| `sdk.1rtb.net` / `s.1rtb.net` | 1rtb |
| `adimage.bwton.com` | 博通 |
| `sigmob.cn` / `sigmob.com` | Sigmob |
| `api.touch-moblie.com` | 触漫 |
| `g.cn.miaozhen.com` | 秒针 |
| `yunxish.com` | 云曦聚合 |

### 自有广告平台

| 域名 | App |
|---|---|
| `ap.dongqiudi.com` / `dab.dongqiudi.com` | 懂球帝（决策 / 埋点） |
| `goblin.hupu.com` / `e-goblin.hupu.com` | 虎扑（拉取 / 曝光点击） |
| `cm.bilibili.com` / `gaoneng.bilibili.com` | B站（转化上报 / 落地页） |

## 编译

```bash
sing-box rule-set compile --output ads.srs ads.json
```

## 已知陷阱

整理过程中踩过的坑，自建规则集时同样适用：

1. **`rule-set match` 必须加 `-f binary`** 才能读 `.srs`，否则报 `invalid character 'S'`
2. **`rule-set match` 命中信息走 stderr**，退出码恒为 0，不能只看 stdout 或 `$?`
3. **`domain_suffix` 是后缀匹配**：`gdt.qq.com` 自动覆盖 `win.gdt.qq.com`
4. **`domain_suffix` 别写前导点**：`.a.com` 不匹配 `a.com` 本身
5. **`domain_regex` 不加锚点**：`ad` 会匹配一切含 ad 的域名
6. **空规则编译能过但启动 FATAL**：`{"type":"logical","rules":[]}` 报 `missing conditions`
7. **字段生效范围取决于引用方**：规则集被路由规则引用时不认 `query_type`（FATAL），
   被 DNS 规则引用时 `port` 条件静默失效
8. **白名单是后缀匹配**：写 `qq.com` 会连带跳过 `gdt.qq.com` 等广告子域

## 免责

规则集基于公开的广告 SDK 域名特征整理，仅用于个人网络环境减少广告流量。
不同 App 版本、不同时间的广告域名会变化，规则可能失效或误伤，请自行验证。

## License

MIT
