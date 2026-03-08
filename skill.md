---
name: hip3-tradefi-skill
description: HIP-3 TradeFi Asset Index - Query and discover all RWA/TradeFi perpetual assets on Hyperliquid including stocks, commodities, indices, forex, sector ETFs and pre-IPO assets across all HIP-3 DEXs
---

# HIP-3 智能资产索引

查询 Hyperliquid 上所有 HIP-3 (builder-deployed perps) 的 RWA/TradeFi 资产，包括股票、商品、指数、外汇、板块 ETF、加密货币等。

## API 数据源

所有数据来自 Hyperliquid 官方 API：`POST https://api.hyperliquid.xyz/info`

### 1. 列出所有 HIP-3 DEX

```json
{"type": "perpDexs"}
```

返回所有已部署的 perp DEX 列表（xyz, flx, vntl, km, cash, hyna 等）。

### 2. 获取某个 DEX 的资产列表和行情

```json
{"type": "metaAndAssetCtxs", "dex": "xyz"}
```

返回两个数组：
- `[0]` 元数据：`universe` 数组（资产名、精度、杠杆、保证金模式等）
- `[1]` 行情上下文：每个资产的 markPx、oraclePx、funding、openInterest、dayNtlVlm 等

### 3. 获取资产描述信息

```json
{"type": "perpAnnotation", "coin": "xyz:CL"}
```

**coin 参数必须带 DEX 前缀**，如 `xyz:CL`、`xyz:MU`、`vntl:SPACEX`。

返回示例：
```json
{
  "category": "commodities",
  "description": "CL tracks the value of 1 barrel of West Texas Intermediate (WTI) Light Sweet Crude Oil. WTI serves as a primary global benchmark for oil prices due to its high quality (low density, low sulfur)."
}
```

更多示例：
```
xyz:MU → "MU tracks the value of 1 share of common stock in Micron Technology, Inc. Micron manufactures DRAM and NAND flash memory..."
xyz:SILVER → "SILVER tracks the value of 1 troy ounce of silver..."
xyz:GOLD → "GOLD tracks the value of 1 troy ounce of gold..."
vntl:SPACEX → "SPACEX tracks the total valuation of SpaceX, in billions. E.g. if 1 SPACEX = $500, then the implied total valuation..."
```

> 注意：不是所有 DEX 的资产都有描述（如 `flx:OIL` 返回 `null`），主要 xyz 和 vntl 有完整描述。

### 4. 获取资产分类

```json
{"type": "perpCategories"}
```

返回 `[["km:AAPL", "stocks"], ["km:GOLD", "commodities"], ...]`

### 5. 获取所有中间价

```json
{"type": "allMids", "dex": "xyz"}
```

### 6. 获取 OI 上限

```json
{"type": "perpDexLimits", "dex": "xyz"}
```

## 查询思路

所有接口均为 `POST https://api.hyperliquid.xyz/info`，Content-Type: application/json。

### 用 curl 查询示例

```bash
# 1. 列出所有 HIP-3 DEX
curl -s -X POST https://api.hyperliquid.xyz/info \
  -H 'Content-Type: application/json' \
  -d '{"type":"perpDexs"}'

# 2. 获取某个 DEX 的全部资产 + 实时行情（markPx/funding/OI/volume）
curl -s -X POST https://api.hyperliquid.xyz/info \
  -H 'Content-Type: application/json' \
  -d '{"type":"metaAndAssetCtxs","dex":"xyz"}'

# 3. 获取单个资产的描述（coin 必须带 DEX 前缀）
curl -s -X POST https://api.hyperliquid.xyz/info \
  -H 'Content-Type: application/json' \
  -d '{"type":"perpAnnotation","coin":"xyz:CL"}'

# 4. 获取所有资产分类
curl -s -X POST https://api.hyperliquid.xyz/info \
  -H 'Content-Type: application/json' \
  -d '{"type":"perpCategories"}'

# 5. 获取某 DEX 所有资产中间价
curl -s -X POST https://api.hyperliquid.xyz/info \
  -H 'Content-Type: application/json' \
  -d '{"type":"allMids","dex":"xyz"}'
```

### 典型查询流程

1. **发现所有 DEX** → `perpDexs`，遍历返回结果中非 null 的项，取 `name` 字段
2. **拉某个 DEX 的资产列表** → `metaAndAssetCtxs`，`result[0].universe` 是资产元数据数组，`result[1]` 是对应的行情数组，两者按下标一一对应
3. **查某个资产的描述** → `perpAnnotation`，coin 格式为 `dex:TICKER`（如 `xyz:MU`）。cash 和 hyna DEX 的资产暂无描述（返回 null）
4. **批量获取价格** → `allMids`，返回 `{"xyz:MU": "363.10", "xyz:NVDA": "176.11", ...}`

