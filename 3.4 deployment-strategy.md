# 006 部署与商业化策略讨论记录

**日期**：2026-05-03
**状态**：讨论性文档（非 ship 门槛产物），为阶段 1 上线 + 未来 007 提供参考
**背景**：user 在 HBuilderX 编译 H5 成功后询问：①如何访问编译产物；②传统服务器 vs uniCloud 选型；③后端语言 Java Spring / C# / Python 选型；④uniCloud 前端托管与小程序/App 的关系
**关联**：
- 007 C# 重写 spec（commit `0910d5c`，2026-05-02 冻结）—— 本文档不推翻该决策，仅对时机提出参考
- `release-notes.md` 阶段 A/B/C 灰度上线流程
- CLAUDE.md 全局铁律 1-4（密钥 / 合规）

---

## 1. H5 编译产物如何本地访问

HBuilderX build H5 产物位于 `frontend-uniapp/unpackage/dist/build/web/`。

**不能 `file://` 直接打开**（HBuilder 已警告），原因：

1. `manifest.json` H5 路由模式 = `history`（line 12），任意非根路径刷新 404，必须 SPA fallback 到 `index.html`
2. `api/client.ts` 走相对路径 `/api/*`（`VITE_API_BASE` 默认空 → 同源），必须反向代理到后端 `127.0.0.1:8001`

### 三种访问方案

| 方案 | 命令 / 配置 | 适用场景 |
|---|---|---|
| A. vite dev server | `pnpm -C frontend-uniapp run dev:h5` → `http://127.0.0.1:5174` | 日常开发（`vite.config.ts:76-85` 已配 `/api` proxy） |
| B. vite preview | 补 `preview.proxy` 后 `npx vite preview` | 验证 build 产物 + PWA |
| C. nginx for Windows | 见下方 `nginx.conf` 片段 | 贴近生产的验收演练（对齐 T112-T113 灰度） |

### 方案 C 参考 nginx 配置

```nginx
server {
  listen 8080;
  root  F:/AIProject/StockTrading/frontend-uniapp/unpackage/dist/build/web;
  index index.html;

  location / { try_files $uri $uri/ /index.html; }

  location /api/ {
    proxy_pass http://127.0.0.1:8001;
    proxy_set_header Host $host;
  }
  location /health { proxy_pass http://127.0.0.1:8001; }
}
```

---

## 2. 后端语言选型对比

### 2.1 决策矩阵

| 维度 | Python (FastAPI) | C# (.NET 8) | Java Spring Boot |
|---|---|---|---|
| 单人开发速度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Decimal 金额处理 | `Decimal` 完美 | `decimal` **原生**最强 | `BigDecimal` 啰嗦 |
| OCR / Anthropic SDK 生态 | 官方 Python SDK | .NET 社区 SDK 可用 | 第三方为主 |
| 启动时间 / 内存 | 50 MB | AOT 后 30 MB 单文件 | JVM 冷启 300 MB+ |
| 2H4G 小机器友好度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Windows 本地开发 | OK | 最丝滑（VS / Rider） | OK |
| 部署形态 | uvicorn + systemd | `dotnet publish -p:PublishAot=true` 单文件 | fat jar + systemd |
| 国内招人（未来扩团队） | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 样板代码量 | 最少 | 少 | 最多 |
| 类型安全（跨端契约稳定性） | pydantic 够用 | **最强** | 强 |
| 长期维护心智负担 | 中（GIL / 动态类型） | 低 | 高（Spring 黑魔法） |

### 2.2 结论

- **Java Spring 直接排除**：单人开发 + 小机器 + 本项目复杂度下，Spring 的模板代码、JVM 内存、Maven/Gradle 维护成本全是纯负担，其优势（大团队 / 复杂分布式 / 招人）在本阶段 0 收益。
- **Python vs C# 才是真正选择**：
  - **继续 Python**：零迁移成本，001-005 全部不变量（payload_sha256 / Decimal-only / append-only / 进程隔离 / 后端零出站）自动保留，立即可进入阶段 1 上线
  - **迁 C#（已是 007 spec 计划）**：`decimal` 原生类型、AOT 单文件、Windows 开发体验、EF Core + PostgreSQL 迁移工具链更体系；代价是 2-4 周重写 + 不变量重新验证

