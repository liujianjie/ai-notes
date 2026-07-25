# 三家 AI 内存厂商排序：Samsung / SK Hynix / Micron

> 研究日期：2026-06-20
> 研究方法：Serenity.skill 供应链卡点 + 公开证据
> 适用范围：研究优先级排序，**交易决定由用户自定**

---

## 先定卡点，再排公司

AI 算力扩张里，"内存"这一层早就不是普通 DRAM 在卡，而是被压到 **HBM 堆叠 + 先进 base die + NVIDIA 认证** 这三个细分动作上：

1. **HBM4 / HBM4E 12–16 层堆叠** — TSV、MR-MUF、混合键合良率决定能不能交货。
2. **HBM base die 上 TSMC 2/3nm** — HBM4E 这一代起，base die 用先进逻辑工艺，TSMC 产能成新瓶颈。
3. **NVIDIA Rubin / Rubin Ultra 客户认证** — 认证周期 12–18 个月，一旦锁定，份额非常黏。

谁离这三个动作最近，谁就在 AI 浪潮里站得最稳。

---

## 问题 1：谁能"站住脚"？

**全部能站住，但护城河厚度差很多。**

| 公司 | 卡住的环节 | 站稳的根据 | 站稳强度 |
|---|---|---|---|
| **SK Hynix** | HBM4 主力供应 + HBM4E 抢跑 TSMC 3nm base die | NVIDIA Rubin HBM4 拿到 ~66–70% 份额；HBM4E 12 层样品已提前送样 NVIDIA、Microsoft；2026 Q1 HBM 市占 58–62% | **强（核心承重墙）** |
| **Micron** | 12-hi HBM3E 锁定 AMD MI350X；HBM4 Q1 2026 已 HVM | 整个 2026 HBM 产能已售罄；Q2 FY2026 数据中心收入 +47% QoQ；HBM4E 走 TSMC base die | **强（第二供应商不可替代）** |
| **Samsung** | HBM4 拿到 NVIDIA ~30% 份额；HBM 销售 2026 较 2025 三倍 | 2025-09 才通过 HBM3E 12-hi 认证、首批仅 1 万颗；HBM4 logic die 用自家 4nm（vs SK Hynix 用 TSMC 3nm）；非内存板块 2026 仍亏 KRW 3–4 万亿 | **中（HBM 站住，整家公司被 Foundry 拖）** |

**判断**：SK Hynix 和 Micron 是"AI 算力扩一倍，它们必扩"的位置；Samsung 在 HBM 这一层"也站住了"，但整家公司还要等 Foundry 转正才算真正卡死位置。

---

## 问题 2：谁"涨最猛"？

弹性 = 体量小 + 利空已过 + 还没被完全定价。按这个口径：

### 第一档：Micron
- 市值刚过 1T 美元（三家里最小），同样的 HBM 利好对 EPS 杠杆最大。
- Q1 FY2026 收入 $13.64B（+57% YoY），HBM 收入单季破 $10 亿。
- AMD 唯一可信替代供应，AMD MI 系列起来时收入随之翻倍。
- **风险**：DRAM 79% + NAND 21%，NAND 这一块周期下行时拖后腿；HBM4E 量产要到 2027。

### 第二档：Samsung
- 起点最低（HBM 长期掉队），过认证后是"补涨"逻辑，HBM 2026 翻 3 倍是已经卖出的产能。
- HBM4 logic die 价格已上调 40–50%，毛利弹性大。
- **风险**：非内存板块 2026 仍亏 3–4 万亿韩元，HBM 单一板块的利好被整体稀释；HBM4 良率传闻一度 ~65%，比 SK Hynix 慢半步。

### 第三档：SK Hynix
- 2026 YTD 仅 +11.5%（三家里涨幅最小），不是因为基本面差，而是 2024–2025 已涨过头，估值修复完了。
- 上涨弹性更看 HBM 涨价兑现度（2026 提价 ~20% 已被计入）。

**结论**：要"涨最猛"，**Micron** 弹性最直接（小盘 + AMD 锁单 + HBM 业务从零放大）；**Samsung** 是补涨型的猛涨，逻辑成立但需要 Foundry 同时改善。