## 已知 HIP-3 DEX

| DEX | 全名 | 定位 | 资产数 |
|-----|------|------|--------|
| xyz | XYZ (trade.xyz) | 最大，股票+商品+指数 | ~50 |
| flx | Felix Exchange | 商品+部分股票/加密 | ~12 |
| vntl | Ventuals | 板块 ETF、Pre-IPO (SpaceX/OpenAI/Anthropic) | ~13 |
| km | Kinetiq Markets | 股票+商品+指数+债券 | ~17 |
| cash | dreamcash | 精选股票+商品 | ~12 |
| hyna | HyENA | 加密货币永续合约 | ~21 |

## 全部资产索引（含官方描述）

> 描述来自 `perpAnnotation` API，cash 和 hyna DEX 暂无描述数据。

### xyz — 股票

| 代码 | 中文 | 描述 |
|------|------|------|
| xyz:AAPL | 苹果 | 追踪 Apple Inc. 1 股普通股价值。Apple 设计消费电子、软件和服务，包括 iPhone、Mac、iPad 和 Apple Watch。 |
| xyz:AMD | AMD | 追踪 AMD 1 股普通股价值。AMD 设计高性能处理器和显卡芯片，用于 PC、游戏主机、数据中心和 AI 工作负载。 |
| xyz:AMZN | 亚马逊 | 追踪 Amazon.com 1 股普通股价值。Amazon 运营全球电商市场和云计算平台 (AWS)，以及数字媒体和物流服务。 |
| xyz:BABA | 阿里巴巴 | 追踪阿里巴巴 1 份 ADR 价值。阿里巴巴运营电商市场、云服务和数字支付。 |
| xyz:COIN | Coinbase | 追踪 Coinbase Global 1 股普通股价值。Coinbase 运营加密货币交易所，提供交易、托管和数字资产基础设施。 |
| xyz:COST | 好市多 | 追踪 Costco 1 股普通股价值。Costco 运营会员制仓储零售店，提供批量商品和杂货。 |
| xyz:CRCL | Circle | 追踪 Circle Internet Group 1 股普通股价值。Circle 发行和管理 USDC 稳定币，提供数字资产支付基础设施。 |
| xyz:CRWV | CoreWeave | 追踪 CoreWeave 1 股普通股价值。CoreWeave 提供为 AI 和高性能计算优化的 GPU 云基础设施。 |
| xyz:GME | 游戏驿站 | 追踪 GameStop 1 股普通股价值。GameStop 是专注于视频游戏、消费电子和收藏品的零售商。 |
| xyz:GOOGL | 谷歌 | 追踪 Alphabet Inc. A 类普通股 1 股价值。Alphabet 是 Google 母公司，运营搜索、广告、云计算和 AI 业务。 |
| xyz:HOOD | Robinhood | 追踪 Robinhood Markets 1 股普通股价值。Robinhood 运营零售经纪平台，提供零佣金股票、期权和加密货币交易。 |
| xyz:HYUNDAI | 现代 | 追踪现代汽车 1 股普通股价值。Oracle 将韩元价格按 USD/KRW 汇率转换为美元。现代是全球汽车制造商，现代汽车集团持有 Boston Dynamics 80% 股权。 |
| xyz:INTC | 英特尔 | 追踪 Intel 1 股普通股价值。Intel 设计和制造半导体和计算平台，生产 CPU 和相关硬件。 |
| xyz:KIOXIA | 铠侠 | 追踪铠侠控股 1 股普通股价值。Oracle 将日元价格按 USD/JPY 汇率转换。铠侠是 NAND 闪存和固态存储解决方案的领先制造商。 |
| xyz:LLY | 礼来 | 追踪 Eli Lilly 1 股普通股价值。礼来开发和制造药品，包括糖尿病、肿瘤、免疫等治疗领域。 |
| xyz:META | Meta | 追踪 Meta Platforms 1 股普通股价值。Meta 运营 Facebook、Instagram 和 WhatsApp 等社交媒体和数字广告平台。 |
| xyz:MSFT | 微软 | 追踪 Microsoft 1 股普通股价值。Microsoft 开发软件、云计算服务和硬件，产品包括 Windows、Office、Azure 等。 |
| xyz:MSTR | MicroStrategy | 追踪 Strategy Inc (原 MicroStrategy) 1 股普通股价值。Strategy 是主要的企业比特币持有者。 |
| xyz:MU | 美光 | 追踪 Micron Technology 1 股普通股价值。Micron 制造 DRAM 和 NAND 闪存，用于数据中心、消费电子和工业系统。 |
| xyz:NFLX | 奈飞 | 追踪 Netflix 1 股普通股价值。Netflix 运营全球订阅制流媒体服务，提供原创和授权影视内容。 |
| xyz:NVDA | 英伟达 | 追踪 NVIDIA 1 股普通股价值。NVIDIA 设计 GPU 和 AI 计算平台，其芯片驱动游戏、数据中心、人工智能和高级计算。 |
| xyz:ORCL | 甲骨文 | 追踪 Oracle 1 股普通股价值。Oracle 提供企业软件、云基础设施和数据库管理系统。 |
| xyz:PLTR | Palantir | 追踪 Palantir Technologies 1 股普通股价值。Palantir 开发大规模数据集成和分析的数据分析和 AI 软件平台。 |
| xyz:RIVN | Rivian | 追踪 Rivian Automotive 1 股普通股价值。Rivian 设计和制造电动探险车和商用送货车。 |
| xyz:SKHX | SK海力士 | 追踪 SK hynix 1 股普通股价值。Oracle 将韩元价格按汇率转换。SK hynix 制造 DRAM 和 NAND 存储半导体。 |
| xyz:SMSN | 三星 | 追踪三星电子 1 股普通股价值。Oracle 将韩元价格按汇率转换。三星是半导体、消费电子和显示技术的全球领导者。 |
| xyz:SNDK | 闪迪 | 追踪 Sandisk 1 股普通股价值。Sandisk 开发和制造闪存存储解决方案。 |
| xyz:SOFTBANK | 软银 | 追踪软银集团 1 股普通股价值。Oracle 将日元价格按汇率转换。软银是日本投资控股公司，在科技和电信领域有重大投资。 |
| xyz:TSLA | 特斯拉 | 追踪 Tesla 1 股普通股价值。Tesla 设计制造电动汽车、电池储能系统和太阳能产品，是 EV 和自动驾驶技术全球领导者。 |
| xyz:TSM | 台积电 | 追踪台积电 1 份 ADR 价值。TSMC 是全球最大的专业半导体代工厂，为全球科技公司制造先进芯片。 |
| xyz:EWJ | 日本ETF | 追踪 MSCI Japan ETF 1 份价值。该 ETF 追踪日本股票指数。 |
| xyz:EWY | 韩国ETF | 追踪 MSCI South Korea ETF 1 份价值。该 ETF 追踪韩国股票指数。 |
| xyz:URNM | 铀矿ETF | 追踪 Sprott Uranium Miners ETF 1 份价值。提供铀矿公司和实物铀持仓的敞口。 |
| xyz:USAR | 美国稀土 | 追踪 USA Rare Earth 1 股普通股价值。开发国内稀土供应链，用于关键技术和国防。 |