### 2.3 对 007 时机的建议（参考，不推翻 2026-05-02 决策）

- **目标"赶紧上线收钱"** → 先用 Python 跑阶段 1 拿真实反馈与现金流，007 延后 2-3 个月
- **目标"打造长期可维护独立产品"** → 007 按原计划，但建议放在阶段 1 上线观察期之后，而非立即
- **决策不变的情况**：阶段 1 用 Python + 阶段 2 迁 C# 是合理路径，007 作为"迁移期 feature"而非"上线前置依赖"

---

## 3. uniCloud 概念澄清与多端访问模型

### 3.1 uniCloud 实际上是三个独立产品

user 提问中"前端托管 + 小程序/App 访问 uniCloud 服务器"包含概念混淆，必须先分清：

| 产品 | 作用 | 传统对应 |
|---|---|---|
| uniCloud 前端网页托管 | 静态文件 CDN（仅服务 H5 浏览器用户） | 阿里云 OSS + CDN / Vercel |
| uniCloud 云函数 | Node.js serverless 后端运行时 | AWS Lambda / 自建 FastAPI |
| uniCloud 云数据库 | 云函数连接的 MongoDB 风格 DB | SQLite / PostgreSQL |

### 3.2 关键事实

- **小程序 / App 不"访问"前端托管**：小程序是 `wxapkg` 包本地运行，App 是 apk 本地运行，它们不从 web 加载 HTML
- 前端托管只对 **H5 浏览器用户**有意义
- 小程序 / App / H5 三端统一需要的是**后端 API 域名**，不是前端静态文件

### 3.3 正确的三端访问架构

```
  H5 用户浏览器     →  https://yourdomain.com (静态 HTML/JS)
                       [ uniCloud 前端托管 或 自建 nginx ]

  小程序（wxapkg）  ↓
  Android App     ↓   三端都通过 HTTPS 调用同一个 API 域名
  H5（JS 发请求）  ↓
                       https://api.yourdomain.com/api/*
                       [ 你自己的后端服务器 — 任意语言 ]
```

小程序后台"request 合法域名"白名单配的是 `api.yourdomain.com`，与前端托管域名无关。

### 3.4 后端三种落地方式对比

| 后端方案 | 小程序 | App | H5 | 需做的事 |
|---|---|---|---|---|
| 自建服务器（阿里云轻量 + Python/C#） | ✅ 配白名单 | ✅ | ✅ | ICP 备案 + nginx + systemd |
| uniCloud 云函数 | ✅ `uniCloud.callFunction()` | ✅ | ✅ | Node 重写后端 + DCloud 生态锁定 |
| 混合（前端托管 + 自建后端） | ✅ | ✅ | ✅ | 备案 + 双处维护 |

### 3.5 结论

**不推荐购买 uniCloud 前端托管**：
- 会造成两处维护成本（uniCloud 后台 + 自建后端）
- DCloud 生态锁定风险
- 省下的工作仅 nginx 配置，本项目 user 已具备能力
- 对阶段 1 规模无意义

**uniCloud 前端托管合适的人**：完全不想碰服务器 + 纯静态展示站 + 不在乎锁定 → 本项目不符合

**不推荐迁 uniCloud 云函数**：需把 FastAPI 重写为 Node.js + 云数据库，与 007 C# 方向冲突，且 001-005 不变量重新论证成本极高

---

## 4. 分阶段上线路径

### 阶段 0：纯自用零成本期（2026-05-03 补充）

**前提**：仅自己一个人用，不对外，只图省钱 + 验证长期稳定性。

**推荐组合：家里设备 + Cloudflare Tunnel（免费）**