---

## 问题 3：谁"稳定上涨"？

稳定 = 现金流可预测 + 客户黏性强 + 不依赖单一催化。

**首选：SK Hynix**

理由：
- NVIDIA Rubin（2026）+ Rubin Ultra（2027 HBM4E）两代锁定，3 年订单可见。
- 业务结构最纯：DRAM + NAND，没有 Foundry 这种黑洞业务。
- HBM4E base die 抢先用 TSMC 3nm，先进工艺这一层把 Samsung 甩开半代。
- 2026 Q1 已加入万亿美元市值俱乐部，机构定价模型已经把它当"AI 必备股"对待。
- **风险**：估值已经修复，要继续涨需要 HBM4 提价持续兑现，或 AI capex 超预期。

---

## 一句话总结

> **SK Hynix 是承重墙（最稳）、Micron 是弹簧（最猛）、Samsung 是补涨股（赌 HBM4 + Foundry 双修复）。**

| 场景 | 选择 |
|---|---|
| 只能买一只长持 | **SK Hynix** |
| 博下一波兑现弹性 | **Micron** |
| 赌"折价 + 困境反转" | **Samsung** |

---

## 市场可能没看清的地方

1. **TSMC 2/3nm 成 HBM 新卡点** — 市场仍把 HBM 当"内存厂的事"，但 HBM4E 起 base die 上 TSMC 3nm，SK Hynix 已抢先锁产能，Samsung 仍用自家 4nm，这是工艺代差。
2. **Micron 已经"超过 Samsung"** — 部分研究机构（Astute、Presenc）已经说 Micron 在某些客户的 HBM 份额超过 Samsung，这件事 A 股 / 中文圈讨论不多。
3. **Samsung 的 HBM 利好被非内存板块稀释** — 看公司层财报会觉得"也就那样"，但拆出 HBM 单独看其实非常强；这是估值便宜的根本原因。

---

## 什么情况说明这个判断错了

- **通用反方**：NVIDIA 2026 Q3 / Q4 砍单或 Rubin 出货推迟 → 三家同时杀估值，Micron 跌最多。
- **SK Hynix**：HBM4 2026 末提价（已计入 +20%）兑现不了；或 Samsung HBM4 加速反超抢走 NVIDIA 份额。
- **Micron**：NAND 价格周期下行（21% 业务暴露）；HBM4 走 TSMC base die 出良率事故。
- **Samsung**：HBM4 良率持续低于 80%，无法兑现 NVIDIA 30% 份额；Foundry 亏损 2027 仍未转正。

---

## 下一步研究动作

1. **跟踪 NVIDIA Rubin 出货节奏** — 看 SK Hynix / Samsung 实际拿货比例（NVIDIA 季度 transcript + Counterpoint 季度报告）。
2. **看 Samsung HBM4 良率新闻** — 关键节点是 2026 H2 是否站稳 80%+，决定 30% 份额能否兑现。
3. **Micron Q3 / Q4 FY2026 财报** — 看 HBM 单季收入是否突破 $30 亿年化运行率向 $80 亿迈进；看 NAND 价格指引。
4. **HBM 价格曲线** — 2026 Q3 是否兑现 +20% 涨价（TrendForce 月报）。
5. **SK Hynix HBM4E 客户名单确认** — NVIDIA + Microsoft + Google TPU 拿不拿，决定 2027 估值。

---

## 关键数据速查

| 维度 | SK Hynix | Samsung | Micron |
|---|---|---|---|
| 2026 Q1 HBM 市占 | 58–62% | 21–28% | 18–21% |
| NVIDIA HBM4 份额 | 66–70% | ~30% | 待补 |
| 关键大客户 | NVIDIA、Microsoft、Google TPU | NVIDIA（迟到） | NVIDIA、**AMD（核心）** |
| HBM4E base die 工艺 | TSMC 3nm | 自家 4nm（落后） | TSMC |
| 2026 YTD 涨幅 | +11.5% | +16% | +16.3% |
| Forward P/E | ~6–7× | ~6–7× | 较高 |
| 业务纯度 | 高（DRAM + NAND） | 低（Foundry 亏 3–4 万亿韩元） | 中（NAND 21%） |
| 2026 HBM 产能 | 锁定 | 已售罄 | 已售罄 |

