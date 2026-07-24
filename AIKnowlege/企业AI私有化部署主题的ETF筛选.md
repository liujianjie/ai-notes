# 企业 AI 私有化部署主题:ETF 筛选与个股补位

> 2026-07 与 Claude 的讨论整理,承接《企业AI私有化部署的算力来源与产业链公司》。核心问题:企业/政府/银行本地部署模型带来的算力需求(租云或自建),有没有对口的 ETF。数据截至 2026 年 7 月,联网核实过。

## 一、先说判断

**国内外都没有一只完全对口的 ETF。**"私有化部署"是需求侧的场景,ETF 是按产业链环节切的,两者对不齐。正确做法:把产业链拆成环节,每个环节找浓度最高的 ETF;浓度不够的环节,只能落到个股。

另一个重要发现:**名字带"算力"的基金别按名字买**。中证有个"算力基础设施主题指数"(931688,中兴/紫光/旭创为主),2023 年 8 家公募扎堆申报过,但公开检索找不到成规模的已上市跟踪产品。现在市场上被叫"算力ETF"的,底层多是云计算或 AI 指数——汇添富 159273 的宣传语就自称"算力ETF",实际跟踪沪港深云计算指数。买之前看跟踪指数,不看名字。

## 二、A股:按环节排优先级

### 第一优先:云计算ETF(中证云计算与大数据指数 930851)

与产业链表重合度最高的一只。截至 2026 年 6 月底前十大权重:科大讯飞、**中科曙光、浪潮信息**、新易盛、**润泽科技**、**紫光股份**、中际旭创、金山办公、中国长城、宏景科技,合计 51%。服务器整机(浪潮/曙光/紫光/长城)+ IDC(润泽)+ 光模块一网打尽,正是"自建算力"那条链。

同指数产品:云计算ETF易方达(516510,约 25 亿)、云计算ETF鹏华(159739)、华夏(516630)。缺点也明显:科大讯飞、金山办公这类软件应用股稀释了硬件浓度,如果行情只在硬件,ETF 会跑输浪潮/曙光个股。

### 第二:通信ETF(515880,中证全指通信设备)

光模块+服务器硬件占比超六成,中兴、中际旭创、新易盛为核心。同类还有通信ETF华夏(515050,2026 年 1 月由"5G通信ETF"改名,光模块/CPO 暴露高)。这是"配套里光模块例外——格局好毛利高"那条判断的最直接工具。

### 第三:科创芯片ETF(588200,科创板芯片指数)

国产算力芯片浓度最高:海光信息、寒武纪、中芯国际都在里面,对应"500 人企业买一体机场景里芯片厂拿大头"的环节。

### 第四:云租赁那条腿

数据敏感度第二档"租专有云"的受益方是大云厂商,A股买不到,走**恒生科技ETF(513180)或中概互联ETF(513050)**(阿里、腾讯)。想 A+H 一把抓可以看新发的云计算沪港深ETF汇添富(159273),港股权重超 26%,软硬比例 6:4。

### 特殊品种:数据中心公募 REITs

南方润泽科技数据中心REIT(180901)、南方万国数据中心REIT(508060),2025 年 8 月上市首日均 30cm 涨停,认购倍数都超 160 倍。唯一直接持有"机柜资产+租金现金流"的场内工具,和"托管 IDC"环节严格对应。注意 REITs 是收租逻辑不是成长逻辑,且上市即被炒过一轮。

### 次选:泛 AI 类

人工智能AIETF(515070)、人工智能ETF(159819)、创业板人工智能(159363/159381)。覆盖全链但什么都不纯,芯片、光模块、应用混在一起。

## 三、美股/海外

| ETF | 跟踪方向 | 贴题度与现状 |
|---|---|---|
| **RACK**(VanEck 数据中心供应链) | 建设+运营+供电数据中心的美股公司,51 只 | 主题最贴:博通、英伟达、美光(HBM)、安费诺、Arista、Vertiv(液冷)+电力股;但 2026 年 6 月刚发行,规模仅约 4800 万美元,费率 0.50%,流动性是硬伤 |
| **DTCR**(Global X 数据中心与数字基建) | 数据中心 REITs 约 52% + 科技股 45% | 规模约 22 亿美元,2026 年涨超 30%;持 Applied Digital 但不含 CoreWeave/Nebius 等 neocloud |
| **SMH / SOXX**(半导体) | 英伟达、台积电、博通、美光 | 芯片+HBM 环节的标准答案,流动性最好,但不含服务器/IDC |
| **AIPO**(Defiance AI 电力基建) | AI 数据中心的供电侧 | 2025 年 7 月发行已超 5 亿美元,是"算力的电力瓶颈"衍生主题,和私有化部署关系较远 |
| **SRVR**(Pacer) | 数据/基建地产,REIT 占 62% | 收租属性重,弹性小 |
| **WAGI**(WisdomTree AI Infrastructure UCITS) | 七大类:数据中心、设备、半导体、服务器供应链、网络、超大云/neocloud | 覆盖面设计得最像本笔记的产业链表,但 2026 年 6 月才在欧洲(Xetra/米兰/瑞士/伦敦)上市,内地投资者难触达 |