| 组件 | 方案 | 成本 |
|---|---|---|
| 运行宿主 | 家里电脑长开 / NAS / 树莓派 4B（二手 ~300 元） | 电费约 25-50 RMB/年（树莓派）到 150 RMB/年（PC 24h） |
| 容器化 | Docker Compose（后端 + PostgreSQL + nginx 三件套） | 0 |
| 公网入口 | Cloudflare Tunnel（`cloudflared`，个人免费、无带宽硬限、自动 HTTPS） | 0（无域名用 `*.trycloudflare.com`）/ 55 RMB/年（自有域名） |
| H5 静态 | 和后端同一台跑 nginx，或 Cloudflare Pages 免费托管 | 0 |
| 数据备份 | 每日 `pg_dump` → rclone 推到 OneDrive / 百度网盘免费空间 | 0 |
| 小程序 | 个人主体真机调试（不提审、不上架）| 0 |
| App | HBuilderX 云打包（免费额度内）→ 自己手机 adb 装 | 0 |

**关键：Cloudflare Tunnel 天然解决三个痛点**：

1. **无公网 IP 也能被访问**（家里宽带 NAT 后也能 serve）
2. **自动 HTTPS + 自动 DDoS 防护**（比自建 nginx + certbot 省心）
3. **无需备案**（入口是 Cloudflare 的边缘节点，**仅限自用**；面向国内公众使用时此路径存在合规灰区，转阶段 1 时必须切）

**⚠️ 淘汰的"看似免费"方案**（不推荐）：

| 方案 | 问题 |
|---|---|
| Vercel / Netlify 免费层 | 只支持静态 + serverless，Python FastAPI 要重写；国内访问慢 |
| Supabase / Firebase 免费层 | 厂商锁定 + 国内访问慢 + 数据出境合规风险 |
| Oracle Cloud Free Tier（ARM 永久免费） | 新加坡/日本节点，国内访问 200-400 ms；小程序白名单需要 HTTPS 域名，跨境延迟影响体验 |
| 花生壳 / ngrok 免费版 | 带宽 1 Mbps、随机域名、每次重启变 URL |
| Tailscale Funnel 免费版 | 客户端要装 Tailscale，小程序 / App 装不了 → 不适合本项目 |

### 阶段 0 → 阶段 1 的丝滑转换设计

**本项目阶段 0 设计时就应预留的三个迁移友好点**：

1. **后端用 Docker + `docker-compose.yml` 打包**（不裸跑 uvicorn）
   → 阶段 1 迁阿里云时同一个镜像 `docker compose up -d`，0 代码改动
2. **数据库从阶段 0 就用 PostgreSQL**（不用 SQLite）
   → 避开"单用户 SQLite → 多用户 PG"再次迁移的二次踩坑；PG 单机 2H4G 性能足够撑 100+ 用户
   → 若 user 偏好继续 SQLite，保留 `DATABASE_URL` 环境变量抽象，迁移时改一行
3. **域名从阶段 0 就买 + DNS 用 Cloudflare 管理**（不走 `*.trycloudflare.com`）
   → 阶段 1 切服务器时只改一条 DNS A 记录，小程序白名单 / App 打包域名零感知

**转换步骤清单**（预估 3-7 天，不含备案等待期）：

| Day | 动作 |
|---|---|
| D1 | 购阿里云轻量 2H4G（首年 ~100 RMB），域名 A 记录先不切 |
| D1 | 新机 `docker compose up -d`（同一份 compose 文件）+ `pg_dump` 从家里机器导过去 |
| D2 | 提交 ICP 备案（阿里云通道） |
| D2-D20 | 备案等待期仍用阶段 0 路径 serve 自用流量，新机仅做灰度 |
| 备案通过 | DNS A 记录切到新机 IP，Cloudflare Tunnel 保留一周作回滚兜底 |
| +7 天 | 确认稳定后关闭 Cloudflare Tunnel，阶段 0 机器降级为冷备份 |
| 小程序 | 若 API 域名不变（推荐），小程序白名单 0 修改；若变，后台改一处 |

**零代码改动**：只要阶段 0 遵守"Docker + PG + 自有域名"三条规则，阶段 1 切换不动业务代码。

### 阶段 1：自用 + 白名单少量用户（现在 ~ 半年）

