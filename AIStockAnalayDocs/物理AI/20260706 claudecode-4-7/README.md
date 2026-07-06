# 物理AI 投研工具集 · 2026-07-06

Claude Code 4.7 · Serenity Skill 产出 · 一次会话完成

---

## 背景

围绕孙宇晨"物理AI 是未来三年主线"的说法（源头已核查，是自媒体加工），
用 Serenity 卡点思维梳理**全球 8 家值得研究的物理AI 标的**，
产出四件套投研工具，覆盖"选股 → 尽调 → 时序 → 仓位"完整循环。

**总仓假设**：20 万人民币。

---

## 五件套

| 文件 | 用途 | Artifact URL |
|---|---|---|
| `physical-ai-scorecard.html` | **选股** · 20 家标的 8 维度评分 + 拓扑图 + Top 8 spotlight 雷达图 | [artifact/5f0ceee8](https://claude.ai/code/artifact/5f0ceee8-cf03-4b10-b825-13e02ef355e4) |
| `physical-ai-evidence-cards.html` | **尽调** · Top 8 每家一张证据卡（该查什么财务口径 + IR/披露 URL + 60 天关注） | [artifact/85b732f8](https://claude.ai/code/artifact/85b732f8-8c34-4854-a1e6-1dbc99ef1240) |
| `physical-ai-catalyst-timeline.html` | **时序** · 未来 24 个月催化剂时间轴（8 季度 × Top 8 + 行业 + 宏观） | [artifact/d041ec13](https://claude.ai/code/artifact/d041ec13-0c4f-44fb-94c3-b1a4d95a63d7) |
| `physical-ai-portfolio-simulator.html` | **仓位** · 组合分配 + 压力测试模拟器（20 万人民币，含跨市场约束 + A 股映射 + 4 情境） | [artifact/6a73ba02](https://claude.ai/code/artifact/6a73ba02-bd19-4f9f-8aec-307128e521cc) |
| `sk-hynix-memo.html` | **单家深挖** · SK 海力士 6-8 页备忘录（中等专业度，一分钟速读 + 护城河 + 估值 + 三种未来 + 反方问答 + 90 天清单） | [artifact/c01680f3](https://claude.ai/code/artifact/c01680f3-b73e-4729-a2d3-0b309d5d17f1) |

---

## Top 8 名单

按评分排序（8 维度满分 40）：

| # | 公司 | 代码 | 市场 | 卡的环节 | 分 |
|---|---|---|---|---|---|
| 01 | GE Vernova | GEV | US | 燃气轮机 · 电网 | 38 |
| 02 | 三菱重工 | 7011.T | JP | 燃机 · 防卫综合体 | 37 |
| 03 | 信越化学 | 4063.T | JP | 硅片 · EUV 光刻胶 | 36 |
| 04 | SK 海力士 | 000660.KS | KR | HBM3E / HBM4 | 35 |
| 05 | 台积电 | 2330.TW | TW | CoWoS 先进封装 | 34 |
| 06 | Nabtesco | 6268.T | JP | RV 精密减速器 | 34 |
| 07 | Harmonic Drive | 6324.T | JP | 谐波减速器 | 33 |
| 08 | Keyence | 6861.T | JP | 机器视觉 · 传感 | 31 |

---

## 建议使用顺序

1. **打开 Scorecard** · 先看拓扑图理解产业链结构，再看评分表选出重点
2. **打开 Evidence Cards** · 对 Top 8 每家找到 IR 页，按"该查什么财务口径"栏亲自核对最新财报
3. **打开 Timeline** · 明确未来 60-90 天要盯哪些事件，安排研究节奏
4. **打开 Portfolio Simulator** · 尽调完成后决定仓位分配，用压力测试自我攻击一遍
5. **持续更新** · 每次财报季重复第 2-4 步

---

## 重要声明

- **所有具体数字（backlog、营收、margin、股价）都是 methodology-only 估算**，未跑 live source
- 需按各家 IR 页 / SEC EDGAR / DART / KRX / TDnet / EDINET / MOPS / 巨潮资讯网 亲自核对
- 情境回撤是基于产业结构判断的估算，非历史统计
- 20 万人民币在跨市场存在现实约束：日本股 / 韩国股 / 台湾股大陆用户不能直接购买，
  需通过海外券商（盈透/富途/老虎）或 ETF 代理（EWJ / EWY / TSM ADR）
- 工具用于研究优先度、证据核验、组合设计与压力测试，**不是买入卖出建议**

---

## 关键判断（可能与共识不同）

1. **"孙宇晨物理AI"金句是自媒体加工**，一手证据缺失。TRON DAO 官方 3 月 24 日 10 亿基金定位是 agentic AI 不是 physical AI，是自媒体 bridged
2. **电力才是这轮真短板**，不是芯片。GEV / MHI 两家燃气轮机占 Top 3 里两席
3. **HBM 只买海力士一家**，不同时买三星（同层重叠稀释 alpha）
4. **A 股弹性最大也最贵最挤**（绿的谐波 P/E 100+），适合波段不适合底仓
5. **20 万规模应优先纯 A 股 + 少量美股 ADR（TSM / GEV）**，日韩暴露用 ETF 代理

---

## 待做（如果继续研究）

- ✅ A · SK 海力士单家备忘录（已完成，见 sk-hynix-memo.html）
- C · "卖铲人的卖铲人"横向扫描（味之素 ABF / Timken / Brewer Science / 3M Novec 等二阶三阶受益者）
- D · Top 8 反方论点强度打分（每家 3 条空头最强论点 + 杀伤力评估）
- 其他 Top 8 单家备忘录（GEV / 三菱重工 / 台积电 等，参考 sk-hynix-memo.html 模板）