## 四、ETF 覆盖不到、只能个股的环节

- **服务器整机**:全球没有纯服务器 ETF。美股就是 SMCI、戴尔、HPE 三选;A股浪潮信息、工业富联、紫光股份已含在云计算指数里。
- **一体机渠道**(神州数码、拓维信息、软通动力):没有任何浓度像样的 ETF。母笔记的判断本来就是"渠道生意、题材属性大于业绩属性",这层不值得用 ETF 重仓。
- **neocloud 云算力出租**:CoreWeave、Nebius、IREN 只能个股买,美股主流数据中心 ETF 明确没纳入。2026 年 7 月这批股票正因 Meta 自建云的传闻大跌,高杠杆模式的风险母笔记已写到。
- **HBM 纯度**:SK 海力士只在韩国上市,ETF 层面最接近的是 iShares 韩国(EWY,三星+海力士合计权重约四成);A股 HBM 外围(通富微电、雅克科技等)散在半导体指数里,无专门 ETF。

## 五、反方理由(什么情况说明这个筛选错了)

母笔记第六节的冷水同样泼在 ETF 上:这些 A股指数成分人尽皆知,估值把两三年预期打满,买 ETF 只是把个股估值风险打了个包,没消掉。另外两点:

1. 云计算指数里软件股占比高,硬件行情下 ETF 跑输个股;
2. RACK/WAGI 这类"完美贴题"的新品,恰恰是主题过热期的发行产物——ETF 发行潮本身常是情绪高点信号。

如果后续 API 价格继续每年跌一个量级、私有化性价比进一步恶化(母笔记第四节的账),这整条"自建算力"链的需求逻辑都会被削弱。

## 六、下一步可核查

- 云计算ETF最新持仓浓度(每季调仓,看软硬比例变化)
- RACK 规模能否上 2 亿美元(流动性及格线)
- 931688 指数是否有新产品获批上市

## 来源

- [中证算力指数成分与申报历史(21经济网)](https://www.21jingji.com/article/20230525/94df9a65959c3dfccabd537957d1fc81.html)
- [8家公募申报算力ETF(证券时报)](https://www.stcn.com/article/detail/3249524.html)
- [515050改名通信ETF华夏(每经网)](https://www.nbd.com.cn/articles/2026-01-12/4215758.html)
- [云计算ETF汇添富159273(东方财富财富号)](https://caifuhao.eastmoney.com/news/20260506142911979929060)
- [中证云计算与大数据指数前十大权重(新浪)](https://k.sina.com.cn/article_7879922977_1d5ae152101901f20u.html)
- [云计算ETF鹏华159739(界面)](https://www.jiemian.com/article/14737297.html)
- [VanEck RACK 发行公告](https://www.businesswire.com/news/home/20260602638779/en/VanEck-Launches-Data-Center-Supply-Chain-ETF-RACK-to-Capture-AI-Infrastructure-Buildout)
- [RACK 持仓(stockanalysis)](https://stockanalysis.com/etf/rack/)
- [DTCR/SRVR 对比(MoneyShow)](https://www.moneyshow.com/articles/dailyguru-65093/dtcr-and-srvr-two-etfs-offering-ai-and-data-center-exposure/)
- [DTCR 不含 neocloud(Investing.com)](https://www.investing.com/analysis/the-ai-data-center-boom-is-bigger-than-a-single-stockthese-etfs-spread-the-bet-200684155)
- [Defiance AIPO 破5亿美元(GlobeNewswire)](https://www.globenewswire.com/news-release/2026/05/05/3287290/0/en/AIPO-Defiance-AI-Power-Infrastructure-ETF-The-First-ETF-Focused-on-AI-Power-Infrastructure-Surpasses-500-Million-in-AUM.html)
- [WisdomTree WAGI 上市(ETF Express)](https://etfexpress.com/2026/06/10/wisdomtree-launches-ai-infrastructure-etf/)
- [数据中心REITs上市(证券时报)](https://www.stcn.com/article/detail/3044289.html)
- [neocloud 大跌(24/7 Wall St.)](https://247wallst.com/investing/2026/07/16/nebius-sinks-13-as-the-neocloud-trade-unravels-how-coreweave-iren-and-the-ai-data-center-stocks-stack-up/)

以上是研究优先级排序,不构成投资建议。