- **服务器**：1 台阿里云轻量 2H4G 100 GB SSD，约 96 RMB/月包年 540/年
- **前端**：nginx 静态托管 `frontend-uniapp/dist/build/h5/`（对齐 `release-notes.md` 阶段 A/B）
- **后端**：FastAPI + systemd 守护 + SQLite + 每日 cron 备份
- **域名**：ICP 备案（阿里云通道，15-20 工作日）
- **小程序**：个人主体（不能收费但能跑 T088 MP 三核心真机 + 真机调试）
- **备案期过渡**：IP + 自签 HTTPS 内部测

### 阶段 2：开始多人付费（半年后）

**合规先行**（技术比这个简单）：

- **主体**：个体工商户起步（微信支付 / 支付宝商户号都能开），体量大了再升企业
- **经营范围**：避开"证券咨询 / 投资顾问"，用"信息技术服务 / 软件开发 / 数据处理"
- **定位红线**：产品必须是"记账工具"，**不能**出现投资建议 / 荐股 / 买入卖出建议
- **小程序主体**：切换为工商主体（个人号不能挂支付）
- **合规测评**：等保 2.0 二级测评（涉及用户数据 + 支付，一次性约 1-2 万 RMB）

**技术升级**：

- SQLite → PostgreSQL（SQLite 单写锁，10+ 并发会卡）
- 服务器升 4H8G + Redis 缓存
- 007 C# 重写按原计划执行（此时已有现金流支撑 2-4 周重写期）

### 阶段 3：规模化

- CDN + 多机部署 + 对象存储放 OCR 图片
- 数据库读写分离
- 此阶段再谈架构不迟

---

## 5. 合规与风险提醒

### 5.1 备案

- 境内域名 + 境内服务器**必须** ICP 备案，无论走阿里云 ECS / 轻量 / uniCloud 阿里云版 / 腾讯云版，都是**同一个备案通道**，不会更快
- 周期约 15-20 工作日
- 香港 / 新加坡节点可跳备案但不能正经商用

### 5.2 项目名称与上架风险

项目目录名 `StockTrading`、类别含"进攻组合 / 永久组合"等词，**小程序提审会被微信重点审核**。规避建议：

- 对外产品名**避免**"股票交易 / 投顾 / 荐股"字样，改用"资产配置记账 / 个人理财追踪"
- 所有 UI 文案**不出现**具体股票代码作为推荐
- `DevNote/开发日记.md` 如含个股判断**必须**被 `check-*.mjs` 阻止打进生产包（user 已约定 006 期 0 修改此文件）

### 5.3 知识产权

长期订阅付费建议注册软件著作权（成本几百元，防抄袭 + 合规加分）。

---

## 6. 上线 Todo（独立于 006 ship 门槛）

> 本节为未来动作清单，**不**阻塞 006 ship 门槛（自动化六步 + 手动七步）。006 完工后再启动。

### 6.1 阶段 0 自用启动（零成本，可立即开始）

- [ ] 起草 `docker-compose.yml`（后端 + PostgreSQL + nginx，避免未来二次迁移）
- [ ] 宿主选型：家里长开 PC / 二手树莓派 4B / 已有 NAS
- [ ] Cloudflare 账号 + 注册域名（`.com` 约 55 RMB/年，可选；最省走 `trycloudflare.com`）
- [ ] `cloudflared` 隧道配置（一条命令起 HTTPS 公网入口）
- [ ] 本地 `pg_dump` + rclone 推 OneDrive / 百度云每日备份 cron
- [ ] 小程序个人主体真机调试走隧道域名（配白名单）

### 6.2 阶段 1 开放转换（阶段 0 稳定后）

- [ ] 阿里云轻量 2H4G 购买 + ICP 备案提交
- [ ] 域名 `yourdomain.com`（主站）+ `api.yourdomain.com`（API）规划
- [ ] 起草部署脚本三件套：`nginx.conf` + `systemd unit` + `backup.cron`
- [ ] 微信小程序个人主体注册 → 拿真实 `wx` AppID → 回填 `manifest.json` line 22（**不 commit** 真实值）
- [ ] HBuilderX 云打包注册 → 拿真实 `__UNI__` AppID → 回填 `manifest.json` line 3（**不 commit** 真实值）
- [ ] 决定 `manifest.json` 是否加入 `.gitignore` + 是否补 `manifest.template.json`
- [ ] 阶段 1 后 2-3 个月观察期结束，再评估是否立即启动 007 C# 重写