### xyz — 商品

| 代码 | 中文 | 描述 |
|------|------|------|
| xyz:GOLD | 黄金 | 追踪 1 金衡盎司黄金价值。黄金是全球交易的贵金属，广泛用作价值储存、通胀对冲和央行储备资产。 |
| xyz:SILVER | 白银 | 追踪 1 金衡盎司白银价值。白银既是贵金属也是工业金属，需求覆盖投资市场、电子、太阳能和珠宝。 |
| xyz:CL | 原油(WTI) | 追踪 1 桶西德克萨斯中质原油 (WTI) 价值。WTI 因高品质（低密度、低硫）而成为全球主要原油基准价格。 |
| xyz:BRENTOIL | 布伦特原油 | 追踪 1 桶布伦特原油价值。布伦特是全球主要原油基准，为约 70% 全球交易原油定价。 |
| xyz:COPPER | 铜 | 追踪 1 磅铜的价值。铜是核心工业金属，广泛用于电线、建筑、电子和制造业，是全球制造业需求的晴雨表。 |
| xyz:ALUMINIUM | 铝 | 追踪 1 公吨铝的价值。铝是广泛用于交通、建筑、包装和电力基础设施的轻质工业金属。 |
| xyz:NATGAS | 天然气 | 追踪 1 MMBtu Henry Hub 天然气价值。Henry Hub 是美国天然气定价的主要基准。 |
| xyz:PALLADIUM | 钯金 | 追踪 1 金衡盎司钯金价值。钯金主要用于汽车催化转化器，是排放控制系统的关键投入。 |
| xyz:PLATINUM | 铂金 | 追踪 1 金衡盎司铂金价值。铂金主要需求来自汽车催化剂、珠宝和工业应用。 |

