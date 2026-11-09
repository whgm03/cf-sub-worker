# 🚀 Cloudflare Worker Preferred-IP 智能订阅与节点中枢

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/crossgg/cf-sub)
[![Deploy Status](https://github.com/crossgg/cf-sub/actions/workflows/deploy.yml/badge.svg)](https://github.com/crossgg/cf-sub/actions/workflows/deploy.yml)
[![Cloudflare Workers](https://img.shields.io/badge/Platform-Cloudflare%20Workers-F38020?style=flat&logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/)

基于 Cloudflare Workers 打造的轻量级、高可用代理订阅系统与 Web 管理平台。原生支持 **VLESS over WebSocket**、**电信/联通/移动三网实测优选**、**OpenAI 纯美专线分流**与多协议客户端订阅转换。

无需自备云服务器，完全依托 Cloudflare 免费层（Workers + KV）运行！

---

## 🎯 一键部署到 Cloudflare（三种方式任选其一）

| 部署方式 | 特点 | 推荐指数 | 适用场景 |
| :--- | :--- | :---: | :--- |
| **[方式一：Cloudflare 官方一键部署按钮](#-方式一cloudflare-官方一键部署按钮)** | 点击按钮直达 CF 控制台，自动授权导入 | ⭐⭐⭐⭐ | 想通过官方网页流程部署 |
| **[方式二：GitHub Actions 一键自动部署](#-方式二github-actions-一键自动部署全自动创建-kv-与-secrets推荐)** | **全自动创建 KV、自动生成 UUID/密钥**，最少仅需 1 个 Token | ⭐⭐⭐⭐⭐ | **强烈推荐**，最省心且永久自动维护 |
| **[方式三：CF 控制台直接粘贴代码](#-方式三cf-控制台直接复制代码部署小白速通)** | 复制 `_worker.js` 代码直接粘贴发布 | ⭐⭐⭐ | 仅测试体验，不想配置 Git |

---

### 🌐 方式一：Cloudflare 官方一键部署按钮

点击下方按钮直接在 Cloudflare 中创建并部署本 Worker：

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/crossgg/cf-sub)

> 💡 **使用提示**：
> 1. 点击上方按钮后，Cloudflare 会引导你授权 GitHub 并自动将本仓库连接到你的 Cloudflare Workers；
> 2. 部署完成后，在 Cloudflare 控制台 Worker 设置中绑定一个名为 `XS_SUB_KV` 的 KV 命名空间，并添加环境变量即可。
> 3. *（已绑定仓库 `crossgg/cf-sub`，点击即可一键直达部署）*

---

### 🤖 方式二：GitHub Actions 一键自动部署（全自动创建 KV 与 Secrets · 推荐）

如果你希望**完全不用手动去 Cloudflare 创建 KV 数据库、不用手动生成 UUID 和密码**，推荐使用内置的 GitHub Actions 自动化工作流。

#### 第一步：添加 1 个必填的 GitHub Secret
1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)，右上角点击 **头像** -> **My Profile** -> **API Tokens** -> **Create Token**；
2. 选择 **Edit Cloudflare Workers** 模板创建一个 Token（包含 Workers、KV 与路由权限）；
3. 打开仓库密钥设置页：[👉 点击前往 Secrets 设置页面](https://github.com/crossgg/cf-sub/settings/secrets/actions)，点击 **New repository secret**：

| Secret 变量名 | 是否必填 | 作用与说明 |
| :--- | :---: | :--- |
| `CF_API_TOKEN` | **必填** | 上一步生成的 Cloudflare API Token |
| `DOMAIN` | 选填 | 你的自定义子域名（如 `sub.yourdomain.com`，主域名需在同账号托管；不填则使用免费 workers.dev 域名） |
| `ADMIN_PASSWORD` | 选填 | 管理员后台密码（若不填，系统将**自动生成 16 位强密码**并在摘要中展示） |

#### 第二步：一键运行部署
1. 推送代码到 `main` 分支，或直接打开：[👉 点击前往 Actions 运行页面](https://github.com/crossgg/cf-sub/actions/workflows/deploy.yml)；
2. 点击右侧 **Run workflow** 按钮；
3. 工作流将**全自动执行**：
   - 自动检测并创建绑定的 KV 数据库（`XS_SUB_KV`）；
   - 自动生成 `VLESS_UUID_V9` 与安全密钥；
   - 自动发布并生成登录初始密码！
4. 执行完成后，点击进入该次运行的 **Summary（摘要）** 卡片，即可查看到管理员邮箱与初始登录密码。

---

### 📋 方式三：CF 控制台直接复制代码部署（小白速通）

1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)，点击 **Workers & Pages** -> **Create** -> **Create Worker**；
2. 输入任意名称（如 `cf-sub`），点击 **Deploy**；
3. 点击 **Edit code**，清空默认代码，将本项目中的 [`_worker.js`](_worker.js) 全部内容复制并粘贴进去，点击 **Deploy** 保存；
4. *(可选，推荐)*：在左侧 **Storage & Databases** -> **KV** 中创建名为 `XS_SUB_KV` 的命名空间，并在 Worker 的 **Settings** -> **Bindings** 中绑定为 `XS_SUB_KV`。

---

## ✨ 核心特性

- ⚡ **三网实测智能优选 (Preferred-IP)**
  - 自动轮转 Cloudflare 官方各网段优选候选 IP，结合外部分布式探针为**移动 (CM)、联通 (CU)、电信 (CT)** 定制最优低延迟节点拓扑。
  - 支持多网探测、TCP/TLS 握手校验与自动熔断降级。
- 🛡️ **严格精准分流策略**
  - **🤖 OpenAI / ChatGPT / Claude 纯美专线**：铁律隔离，绝不混入香港/日本/台湾出口，防止账号风控。
  - **🌐 Google 防跳转**：智能重定向 Google 服务，避免亚洲出口误判地区。
  - **🛑 内置广告拦截**：自动过滤主流广告追踪与恶意流量。
- 📱 **全平台客户端原生适配**
  - 自动渲染输出 **Clash / Mihomo (Clash Meta)** 完整规则组；
  - 自动输出 **Sing-box** 现代化 JSON 配置；
  - 支持通用 **VLESS Base64** 节点链接（适配 v2rayN、Shadowrocket 等）。
- 🖥️ **现代化可视化管理大屏**
  - 内置 Web 管理仪表盘，实时查看优选节点状态、出口 IP 拓扑与每日 100,000 次免费配额消耗；
  - 支持客户端隔离：为不同设备或用户独立签发专属 HMAC 订阅 Token。

---

## 🖥️ 管理后台与订阅使用

1. 打开部署好的 Worker 域名：`https://<你的域名>/`；
2. 输入 **管理员邮箱**（默认 `admin@<你的域名>` 或 `admin@example.com`）与 **管理员密码** 登录后台；
3. **签发专属客户端订阅**：
   - 在后台控制面板中点击“添加客户端”，为你的手机、电脑签发专属 Token；
   - 复制生成的 Clash / Mihomo / Sing-box / VLESS 订阅链接并导入客户端即可使用！

---

## ❓ 常见问题 (FAQ)

### Q1: `PROXYIP` 是做什么用的？必须填吗？
* **回答：默认留空即可，不需要填。**
* **原理**：Cloudflare 禁止 Worker 直接回环连接到 Cloudflare 自己的 CDN 节点（会报 `Error 1000`）。当 Worker 代理的目标也是托管在 Cloudflare 上的站点时，`PROXYIP` 作为外部中转跳板代为转发。
* **实际影响**：日常访问 Google, YouTube, GitHub, Twitter 等绝大部分主流网站走的是 Worker 原生直连（速度最快、延迟最低）。只有在遇到极少数套了 Cloudflare CDN 且打不开的站点时，才需要在后台添加 `PROXYIP`。

### Q2: 每天 100,000 次免费额度够用吗？
* **回答：个人或小团体完全绰绰有余。**
* 节点订阅接口默认启用了 300 秒边缘缓存，并且客户端通常设置数小时更新一次。正常使用下，单人每天消耗的请求数仅几十到上百次。

### Q3: 为什么绑定自定义域名提示冲突？
* 若在 Cloudflare 添加了自定义域名报错，请检查该子域名在 Cloudflare DNS 列表中是否已有 A 或 CNAME 记录。如果有，请先在 DNS 页面删除同名记录，Wrangler 部署时会自动生成路由记录并签发 SSL 证书。

---

## 📄 开源许可与免责声明

- 本项目仅供网络技术研究、学习与分布式网络性能测试使用。
- 请遵守 Cloudflare 服务条款（ToS）及当地法律法规。