---

## 7. 双应用共存：量化交易工具 + 本项目（2026-05-03 补充）

> 本节为**通用指南**。user 的量化工具具体技术栈 / 券商 API / 日间峰值资源待明确后，应据此收敛具体方案。

### 7.1 量化工具与本项目的根本差异

这两个应用看似都是"炒股相关"，但**运维特征完全相反**，不能按同一标准对待：

| 维度 | 资产配置追踪器（本项目） | 量化交易工具 |
|---|---|---|
| 网络方向 | 入站（手机/浏览器调用） | 出站（连券商 / 行情 API） |
| 时延要求 | 秒级，Cloudflare Tunnel 够 | 毫秒级，越接近券商越好 |
| 运行窗口 | 24×7（允许短暂中断） | 交易时段强制在线（A 股 9:30-15:00 约 4h/日） |
| 宕机后果 | 用户等一会再刷 | **丢单 / 踩仓位 / 真金白银损失** |
| 数据吞吐 | 每日几 KB 更新 | 每秒 tick 数百到数千行 |
| 密钥敏感度 | OCR Key（仅扣费） | 券商 API Key（**能直接动钱，最高级**） |
| 备份频率 | 每日 cron | 交易日志实时多副本 |
| 网络入口方案 | Cloudflare Tunnel 合适 | **绝不**走 Tunnel（额外 10-50 ms 可致掉单） |
| 容器化适配 | Docker 完美 | 部分场景不行（见 §8.2） |

### 7.2 一个关键前置判断：量化工具能否容器化

**判断树**：

1. 量化框架是 Python / Rust / Go 纯代码 + REST/WebSocket 券商 API（如 vn.py、zipline、qmt 的 Python SDK 部分）
   → 可 Docker 化，进入 §7.3 方案 A/B
2. 量化工具依赖 **Windows COM 组件 / 券商桌面客户端绑定**（如同花顺 iFinD、通达信 TDX、部分国内券商 QMT Windows 终端）
   → 不能 Docker，必须 Windows 宿主机 native 运行，进入 §7.3 方案 B/C
3. 需要 GPU 做实时推断（深度学习策略）
   → 方案 A 基础上额外要求宿主机有 CUDA GPU + 资源独占

**user 务必先确认自己的量化栈落在哪一类**，否则下面方案选错。

### 7.3 共存部署三方案

#### 方案 A：单台 mini PC + Docker 双容器（最省钱，~1500 RMB 一次性）

```
┌────────── mini PC 16GB RAM / i5 ──────────┐
│ Docker Compose                              │
│  ├─ tracker: FastAPI + PG (2GB limit)       │
│  ├─ quant:   量化框架 + 自有 DB (10GB limit)│
│  └─ cloudflared: 仅暴露 tracker 入站         │
└───────────────────────────────────────────┘
     ↓ outbound                    ↑ inbound
  券商 API                     手机访问记账
```

- 资源限制铁律：`deploy.resources.limits.cpus` / `memory` 写死，量化不能把记账饿死
- 适合：量化策略轻量、T+1 或分钟级、无 Windows COM 依赖

#### 方案 B：两台设备物理隔离（最稳，~2000 RMB 一次性）

```
┌─ 主力 PC（量化，Windows 或 Linux）─┐   ┌─ 树莓派 4B 或 NAS（记账）─┐
│ 量化工具 native 运行              │   │ Docker + tracker + PG       │
│ UPS 500W 直连                     │   │ Cloudflare Tunnel           │
│ 有线网络 + NTP 强同步             │   │ 每日 cron 备份              │
└─────────────────────────────────┘   └───────────────────────────┘
          ↓ 券商                              ↑ 手机
```