---

## 证据来源

- [TrendForce — SK hynix to supply ~2/3 of NVIDIA HBM4; Samsung targets early delivery (2026-01)](https://www.trendforce.com/news/2026/01/28/news-sk-hynix-reportedly-to-supply-about-two-thirds-of-nvidia-hbm4-samsung-targets-early-delivery/)
- [Counterpoint — Global DRAM and HBM Market Share Quarterly](https://counterpointresearch.com/en/insights/global-dram-and-hbm-market-share)
- [Astute — SK hynix 62% of HBM, Micron overtakes Samsung](https://www.astutegroup.com/news/general/sk-hynix-holds-62-of-hbm-micron-overtakes-samsung-2026-battle-pivots-to-hbm4/)
- [TrendForce — Samsung, SK Hynix plan ~20% HBM3E price hike for 2026](https://www.trendforce.com/news/2025/12/24/news-samsung-sk-hynix-reportedly-plan-20-hbm3e-price-hike-for-2026-as-nvidia-h200-asic-demand-rises/)
- [KED Global — Samsung clears NVIDIA HBM3E 12-Hi (2025-09)](https://www.kedglobal.com/korean-chipmakers/newsView/ked202509190008)
- [KED Global — Samsung sells out 2026 HBM supply after NVIDIA shipments](https://www.kedglobal.com/earnings/newsView/ked202510300005)
- [TweakTown — Samsung passes NVIDIA HBM3E 12-Hi qualification: 10,000 units initial](https://www.tweaktown.com/news/107785/samsung-finally-passes-nvidias-strict-hbm3e-12-hi-qualification-tests-10000-units-on-the-way/index.html)
- [TrendForce — SK hynix may use TSMC 3nm for HBM4E logic dies (2026-03)](https://www.trendforce.com/news/2026/03/20/news-sk-hynix-reportedly-weighs-tsmc-3nm-for-hbm4e-logic-dies-to-gain-edge-over-samsung/)
- [TrendForce — Samsung HBM4 logic die price up 40–50%; 4nm full capacity (2026-04)](https://www.trendforce.com/news/2026/04/14/news-samsung-reportedly-lifts-hbm4-logic-die-prices-by-40-50-amid-ai-boom-4nm-at-full-capacity/)
- [TechTimes — SK hynix ships 12-Layer HBM4E samples ahead of schedule (2026-06)](https://www.techtimes.com/articles/318633/20260619/sk-hynix-ships-12-layer-hbm4e-samples-ahead-schedule-tightening-race-samsung.htm)
- [US News — SK Hynix joins $1T club after Samsung, Micron (2026-05)](https://money.usnews.com/investing/news/articles/2026-05-26/sk-hynix-joins-1-trillion-club-after-samsung-micron-on-ai-chip-boom)
- [TrendForce — Micron CapEx hike to $20B, HBM4 ramps 2Q26](https://www.trendforce.com/news/2025/12/18/news-micron-hikes-capex-to-20b-with-2026-hbm-supply-fully-booked-hbm4-ramps-2q26/)
- [Futurum — Micron Q1 FY2026 record results](https://futurumgroup.com/insights/micron-technology-q1-fy-2026-sets-records-strong-q2-outlook/)
- [Sammy Fans — Why Samsung chip business still losing money despite record Q1 (2026-06)](https://www.sammyfans.com/2026/06/18/why-samsungs-chip-business-is-still-losing-money-despite-record-q1-exynos-foundry-update/)
- [Asiae — Samsung / SK Hynix EPS valuation scenarios (2026-05)](https://www.asiae.co.kr/en/article/stock-etc/2026052709090335125)

---

## 免责声明

本文为基于公开材料的研究方法笔记，所有判断给出研究优先级排序与逻辑链，**不构成投资建议**。证据节点会随时间失效，请在交易前自行核对最新财报、订单与价格数据。
