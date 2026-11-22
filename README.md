# NezhaBeautify v2

把 [KomariBeautify](https://github.com/EmersonLopez2005/KomariBeautify)（Komari v1.1.3 + PurCarte v1.2.5）的 5 个功能，重做到 **Nezha v1** 上。
原版是单文件自定义代码，本项目同样保持**单文件**（`NezhaBeautify.html`），直接粘贴进 Nezha 后台的自定义代码即可用。

> **适配目标：nezhahq/nezha v1.x，实测对照版本 v1.14.14（v1 线最新版）。**
> 注意：nezhahq/nezha 的 main 分支现在是 **v2.x**（v2.7.0），与 v1 的 API/前端结构不同。
> 本项目只针对 v1.x；如果你用的是 v2，请勿直接使用（见“已知限制”）。
>
> **有 v2 版了：** 见下文 [NezhaBeautify v2](#nezhaBeautify-v2) 一节。
> `NezhaBeautify-v2.html` / `NezhaBeautify-v2.js` 适配 Nezha **v2.6.2**，
> 是在本站（status.alicloud.ru，Nezha 2.6.2）实测诊断后做的移植版。

## 功能

| # | 功能 | 说明 |
|---|------|------|
| 1 | 欢迎气泡 | 左下角弹出：位置（城市/地区/国家）、OS、浏览器、IP（点击剧透隐藏）、ISP、日期。IP 数据三源故障转移：api.ip.sb → ipwho.is → api.ipapi.is。站名从 `/api/v1/setting` 的 `site_name` 取。 |
| 2 | 资产统计 | 右上角悬浮球，点击展开：服务器总数、总价值、月均支出、剩余总价值、逐台明细。货币 CNY/USD/HKD/EUR/GBP/JPY，汇率走 exchangerate-api.com（失败用默认汇率）。排序默认按 Nezha 的 `display_index`，也可按价格排序；可排除“白嫖中”。 |
| 3 | 地球仪 | 右上角第二个悬浮球，点击打开全屏 3D 地球（Globe.GL）：按国家代码标国旗、画到访客位置的弧线、点击国旗看该国服务器列表；浅/暗色自动切换日间/夜间纹理。 |
| 4 | 保护 | 仅**未登录游客**触发：拦截右键/复制/拖拽/打印/F12/Ctrl+Shift+I 等、检测无头浏览器/开发者工具/可疑扩展/DOM 注入，触发后弹警告，3 秒后跳转（原版行为）。开关见 `CONFIG.protection`。 |
| 5 | 自定义警告弹窗 | 保护组件的弹窗，模糊背景 + 动画。 |

## 安装（Nezha v1）

### 方式一：script 引用（推荐，一行搞定）

在 Nezha v1 后台（`/dashboard/`）→ **设置** → **自定义代码**
（`custom_code`）里只粘贴这一行，保存即可：

```html
<script src="https://cdn.jsdelivr.net/gh/gugetang/NezhaBeautify-v1@main/NezhaBeautify.js"></script>
```

`NezhaBeautify.js` 是由 `NezhaBeautify.html` 用 `build-js.py` 生成的
同款单文件 JS：它自己注入样式和面板节点，后台加载 Globe.GL，
再执行同样的逻辑，和直接粘贴 HTML 效果一样。改配置时，在这行前面再加一行：

```html
<script>window.NezhaBeautifyConfig = { finance: { prices: {} } };</script>
<script src="https://cdn.jsdelivr.net/gh/gugetang/NezhaBeautify-v1@main/NezhaBeautify.js"></script>
```

注意：jsdelivr 对 `@main` 分支有缓存（最长约 12 小时），刚推上去的新版
可能不会立刻生效；等缓存过期，或改用固定提交号引用，或用 purge 链接清缓存：
`https://purge.jsdelivr.net/gh/gugetang/NezhaBeautify-v1@main/NezhaBeautify.js`。

### 方式二：直接粘贴 HTML

1. 登录 Nezha v1 后台（`/dashboard/`）→ **设置** → 找到**自定义代码**（`custom_code`，用户前台用）。
2. 把 `NezhaBeautify.html` 的**全部内容**复制粘贴进去，保存。
3. 刷新用户前台首页即可看到欢迎气泡和右上角两个悬浮球。
4. （可选）按下面“配置”填价格表，否则资产统计会显示“未配置”。

> 注入机制：官方用户前端（nezha-dash-v2）在加载 `/api/v1/setting` 后，用
> `InjectContext(custom_code)` 按文档顺序注入这段 HTML：
> `<style>`/`<link>` 进 `<head>`，`<script>`（含外链 Globe.GL）按序执行，
> 其它节点进 `<body>`。所以本文件必须是“样式 + 节点 + 脚本”一体的单文件。

## 配置（只改文件顶部 `CONFIG`）

```js
var NezhaBeautifyConfig = {
  settingApi: '/api/v1/setting',
  wsServerUrl: '/api/v1/ws/server',
  serverApi: '/api/v1/server',      // 仅登录后可用，WS 失败时的备用
  profileApi: '/api/v1/profile',    // 登录状态检查
  siteNameFallback: 'Nezha',
  protection: { enabled: true, guestsOnly: true, redirectDelayMs: 3000, pollMs: 500 },
  globe: {
    cdnDay / cdnNight / cdnSky / cdnTopology: '…',  // 地球纹理 CDN
    localDay / localNight / localSky: '',           // 填自己的图床 URL 可覆盖 CDN
    flagUrl: code => 'https://flagcdn.com/h60/' + code.toLowerCase() + '.png',
  },
  finance: {
    defaultCurrency: 'CNY',
    excludeFreeDefault: true,   // 默认排除“白嫖中”
    freeTag: '白嫖中',
    longTermYears: 100,
    prices: { /* 见下 */ },
  },
};
```

也可以在页面里先定义 `window.NezhaBeautifyConfig` 再加载本文件，会与默认配置合并。

### 资产统计的价格从哪来（必读）

**Nezha v1 的 Server 模型没有 `price / billing_cycle / expired_at / tags / weight / currency` 字段**，
所以价格必须另行提供，二选一（优先级：价格表 > 公开备注）：

**方式 A：在 `CONFIG.finance.prices` 里按服务器名填：**

```js
prices: {
  'HK-小鸡': { price: 100, currency: 'CNY', cycleDays: 365, expireAt: '2027-01-01' },
  'JP-免费': { price: -1 },   // price = -1 表示免费鸡，按 0 计
}
```

字段：`price`（必填，数字；`-1` = 免费）、`currency`（CNY/USD/HKD/EUR/GBP/JPY 或 ￥/$/HK$/€/£，默认 CNY）、
`cycleDays`（计费周期天数，30=月付、365=年付，默认 30）、`expireAt`（到期日 `YYYY-MM-DD`，不填则剩余价值按 0 计）、
`free: true`（等同于“白嫖中”，受排除开关控制）。

**方式 B：在 Nezha 后台给服务器写公开备注（`public_note`），格式：**

```
price:100CNY/365d;expire:2027-01-01
price:5USD/30d;expire:2026-06-01;白嫖中
```

注意：公开备注是**公开可见**的，写价格等于把价格公开给所有访客；不想公开就用方式 A。
另外 WS 首包才带 `public_note`，本代码会缓存首包备注，后续包为空时不覆盖。

**剩余价值算法（与原版一致）：** 到期 > 100 年视为长期，按原价计；否则按
`价格 ×（剩余天数 / 计费周期天数）`；无到期日则剩余价值为 0。
未配置价格的服务器在明细里显示“未配置”，不计入总额，数量会标注在“服务器数量”后。

## Komari → Nezha v1 映射表

| 原版（Komari + PurCarte） | Nezha v1 改编 | 备注 |
|---|---|---|
| `GET /api/nodes`（公开）取服务器 | `WS /api/v1/ws/server`（公开，除非开 ForceAuth）+ `GET /api/v1/server` 备用（仅登录） | v1 的服务器列表 REST 接口不对游客开放；公开数据走 WS，每 2 秒一包 |
| 节点字段 `name / price / currency / billing_cycle / expired_at / tags / weight` | `StreamServer`：`{id, name, public_note（仅首包）, display_index, host, state, country_code, last_active}` | **无价格/周期/到期/标签/权重字段**，见上节 |
| 排序 `weight` 正序/倒序 | 排序 `display_index`（越大越靠前，Nezha 原生）+ 价格正序/倒序 | |
| 标签 `白嫖中`（tags） | 价格表 `free: true` 或公开备注含“白嫖中” | Nezha 无 tags |
| `GET /api/me` → `logged_in` 判断登录 | `GET /api/v1/profile`：200 + `success:true` = 已登录 | Komari 的 `/api/me` 在 Nezha 不存在 |
| 自定义代码放 Komari 设置 → 自定义头部/Body | Nezha 后台 → 设置 → 自定义代码（`custom_code`） | 见“安装” |
| `MutationObserver` 监视 `.radix-themes`，节点挂到该容器下 | 监视 `#root`，节点直接挂到 `document.body` | Nezha 前端无 `.radix-themes`；根节点是 `<div id="root">`（React 18） |
| 样式变量 `--accent-9 / --purcarte-card-color / --gray-a5` 等 | 映射到 Nezha 的 Tailwind/shadcn 变量 `--background / --card / --foreground / --primary / --muted-foreground / --border / --destructive`（HSL 三元组），并自带回退值 | 详见下节 |
| 主题检测 `localStorage.appearance` | `html.dark` class + `localStorage "vite-ui-theme"`（light/dark/system）+ `prefers-color-scheme` 回退 | |
| 地球国旗 `/assets/flags/{CODE}.svg`（Komari 自带） | `flagcdn.com/h60/{code}.png`（3:2，`CONFIG.globe.flagUrl` 可改），加载失败自动隐藏 | Nezha 前端不提供该路径 |
| 地球纹理 unpkg three-globe CDN | 同上 CDN 默认；本项目 `images/` 有本地副本，可自托管覆盖 | 见 `images/SOURCE.unpkg.txt` |

## Nezha v1 API 核实记录（来源：nezhahq/nezha v1.14.14 源码）

- 路由表：`cmd/dashboard/controller/controller.go`（`routers()`），`@BasePath /api/v1`。
- `GET /api/v1/setting`：`fallbackAuth` 组，可公开访问；返回
  `CommonResponse{success, data:{config:{site_name, custom_code, custom_code_dashboard, ...}}}`。
  字段定义：`model/config.go:27-28`（`custom_code` / `custom_code_dashboard`），
  `model/setting_api.go:11-12`。
- `GET /api/v1/ws/server`：`optionalAuth` 组（`ForceAuth=true` 时才强制登录）；
  `cmd/dashboard/controller/ws.go:106-196`（`serverStream` / `getServerStat`），
  每 2 秒推送一次，首包带 `public_note`。
  推送体：`model/server_api.go:5-21`（`StreamServer` / `StreamServerData{now, online, servers}`）。
  游客：`hide_for_guest` 的服务器被排除（`GetSortedListForGuest`），
  `host` 只保留 Platform/CPU/内存/磁盘等字段（`Host.Filter()`，`model/host.go:131`）。
- `GET /api/v1/server`：`auth` 组（`controller.go`），**必须登录**；返回
  `CommonResponse[[]Server]`。`Server` 模型见 `model/server.go`：
  `name / uuid / note（仅管理员） / public_note / display_index / hide_for_guest /
  host / state / geoip{ip, country_code} / last_active`——**无价格类字段**。
- `GET /api/v1/profile`：`auth` 组；`cmd/dashboard/controller/user.go:24`
  （`getProfile`），未登录返回 `unauthorized` 错误。登录用 JWT，cookie 名 `nz-jwt`
 （`cmd/dashboard/controller/jwt.go:23,34`）。
- 通用返回体：`model/api.go:21`（`CommonResponse{success, data, error}`）。
- 前端注入：官方用户前端 `hamster1963/nezha-dash-v2`（`frontend-templates.yaml` 中
  Official v2.7.0）：`src/App.tsx` 取 `settingData.data.config.custom_code`，
  经 `src/features/runtime-config/inject.ts`（`InjectContext`）注入；
  根节点 `<div id="root">`（`index.html` / `src/main.tsx`，`WebSocketProvider url="/api/v1/ws/server"`）；
  主题变量见 `src/index.css`（`:root` / `.dark`，HSL 三元组；主题键 `vite-ui-theme`）；
  登录检查用 `GET /api/v1/profile`（`src/lib/nezha-api.ts:fetchLoginUser`）。
- 注意：Nezha v1 官方前端自带一个 2D 世界地图（`src/components/GlobalMap.tsx`），
  本项目的 3D 地球仪是额外功能，与它不冲突。

## 已知限制

1. **资产统计依赖手动填价**：Nezha v1 没有价格/到期字段，不填 `prices` 或公开备注就只能看到服务器数量，金额全是“未配置”。这是 Nezha v1 数据模型本身的限制，不是 bug。
2. **公开备注的价格会被所有访客看到**（包括 WS 首包）。在意隐私请用方式 A（`CONFIG` 价格表）。
3. **WS 首包才有 `public_note`**：页面打开后如果错过首包（例如 WS 重连），备注解析可能暂时缺失；本代码会缓存已拿到的备注，重连后一般能恢复。
4. **ForceAuth 的站**：后台开启“强制要求认证”后，游客连 WS 都拿不到服务器列表，欢迎气泡以外的地球/资产功能对游客不可用（登录后正常）。
5. **保护组件是前端威慑，不是安全边界**：和原版一样，懂行的人仍可绕过；不要把它当成真正的访问控制。`CONFIG.protection.enabled=false` 可彻底关闭。
6. **仅适配 Nezha v1.x**：v2（main 分支，v2.7.0）的 API 与前端结构已变，本文件未经 v2 测试，很可能无法工作。
7. **Globe.GL 走 CDN**（`//unpkg.com/globe.gl`）：离线 / CDN 被墙的环境地球仪打不开，其余功能不受影响。

## 待验证 / Unknowns

以下是在源码里**未能完全核实**、需要你在真实 Nezha v1 实例上测试确认的：

- [ ] `custom_code` 在你所用的 v1.x 小版本里的注入时机：v1.14.14 的官方前端是在设置加载后注入；如果你用的是第三方主题（如 Nezha-Pixel / Nazhua 等），注入方式和根节点可能不同，需按该主题调整。
- [ ] `GET /api/v1/profile` 在**未登录**时的确切 HTTP 状态码（源码里是 gin-jwt 的 `unauthorized`，本代码按 `!res.ok || !json.success` 判断，已覆盖两种情况，但未在真站上实测）。
- [ ] WS 在反代 / 子路径部署下的表现：`wsServerUrl` 默认相对路径按当前 host 拼接；如果你的 Nezha 在子路径或 WS 被反代拦截，需要改成绝对 `wss://…` 地址。
- [ ] `public_note` 价格标记解析：格式是我方约定的（`price:…;expire:…`），Nezha 本身没有这个约定；只有你按本文档写了备注才会生效。
- [ ] 地球纹理 CDN（unpkg）在你访客网络下的可达性；如需自托管，把 `images/` 里的三张图放到你自己的图床，并在 `CONFIG.globe.localDay/localNight/localSky` 里填 URL。
- [ ] `flagcdn.com` 国旗 CDN 的可达性；不可达时国旗标记会自动隐藏（不报错），弧线和计数器不受影响。
- [ ] 汇率 API（exchangerate-api.com）的可达性；失败时自动用 `CONFIG.finance.defaultRates` 默认汇率，并在小部件底部注明。

## NezhaBeautify v2

<a id="nezhaBeautify-v2"></a>

`NezhaBeautify-v2.html`（单文件自定义代码）与 `NezhaBeautify-v2.js`
（script 引用版，由 `build-js-v2.py` 从 v2 HTML 生成）适配
**Nezha v2.6.2**，官方用户前端 **nezha-dash-v2 2.7.0 @ 1dd038b**。
v1 版文件（`NezhaBeautify.html` / `NezhaBeautify.js`）不动，仍只适配 v1.14.x。
v2 文件由 `make-v2.py` 从 v1 文件转换生成，5 个功能全部保留。

### 安装（Nezha v2）

#### 方式一：script 引用（推荐，一行搞定）

> ⚠️ **还没推上 GitHub**：`NezhaBeautify-v2.js` 目前只在本地，
> 下面的链接是占位写法，推上去之后才能用。推的时候把 `NezhaBeautify-v2.html`、
> `NezhaBeautify-v2.js`、`build-js-v2.py`、`make-v2.py` 和更新后的 `README.md`
> 一起推到 `gugetang/NezhaBeautify-v1` 的 `main` 分支，链接就生效了
> （jsdelivr 对 `@main` 有最长约 12 小时缓存，刚推完可能不立刻生效，
> 可用固定提交号引用或 purge 链接清缓存，见 v1 安装节）。

在 Nezha v2 后台（`/dashboard/`）→ **设置** → **自定义代码**
（`custom_code`）里粘贴这一行（加在现有内容后面，不要删掉已有的行），保存：

```html
<script src="https://cdn.jsdelivr.net/gh/gugetang/NezhaBeautify-v1@main/NezhaBeautify-v2.js"></script>
```

改配置时，在这行前面再加一行：

```html
<script>window.NezhaBeautifyConfig = { finance: { prices: {} } };</script>
<script src="https://cdn.jsdelivr.net/gh/gugetang/NezhaBeautify-v1@main/NezhaBeautify-v2.js"></script>
```

#### 方式二：直接粘贴 HTML

把 `NezhaBeautify-v2.html` 的**全部内容**粘贴进同一位置（`custom_code`），保存。
与 v1 版一样，价格表填在文件顶部 `CONFIG.finance.prices`（见上文“资产统计的价格从哪来”）。

### v1 → v2 映射表

| v1 版 | v2 版 | 依据 / 备注 |
|---|---|---|
| 适配 nezhahq/nezha v1.14.14 | 适配 nezhahq/nezha **v2.6.2**，前端 nezha-dash-v2 **2.7.0 @ 1dd038b** | 后台/页脚显示 2.6.2；`service/singleton/frontend-templates.yaml` 与 dash `package.json` |
| `GET /api/v1/profile`：游客 401 | 游客返回 **HTTP 200 + `{"error":"ApiErrorUnauthorized"}`**，必须看 `json.success` | v2.6.2 `cmd/dashboard/controller/user.go:24-28`；2026-11-20 在 status.alicloud.ru 上实测 |
| `GET /api/v1/server`：游客 401 | 游客同样返回 HTTP 200 + 错误体；代码按 `json.success` 判断 | v2.6.2 `cmd/dashboard/controller/controller.go:151`（auth 组） |
| `GET /api/v1/setting` 公开 | 相同，但**不含 version 字段**，版本号只能看后台/页脚 | v2.6.2 `controller.go:97`（fallbackAuth）；`model/config.go:40-41` |
| `WS /api/v1/ws/server` 每 2 秒 | 相同（`optionalAuth`，ForceAuth 时才强制登录） | v2.6.2 `controller.go:109`；`cmd/dashboard/controller/ws.go:109`（serverStream，2 秒间隔 ws.go:154,170）；`model/server_api.go:5-21` |
| Server 模型无价格字段 | 相同（`model/server.go:21-40`，无 price/billing/expire/tags/weight/currency） | 资产统计仍用 `CONFIG.finance.prices` 价格表 + `public_note` 解析 |
| 主题变量 HSL 三元组 + `vite-ui-theme` | 相同（Tailwind v4，`src/index.css` `:root` / `.dark`，`@tailwindcss/postcss ^4.3.2`） | 变量名与 v1 相同，v2 文件的 `--nb-*` 映射不用改 |
| 外链 `<script src="//unpkg.com/globe.gl">` | **删掉**，改由 JS 后台动态加载 Globe.GL（unpkg 失败自动换 jsdelivr `globe.gl@2.46.2`，两个都实测 200） | 见下文诊断 (d)：InjectContext 会逐个等待外链加载，失败会清掉整段自定义代码 |
| 初始化：等 `#root` 就绪后 attach 一次 | **守护者挂载**：节点挂 `document.body`，拍 outerHTML 快照，body 的 MutationObserver + 3 秒轮询持续守护，被移走挪回、被删掉按快照重建并重跑对应功能初始化 | 见下文诊断 (b)；v2 前端里没有旧代码等的那类主题容器 |
| 各功能初始化顶层依次执行，一个抛错全灭 | 各功能各自 try/catch，互不拖累 | InjectContext 会捕获内联脚本执行期的 window error 并清掉整段代码 |
| 保护：`GET /api/v1/profile` 判断登录 | 相同接口，但按 `json.success` 判断（见上） | 旧站内联代码用已不存在的接口判断登录，见下文诊断 (e) |
| 新增 `GET /api/v1/service` 说明 | 本项目不调用它（延迟是 NezhaLatency 项目的事），只在 CONFIG 注释里列出 | v2.6.2 `controller.go:112`；`model/service_api.go:24-33` |

### v2 API 核实记录（来源：nezhahq/nezha v2.6.2 与 hamster1963/nezha-dash-v2 2.7.0 源码）

- 路由表：`cmd/dashboard/controller/controller.go`（`@BasePath /api/v1`）。
  `GET /api/v1/setting`（fallbackAuth，`:97`）、`GET /api/v1/ws/server`
  （optionalAuth，`:109`）、`GET /api/v1/service`（`:112`）、
  `GET /api/v1/service/:id/history`（`:114`）、`GET /api/v1/server/:id/service`
  （`:115`）、`GET /api/v1/profile`（auth，`:127`）、`GET /api/v1/server`
  （auth，`:151`）。
- `GET /api/v1/service/:id/history`：`cmd/dashboard/controller/service.go:165-175`，
  `?period` 只接受 `1d/7d/30d`（默认 `1d`），访客只能查 `1d`
  （`tsdb.ParseQueryPeriod`，见 `service.go:166`）。
- 前端注入：`src/features/runtime-config/inject.ts`（`InjectContext`）——
  按文档顺序注入，`<style>` 进 head，外链 `<script>` 逐个 `await load`
  （30 秒超时，失败 `dispose()` 清掉全部已注入节点并抛错），内联
  `<script>` 进 body（执行期 window error 会被捕获并触发 dispose）；
  每次调用开头先 `disposeCurrent()` 清掉上一次注入的节点。
  `src/App.tsx:98-127`：设置加载后调用 `InjectContext(custom_code)`，
  effect 依赖是 `[customCode, settingsPending]`；设置查询
  `refetchOnMount: true, refetchOnWindowFocus: true`（`App.tsx:70-76`）。
  所以 custom_code 字符串一变就重新注入、旧节点被清掉；路由切换不卸载
  App，不会动 body 下的节点。根节点 `<div id="root">`（`index.html:179`，
  `src/main.tsx`，`WebSocketProvider url="/api/v1/ws/server"`）。
- 主题：`src/index.css`（`:root` 约 `:119`、`.dark` 约 `:153`，HSL 三元组；
  主题键 `vite-ui-theme`）；`src/main.tsx` 引入 `flag-icons`（前端自带国旗，
  自定义代码仍用 flagcdn，见 v1 说明）。

### 诊断：status.alicloud.ru 上自定义代码“时有时无”（2026-11-20）

**首要原因（已证实）：测试期间 custom_code 本身被人改了。**
2026-11-20 23:12 匿名 curl `/api/v1/setting`，custom_code 是 **106399 字符**，
含 NezhaLatency.js 引用和整套欢迎气泡/资产统计/地球内联代码；23:24 两次
匿名 curl 都是 **1789 字符**，只剩 `window.CustomBackgroundImage` /
`CustomMobileBackgroundImage` / `CustomLogo` / `CustomDesc` /
`DisableAnimatedMan` / `FixedTopServerName` / `ForceUseSvgFlag` / `CustomLinks`
这几行，加上 `TrafficScriptConfig` 和 traffic-progress.js 的 script 标签，
NezhaLatency.js 和内联代码全没了（本次任务中再次 curl 复核确认仍是 1789 字符）。
浏览器登录后两次重载（23:16–23:21）看到整套自定义层都没渲染，与这次改动
的时间窗口吻合——代码被从设置里删了，前台当然什么都不显示。这很可能就是
站长本人在测试期间改的设置。**结论：先以当前 1789 字符的 custom_code 为准，
不要再按 106KB 那版来推断问题；v2 版测试前务必先确认后台 custom_code 里
到底有什么，再决定加哪一行。**

其余假说逐一验证如下（旧 106KB 版内联代码取自 23:12 的 `/api/v1/setting`
快照，`/tmp/status-setting.json`）：

| # | 假说 | 验证结果 |
|---|---|---|
| (0) | 测试期间 custom_code 被改动 | ✅ **证实，首要原因**。106399 → 1789 字符（23:12→23:24，匿名 curl 两次复核）。新版里欢迎气泡/资产/地球/延迟本来就不存在，不是渲染失败 |
| (a) | 内联代码依赖的旧接口在 v2 上 404，导致初始化中断 | ✅ **证实（对旧 106KB 版）**。`/api/nodes`、`/api/me` 在 v2.6.2 路由表里根本不存在（源码 grep 为空），实测都返回 `{"error":"404 Not Found"}`。旧代码资产统计 `fetch('/api/nodes')` 失败抛错 → `calculate` 从不执行 → 金额一直是 `...`，与实测完全吻合；地球 `fetch('/api/nodes')` 拿到 404 体，`json.status !== 'success'` → 节点列表为空 → 计数 `TOTAL SERVERS 0`，与实测完全吻合。**但这只能解释旧版“有球无数据”，解释不了登录后“整个自定义层消失”**（那是假说 0） |
| (b) | v2 前端重渲染清掉挂载节点，旧代码只盯 `#root` 或不存在的容器 | ⚠️ **部分证实，是隐患不是主因**。旧代码的 MutationObserver 等的是前端里不存在的主题容器（nezha-dash-v2 源码 grep 为 0），永远等不到、也永远不 disconnect；但 InjectContext 是把 HTML 节点直接挂在 body 下的，React 不动 body 节点，路由切换也不会清掉它们，所以单凭这一项解释不了消失。真正的清除机制是 InjectContext 的 dispose：custom_code 一变就清掉上一次注入的全部节点（见上文注入机制）。v2 版改用守护者挂载（快照 + body 观察 + 3 秒轮询）对冲这一点 |
| (c) | NezhaLatency.js 与内联代码 / traffic-progress.js 互相干扰或重复注入 | ❌ **无证据，已排除为主要原因**。三者用的 id、全局变量、接口互不重叠：NezhaLatency 用 `nl-*` id 和 `/api/v1/service` + WS；traffic-progress.js（17KB，实测下载）只 fetch `/api/v1/service`、建自己的 div/style、无 WebSocket；内联代码用自己的 id。NezhaLatency 的 script 版有 `nl-fab` 去重保护；重复注入时 InjectContext 的 dispose 只清它自己跟踪的节点，脚本副作用建的节点不受它管。延迟按钮“点了不弹出面板”在旧版上无法复现定位：`openModal` 会立即加 `show` class 再 `loadAll`，失败也会显示提示而不是静默无反应；可能与当时 custom_code 正在被改动有关，列入待验证 |
| (d) | jsdelivr `@main` 缓存 / Globe.GL / ECharts CDN 加载失败 | ❌ **已排除**。2026-11-20 实测：`NezhaLatency.js` jsdelivr 200（47908 字节）、`NezhaBeautify.js`（v1）200（82760 字节）、`globe.gl` unpkg 跟随 302 后 200（1.88MB，`globe.gl@2.46.2`）、jsdelivr 的 `globe.gl@2.46.2` 200、ECharts jsdelivr 200。NezhaLatency 的 ECharts 加载器本身也有 jsdelivr→unpkg 回退 + 表格降级。**但机制上有真风险**：InjectContext 会逐个等待外链脚本，30 秒超时或失败就清掉全部节点，所以 v2 版删掉了外链 Globe.GL 标签，改由 JS 后台动态加载（unpkg→jsdelivr 回退） |
| (e) | 登录态下保护逻辑 fetch 失败走 catch 分支的影响 | ✅ **证实（旧版逻辑缺陷）**。旧代码末尾 `fetch('/api/me')`，v2 上 404 → `!response.ok` 抛错 → `.catch` 里直接 `initializeProtection()`。也就是说旧版在 v2 上**无论登录与否都会启动保护**（右键/复制拦截对管理员也生效）。v2 版改用 `/api/v1/profile` + `json.success` 判断，只在访客时启用 |

### 资产统计（v2）：读后端账单数据（2026-11-20）

**结论：后端确实有价格数据，就藏在服务器的 `public_note` 字段里。**
Nezha v2 Server 模型本身没有价格字段（`model/server.go:27` 只有
`PublicNote string json:"public_note,omitempty"`），但后台“编辑服务器 →
公开备注 → 自定义字段 → 账单信息”表单保存后，会把账单信息写成 JSON
存进 `public_note`，WS 首包（`/api/v1/ws/server`）原样下发，v2 版直接
解析它，不用手填价格表。

**官方格式（已实测，见下）**：

```json
{"billingDataMod":{"startDate":"2026-11-15T16:00:00.000Z",
 "endDate":"2026-12-15T16:00:00.000Z","autoRenewal":"1",
 "cycle":"Month","amount":"$3.99"},
 "planDataMod":{"bandwidth":"1Gbps","trafficVol":"1TB/Month",
 "trafficType":"2","IPv4":"1","IPv6":"0","networkRoute":"CN2GIA,CN2GIA,CMIN2",
 "extra":"Premium"}}
```

字段含义（后台表单原样，截图已核对）：`startDate` 开始时间、
`endDate` 结束时间、`autoRenewal` 自动续费（`"1"`=启用，`"0"`=禁用）、
`cycle` 周期（`Day`/`Week`/`Month`/`Year`，表单显示 日/周/月/年）、
`amount` 金额（`"$3.99"`；`"0"`=免费；`"-1"`=按量计费）；
`endDate` 为 `"0000-00-00T23:59:59+08:00"` 表示不过期。套餐配置
（`planDataMod`：带宽/流量/IPv4/IPv6/线路/备注）只做展示参考，
不参与金额计算。

**证据**：(1) 站上前台 bundle `assets/index.CxhV59WK.js`（1,132,210 字节）
里 `billingInfo.tsx` 的计费组件：`价格：{amount}/{cycle}`、`"0"` 显示绿色
“免费”、`"-1"` 显示粉色“按量计费”、`0000-00-00` 开头显示“无限期”，
否则 `剩余：{days} 天` + 剩余百分比条；(2) `assets/utils.DKRb2Zh4.js`
里的 `parsePublicNote` / `getDaysBetweenDatesWithAutoRenewal`
（`nezha-dash-v2 src/lib/utils.ts:219+` 源码同款）：周期换算
月=1、年=12、季=3、半年=6，不续费按 `剩余天数/(结束-开始)` 算百分比，
续费且未到期按 `剩余天数/(30×月数)`，续费且已到期顺延到下一个周期；
(3) WS 首包实测（2026-11-20 23:50，`Origin: https://status.alicloud.ru`）：
22 台服务器**全部**带 `billingDataMod`，周期 月×5、年×14、日×3，
自动续费 `"1"`×21（仅 1 台缺失），金额含 `$3.99`、`$20.21`、`$5.00`、
`0`×5（免费，不过期）、`$0.6`×3、`￥10`、`￥14.99`、`$9.90`、`$10.95`、
`$6.75`、`$23.98/4`、`€3.79`、`$3.59`、`$0.03`；(4) 后台 bundle
`/dashboard/assets/index-ZxH4_bVK.js` 里的 zod schema：`billingDataMod` /
`planDataMod` 各字段均为可选字符串，保存时 `JSON.stringify(...)` 写入
`public_note`，原始文本模式则直接存原串；(5) 用户 2026-11-20 23:57 发的
后台截图：RakSmart US - SiliconValley，开始 2026/11/16、结束 2026/12/16、
自动续费启用、周期 月、金额 $3.99，与 WS 首包里该服务器的
`billingDataMod`（`2026-11-15T16:00:00.000Z` / `2026-12-15T16:00:00.000Z` /
`"1"` / `"Month"` / `"$3.99"`，UTC 与 +08:00 时区换算一致）完全吻合。
**管理员工单**：在后台 → 服务器 → 编辑 → 公开备注 → 自定义字段 →
账单信息里填开始/结束时间、自动续费、周期、金额，保存即可被读到；
想让某台显示“免费”填金额 `0`，按量计费填 `-1`。

**实现（`NezhaBeautify-v2.html`，函数名）**：
`parseBillingNote()` 解析 JSON（只要 `billingDataMod`）；
`parseAmountString()` 解析金额（符号定币种，`$23.98/4` 取 23.98）；
`cycleToDays()` / `cycleMonthsOf()` / `cycleLabelOf()` 换算周期；
`officialRemaining()` 与官方 `getDaysBetweenDatesWithAutoRenewal` 完全一致
的剩余天数/百分比算法；`scrapeCardBilling()` 兜底读卡片 DOM。
**优先级**：`CONFIG.finance.prices`（最高，可覆盖）> 后端
`billingDataMod` > 前台卡片 DOM（读已渲染的 `价格：$3.99/Month` /
`剩余：25 天`，只在 WS 拿不到数据时用）> 旧版文本标记
`price:100CNY/365d;expire:2027-01-01`（兼容保留）> 无价格（“未配置”）。
明细行显示原始金额 + 剩余天数（`剩25天` / `无限期` / `已过期X天` /
`按量计费`）；剩余价值 = 单价 × 剩余天数 / 周期天数（不过期按一个周期计，
与 >100 年长期规则一致）；按量计费不计入总额。
**本地验证**：用 WS 首包 22 台真实样本跑解析函数，RakSmart 剩余 25 天
（与卡片“剩25天”一致）、非续费服务器 201 天/55%、全部 22 台都有后端
数据、0 台未配置；`node --check` 通过。

### 地球按截图调整（2026-11-20）

用户发了两张目标截图（Komari 参考站风格，TOTAL SERVERS 25）：夜间地球 +
星空背景、日本/英国/加拿大国旗、蓝色弧线、绿色圆形 You 标记；白天地球 +
多国国旗、蓝色弧线、同款绿色圆形 You 标记。对照改动（只改 v2 版）：

- **计数器**：`processData()` 原来只计有 `country_code` 的台数，站上实测
  显示 21 而不是 22；改成计全部服务器 `servers.length`（旗帜仍只给有坐标
  的国家画）。
- **国旗更大更清晰**：`calculateFlagSize()` 从 base 24 / min 16 / max 40
  改为 base 36 / min 24 / max 48（重叠时缩小下限 16→24），
  `.earth-flag-img` 宽度 24→36px，加 `border-radius: 6px` 和阴影。
- **You 标记**：原来是 `/favicon.ico` 图片；改成白色圆底 + 3px 绿色
  `#22c55e` 圆环 + 绿色呼吸光晕（`userPulse` 改绿），圆内放访客所在国家
  的国旗（flagcdn `w80`，国家代码来自 IP 查询的 `country_code`，
  ipwho.is / api.ip.sb 已实测返回该字段；加载失败只剩绿环，不编造图片）。
  拿不到国家代码时用内联绿色定位针 SVG（纯 CSS/SVG，不依赖外部图片）。
  注意：截图里的绿色中国地图是参考站访客本人的标记样式，本项目用访客
  实际所在国国旗概括，不凭空编一个中国地图图片 URL。
- **昼夜/弧线**：昼夜贴图按主题切换已存在（浅色白天贴图、深色夜间贴图 +
  星空背景），核对无误保留；弧线和光圈颜色固定为 `#3b82f6` 蓝色
  （原来跟随主题色，深色下可能偏白），虚线动画保留。

### 地球按 Komari 夜图再调（2026-11-21）

用户又发了 Komari 夜间地球参考图并指出：黑夜模式样式不如参考图、国旗
大小不一样。按图再改（只改 v2 版）：

- **国旗 3:2**：源图从 flagcdn `w40`（40×27，约 4:3）换成 `h60`
  （90×60，实测精确 3:2）；`.earth-flag-img` 固定宽 36px、高 24px，
  `object-fit: cover`，缩放时高度按宽度 ×2/3 同步（`updateFlagSize`）。
- **弧线**：夜间冰蓝 `#7dd3fc`、白天绿色 `#22c55e`；线宽
  `arcStroke` 0.8→0.3；参考图是细实线，虚线改为实线
 （`arcDashLength` 1、`arcDashGap` 0、不再动画）。
- **光圈**：夜间冰蓝 `#7dd3fc`、白天太阳色暖黄 `#fbbf24`
 （`globeRingColor()`）；国旗辉光跟弧线同色（`globeLineColor()`）。
- **夜间大气辉光**：`getBackgroundConfig()` 深色氛围色改成冰蓝
  `rgba(125, 211, 252, 0.35)`，与参考图的蓝色辉光一致；白天保持不变。
- **主题切换**：`updateGlobeTheme()` 现在同步更新弧线/光圈颜色
  （`arcColor` / `ringColor`），切换昼夜不用重开地球面板。

### 地球最高清 + 欢迎气泡磨砂玻璃（2026-11-21）

只改 v2 版：

- **地球清晰度拉到最高（贴图）**：白天贴图原来是
  `earth-day.jpg`，实测只有 1600×800（239K），是白天糊的主因；
  换成 `earth-blue-marble.jpg` **4096×2048**（unpkg 实测 200，
  1.4MB）。夜贴图 `earth-night.jpg` 本来就是 4096×2048，保留；
  `earth-topology.png` 2048×1024、`night-sky.png` 4096×2048 也都
  实测过。8K（8192×4096）找过：solarsystemscope 的 8k 图被验证码墙
  挡住、NASA EO 图超时，都不可靠，所以 **4096 是当前可靠 CDN 的
  最高级**。每张贴图都有 jsdelivr 回退（同文件、实测 200 且字节数与
  unpkg 一致，如白天回退是
  `cdn.jsdelivr.net/npm/three-globe@2.31.0/example/img/earth-blue-marble.jpg`；
  注意 `@2.46.2` 的 npm 包里没有这张图，曾返回 404，所以回退 pin 在
  2.31.0），首次点地球球时先用 `Image` 预加载探测，主链失败自动换
  回退地址（`resolveGlobeTextures()`，结果缓存到 `resolvedTex`）。
- **地球清晰度拉到最高（渲染）**：按 globe.gl 2.46.2 的真实 API
  （README + 源码核实）：链上显式设
  `.rendererConfig({ antialias: true, alpha: true })`；
  `.renderer().setPixelRatio(Math.min(window.devicePixelRatio, 2))`
  （默认是 1，高分屏不设会糊，上限 2 兼顾性能）；
  `globeMaterial().map / .bumpMap` 的 `anisotropy` 取
  `renderer.capabilities.getMaxAnisotropy()`（库里默认是 1，斜视
  地球边缘时贴图会糊），`minFilter`/`magFilter` 显式写成线性
  mipmap（数值常量 1008/1006，因 globe.gl 没把 THREE 暴露成全局
  变量）。在 `onGlobeReady`、初始化后延时 1.5 秒、换主题后延时
  0.8 秒都补调一次（`applyGlobeHiRes()`），因为贴图异步加载、换
  主题重设贴图后 anisotropy 会回到默认 1。球体分段无法再调：
  three-globe 里写死 `SphereGeometry(r, 75, 75)`，没有公开可调参数，
  75×75 已是库内最高。
- **欢迎气泡磨砂玻璃半透明**：只改 `.welcome-bubble` 本身（资产
  统计/警告弹窗不动）。之前通用规则给的底是 `var(--nb-card)` 不透明
  色，`backdrop-filter` 再强也透不出背景；现在单独改成半透明底 +
  `backdrop-filter: blur(18px) saturate(180%)`、半透明边框、柔和阴
  影：白天 `rgba(255,255,255,0.62)` 底 + `rgba(255,255,255,0.65)`
  边框；黑夜（`html.dark`）`rgba(15,23,42,0.55)` 底 +
  `rgba(255,255,255,0.14)` 边框；分隔线改淡
  `rgba(148,163,184,0.25)`；文字颜色仍用 `--nb-fg` / `--nb-accent`，
  两边都清晰可读。文案、位置、出现/消失逻辑都没动。

### 地球换 2k + 加载不再阻塞（2026-11-21）

只改 v2 版（上一版 430a630 的地球在站上 3 次重测都打不开：点地球球
后弹窗空白、只有 TOTAL SERVERS 22，等 105 秒以上也不出模型）：

- **贴图全部换成 2k 级（实测尺寸/状态）**：

  | 贴图 | 旧（430a630） | 新 | 实测 |
  |---|---|---|---|
  | 白天 | `earth-blue-marble.jpg` 4096×2048，1.4MB | three.js 官方 `earth_atmos_2048.jpg` **2048×1024**，512,606B | jsdelivr gh 与 raw 均 200，字节一致 |
  | 夜晚 | `earth-night.jpg` 4096×2048，715,000B | three.js 官方 `earth_lights_2048.png` **2048×1024**，734,910B（深蓝底 + 城市灯光，全图实测有亮点；注意文件名是 `.png`，`.jpg` 是 404） | jsdelivr gh 与 raw 均 200，字节一致 |
  | 星空 | `night-sky.png` 4096×2048，904,154B | 不变（例外，见下） | unpkg 200 |
  | bump | `earth-topology.png` 2048×1024 | 不变，本来就是 2k | unpkg 200 |

  总阻塞贴图从约 3MB 降到约 1.6MB。找过的备选：`earth-day.jpg`
  只有 1600×800（可作白天备选，但不如 2048 的 atmos 图）；
  `earth-dark.jpg` 也是 2048×1024，但实测接近纯黑（均值
  11,11,11，几乎无灯光），不能做夜图；unpkg/jsdelivr npm 的
  three 包不带 `textures` 目录，`earth_atmos_2048.jpg` 在那里
  都是 404，只能走 `cdn.jsdelivr.net/gh/mrdoob/three.js@r160` /
  `raw.githubusercontent.com`。
- **星空是唯一的例外**：three-globe 与 three.js 官方来源里只有
  `night-sky.png` 4096×2048 这一张星空图，没有可靠的 2k 星空图
  （web 搜索只找到未验证的 AI 纹理站，无直接图片 URL，不用）。
  所以星空仍用原图，但不再参与建地球前的探测，建好地球后由
  Globe.GL 异步加载，不阻塞模型出现。
- **为什么上一版加载不出来（实测证据）**：430a630 的点地球流程是
  先 `resolveGlobeTextures()` 等 4 张贴图全部 `Image` 预加载完
  才调 `initGlobe()`，而 `resolveTexture()` 没有超时——某张图
  连接卡住时 `onload`/`onerror` 都不触发，`pending` 永远到不了 0，
  建地球的回调永远不跑，弹窗就是空白；而且 `isResolvingTextures`
  一直为 `true`，再点地球球也会直接 `return`，等于永久打不开。
  站上重测 3 次空白弹窗与这个死锁完全吻合（之前 1600×800 白天图
  + 4096 夜图那版没有这个预加载门，地球能正常加载）。
- **加载逻辑改动**：点地球球先直接用 2k 主链地址调 `initGlobe()`
  建地球（Globe.GL 加载贴图本身是异步的，不挡模型出现），再后台
  跑 `resolveGlobeTextures()`（只探白天/夜晚/bump 三张，每张 3 秒
  超时 `TEX_PROBE_TIMEOUT_MS`，超时即用回退地址），探测完由新增
  的 `applyResolvedTextures()` 把已建地球的贴图地址换成探测结果
  （主链正常时就是主链地址）。旧的 `if (isResolvingTextures)
  return;` 阻塞门已删。
- **保留不动**：国旗 h60 3:2、`arcStroke` 0.3、夜冰蓝 `#7dd3fc` /
  日绿 `#22c55e` 弧线、太阳色 `#fbbf24` 光圈、夜间冰蓝大气辉光、
  TOTAL SERVERS 计数、You 标记、欢迎气泡磨砂玻璃、资产统计、守护者
  挂载、渲染高清参数（antialias、pixelRatio 上限 2、max
  anisotropy）。

### 仍需在真实站上验证的清单（v2 版）

- [ ] 后台 custom_code 现状确认：测试前先读 `/api/v1/setting`，确认当前是 1789 字符版（只有背景/Logo/外链 + traffic-progress.js），再加 v2 的 script 行。
- [ ] 延迟面板在旧版上“点了不弹出”的原因：v2 版不含延迟功能（那是 NezhaLatency 的事），但若复现请看浏览器控制台有无 `Custom code initialization failed` 横幅、以及 `/api/v1/service` 是否正常。
- [ ] WS 在反代 / 子路径下的表现；资产统计后端解析（见上节 v2 说明）与卡片 DOM 兜底。
- [ ] Globe.GL 纹理 CDN（unpkg three-globe）与 flagcdn 在访客网络下的可达性；地球仪点开后 `Globe.GL` 是否从 unpkg 或 jsdelivr 加载成功。
- [ ] 守护者挂载：改动 custom_code 后重新注入时，旧节点是否被干净替换、有无重复悬浮球（理论上 InjectContext dispose + v2 的 `welcome-bubble-container` 去重保护会兜底）。
- [ ] 登录态下保护逻辑：用管理员账号登录后右键应**不**触发警告；游客应触发（3 秒后跳转）。

## 文件清单

```
nezha-beautify-v1/
├── README.md              # 本文件
├── NezhaBeautify.html     # v1 单文件自定义代码（样式 + 节点 + 脚本），粘贴进 custom_code
├── NezhaBeautify.js       # v1 script 引用版（build-js.py 生成）
├── NezhaBeautify-v2.html  # v2 单文件自定义代码（适配 Nezha 2.6.2，见上文）
├── NezhaBeautify-v2.js    # v2 script 引用版（build-js-v2.py 生成，未推 GitHub）
├── build-js.py            # v1：从 NezhaBeautify.html 生成 NezhaBeautify.js
├── build-js-v2.py         # v2：从 NezhaBeautify-v2.html 生成 NezhaBeautify-v2.js
├── make-v2.py             # 从 v1 HTML 转换生成 NezhaBeautify-v2.html（v1 文件不动）
└── images/
    ├── earth-day.jpg      # 地球日间纹理（three-globe 示例图本地副本）
    ├── earth-night.jpg    # 地球夜间纹理（本地副本）
    ├── night-sky.png      # 星空背景（本地副本）
    └── SOURCE.unpkg.txt   # 纹理来源 URL
```

纹理来源：`https://unpkg.com/three-globe/example/img/earth-day.jpg`、
`earth-night.jpg`、`night-sky.png`（three-globe 示例资源）。
本项目未修改这些图片；如需商用请自行确认 three-globe 示例资源的许可。