- 一台崩绝不波及另一台
- 交易 PC 做系统升级可挑非交易日，记账机任何时候升级都无所谓
- 适合：量化吃重资源 / 必须 Windows native / 对可用性要求高

#### 方案 C：量化仅交易时段跑 + 记账永开（降低峰值需求，硬件最小）

```
平日 00:00 - 09:29  → 仅记账容器运行，量化 systemd unit inactive
交易 09:30 - 15:00  → 量化 systemd 启动，两者并行
收盘 15:01 - 次日   → 量化 stop，可能触发收盘后 batch（backtest / 日结）
```

- systemd timer 或 cron 精确定时启停
- 单台 8GB RAM 够用（记账 500MB + 量化峰值 6GB）
- 缺点：启停逻辑要写稳；美股策略要跑夜间时段此方案复杂

### 7.4 共存运行的铁律

无论选 A/B/C，这些不能妥协：

1. **数据库物理分离**：不共用一个 PG 实例 / 不共用一份 SQLite 文件。交易审计表和记账表混在同个 DB 是运维灾难（一次误操作同时毁两份）
2. **OS 用户分离**：`useradd tracker` + `useradd quant`，互不能读对方目录，防一个应用被攻破横向扩散
3. **密钥隔离**：券商 API Key 和 Anthropic OCR Key **分别**存两个不同的 secrets 目录 / systemd EnvironmentFile，任一应用读不到对方的环境变量
4. **日志分目录**：`/var/log/tracker/` vs `/var/log/quant/`，交易日志按日切割 + 永不删除（监管审计需求）
5. **磁盘空间监控**：tick 数据膨胀很快，设 80% 告警，防量化把磁盘撑爆让记账也挂
6. **量化不走 Cloudflare Tunnel**：Tunnel 只暴露记账端口，量化纯 outbound 不需要入站
7. **电力 UPS**：交易时段断电 = 钱包流血。家用小型 UPS（500-800 RMB）保 15 分钟过渡 + 优雅关机

### 7.5 告警分级（两个应用的 SLA 差几个数量级）

| 场景 | 记账挂了 | 量化挂了 |
|---|---|---|
| 通知方式 | 微信推送 / 邮件即可 | **电话 + 短信**（Twilio / 阿里云语音 VMS） |
| 响应 SLA | 24h 修即可 | 交易时段内 **5 分钟**必须有人 |
| 自动行为 | 无 | 自动平仓 / 取消未成交单 / 切人工盯盘（看策略设计） |

### 7.6 转阶段 1（记账开放使用）时的连带影响

**记账上云，量化继续留家里**：

- 券商 API 对延迟敏感，且多数国内券商 API 不允许在公有云 IP 段访问（风控策略），量化**不能**迁阿里云
- 两个应用从阶段 1 开始正式物理分离 → 方案 A/C 要切到方案 B
- 只保留代码层共享（如果有共用的 Decimal 工具、财务函数）→ 建议独立成 pip / npm 内部包，不 copy-paste

### 7.7 Todo 补充

- [ ] 界定量化工具的技术栈 / 是否依赖 Windows COM / 日间峰值资源（决定 §7.3 方案）
- [ ] 家庭宽带稳定性压测（交易时段连续 4h 无掉线 + 延迟到券商 < 50ms）
- [ ] UPS 预算 + 容量核算（目标：断电 15 分钟优雅停机）
- [ ] 两个应用 compose yaml / systemd unit 独立编写，不合一
- [ ] 告警服务选型（Twilio 国际 / 阿里云语音服务境内）
- [ ] 交易日志实时备份方案（rsyslog 远程 / S3 实时上传 / 云数据库追加）
- [ ] 如果量化涉及自动下单实盘，评估是否需要合规资质（个人账户程序化交易目前灰色，部分券商有合规要求）

---

## 8. 本文档的定位

- **不**是 006 ship 门槛产物（`plan.md` / `tasks.md` 不引用此文档）
- **不**推翻 007 C# 重写决策，仅对时机提供参考视角
- **是**一次性的策略讨论快照，用于未来部署阶段查阅
- 若 user 后续做出与本文档不同的决策，以新决策为准，本文档不需更新