### xyz — 指数 & 外汇

| 代码 | 中文 | 描述 |
|------|------|------|
| xyz:XYZ100 | XYZ100指数 | 追踪 100 家最大、最活跃交易的非金融公司的修正市值加权指数，作为美国大盘科技和成长股基准。 |
| xyz:JP225 | 日经225 | 基于日元的价格加权指数，追踪 225 家日本领先企业，日本股市的主要基准。Oracle 不做汇率转换。 |
| xyz:KR200 | 韩国200 | 基于韩元的市值加权指数，追踪 200 家韩国大盘股，韩国股市的主要基准。Oracle 不做汇率转换。 |
| xyz:DXY | 美元指数 | 追踪美元兑六种主要外币（欧元、日元、英镑、加元、瑞典克朗、瑞郎）的篮子指数。 |
| xyz:EUR | 欧元 | 追踪 EUR/USD 汇率。欧元是欧元区官方货币，全球外汇市场最活跃交易货币之一。 |
| xyz:JPY | 日元 | 追踪 USD/JPY 汇率。日元是日本官方货币，全球外汇市场最活跃交易货币之一。 |

### flx — Felix Exchange

| 代码 | 中文 | 描述 |
|------|------|------|
| flx:GOLD | 黄金 | 追踪黄金价格的永续合约。1 合约代表 1 金衡盎司黄金。 |
| flx:SILVER | 白银 | 追踪白银价格的永续合约。1 合约代表 1 金衡盎司白银。 |
| flx:OIL | 原油 | 追踪 WTI 原油价格的永续合约。1 合约代表 1 桶原油，全球关键能源商品。 |
| flx:COPPER | 铜 | 追踪铜价格的永续合约。1 合约代表 1 磅铜。 |
| flx:PALLADIUM | 钯金 | 追踪钯金价格的永续合约。1 合约代表 1 金衡盎司钯金。 |
| flx:PLATINUM | 铂金 | 追踪铂金价格的永续合约。1 合约代表 1 金衡盎司铂金。 |
| flx:NVDA | 英伟达 | 追踪 NVIDIA 股票价格的永续合约。NVIDIA 设计 GPU、计算平台和 AI 基础设施。 |
| flx:TSLA | 特斯拉 | 追踪 Tesla 股票价格的永续合约。Tesla 是电动汽车和清洁能源公司。 |
| flx:COIN | Coinbase | 追踪 Coinbase 股票价格的永续合约。Coinbase 是美国领先的加密货币交易所。 |
| flx:CRCL | Circle | 追踪 Circle 股票价格的永续合约。Circle 开发基于稳定币和公链的支付技术。 |
| flx:XMR | 门罗币 | 追踪 Monero (XMR) 价格的永续合约。XMR 是隐私导向的加密货币，使用高级密码学隐藏交易信息。 |
| flx:USDE | USDe | 追踪 Ethena USDe 价格的永续合约。USDe 是通过加密衍生品 delta 中性对冲策略维持约 $1 价值的合成美元稳定币。 |

### vntl — Ventuals (板块 ETF & Pre-IPO)

| 代码 | 中文 | 描述 |
|------|------|------|
| vntl:SPACEX | SpaceX | 追踪 SpaceX 总估值（十亿美元计）。如 1 SPACEX = $500，则隐含公司总估值为 $5000亿。 |
| vntl:OPENAI | OpenAI | 追踪 OpenAI 总估值（十亿美元计）。如 1 OPENAI = $500，则隐含公司总估值为 $5000亿。 |
| vntl:ANTHROPIC | Anthropic | 追踪 Anthropic 总估值（十亿美元计）。如 1 ANTHROPIC = $500，则隐含公司总估值为 $5000亿。 |
| vntl:MAG7 | 科技七巨头 | 追踪 MAGS ETF，等权重投资"Magnificent Seven"——Alphabet、Amazon、Apple、Meta、Microsoft、Nvidia 和 Tesla。 |
| vntl:SEMIS | 半导体板块 | 追踪 SMH ETF，投资半导体行业——现代计算的支柱，从智能手机到 AI 数据中心。 |
| vntl:BIOTECH | 生物科技 | 追踪 XBI ETF，等权重捕捉生物科技板块，投资开发下一代药物的小型企业。 |
| vntl:DEFENSE | 国防板块 | 追踪 SHLD ETF，投资 AI 驱动的网络安全、大数据分析和下一代硬件（自主机器人、先进航空航天）。 |
| vntl:ENERGY | 能源板块 | 追踪 XLE ETF，投资美国最大的石油、天然气和燃料企业，覆盖能源生产、炼化和分销。 |
| vntl:INFOTECH | 信息技术 | 追踪 XLK ETF，投资标普500科技板块，覆盖企业软件、云基础设施和硬件制造。 |
| vntl:NUCLEAR | 核能板块 | 追踪 NLR ETF，投资核能复兴全产业链——从铀矿开采、燃料浓缩到核电站建设和无碳基荷电力。 |
| vntl:ROBOT | 机器人板块 | 追踪 BOTZ ETF，投资 AI 和机器人革命——工业自动化、手术机器人和自动驾驶技术的全球领导者。 |
| vntl:GOLDJM | 金矿板块 | 追踪 GDXJ ETF，高贝塔黄金敞口，投资初级矿商——利用 AI 钻探、卫星矿物探测和自主加工的新发现引擎。 |
| vntl:SILVERJM | 银矿板块 | 追踪 SILJ ETF，高贝塔白银敞口，投资初级银矿商——AI 硬件建设和绿色能源转型的关键角色。 |

### km — Kinetiq Markets

| 代码 | 中文 | 描述 |
|------|------|------|
| km:AAPL | 苹果 | Apple Inc. 全球科技公司，设计制造消费电子、软件和数字服务。 |
| km:BABA | 阿里巴巴 | Alibaba Group (ADR). 中国跨国科技集团，运营电商、云计算、数字媒体和金融服务。 |
| km:GOOGL | 谷歌 | Alphabet Inc (A 类). Google 母公司，运营搜索、数字广告、云计算、自动驾驶和 AI 研究。 |
| km:MU | 美光 | Micron Technology. 全球半导体公司，设计制造 DRAM、NAND 闪存和 SSD。 |
| km:NVDA | 英伟达 | NVIDIA Corporation. 领先半导体公司，设计 GPU、数据中心加速器和 AI 计算平台。 |
| km:PLTR | Palantir | Palantir Technologies. 国防导向的数据分析和软件公司，提供 AI 驱动的政府和商业数据平台。 |
| km:TSLA | 特斯拉 | Tesla, Inc. 垂直整合的电动汽车和清洁能源公司。 |
| km:GOLD | 黄金 | 黄金现货价格，广泛被视为价值储存、通胀对冲和市场不确定时期的避险资产。 |
| km:SILVER | 白银 | 白银现货价格，跨投资、电子、太阳能和制造业的贵金属和工业金属。 |
| km:USOIL | 美国原油 | 追踪通过近月期货合约跟踪 WTI 原油日价格变动的美国上市工具。此合约不反映原油现货价格。 |
| km:US500 | 标普500 | 按市值计算美国 500 家最大公司的广泛敞口，覆盖美国股市所有主要板块。 |
| km:USTECH | 纳斯达克 | 美国交易所上市的 100 家最大非金融公司，集中在科技、通信和高增长板块。 |
| km:SMALL2000 | 罗素2000 | 约 2000 家美国小盘上市公司，代表美国股市小市值板块。 |
| km:SEMI | 半导体板块 | 约 30 家美国上市半导体公司，覆盖芯片设计、制造、封装和设备。 |
| km:GLDMINE | 金矿板块 | 全球上市黄金矿业公司的多元化敞口，覆盖大中盘生产商。 |
| km:USENERGY | 美国能源 | 美国上市能源板块公司，覆盖油气勘探、生产、炼化、运输和设备服务。 |

### cash — dreamcash

> cash DEX 暂无 `perpAnnotation` 描述数据。资产包括：AMZN, EWY, GOLD, GOOGL, HOOD, INTC, META, MSFT, NVDA, SILVER, TSLA, USA500。

### hyna — HyENA

> hyna DEX 暂无 `perpAnnotation` 描述数据。主要是加密货币永续合约：BTC, ETH, SOL, HYPE, DOGE, XRP, LTC, LINK, BNB, ADA, BCH, XMR, ENA, SUI, FARTCOIN, IP, LIGHTER, LIT, PUMP, XPL, ZEC。
