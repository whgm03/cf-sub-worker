# Cloudflare Worker 部署指南 · Preferred-IP 三网实测订阅

本项目基于 Cloudflare Workers 运行，提供 VLESS over WebSocket、智能 Preferred-IP 优选与多协议订阅分发功能。

项目仓库地址：[crossgg/cf-sub](https://github.com/crossgg/cf-sub)

---

## 方式一：GitHub Actions 一键自动部署（强烈推荐 · 极简模式）

通过 GitHub Actions CI/CD，你**无需手动创建 KV、无需手动生成 UUID 或密钥**，最少仅需提供 1 个 Cloudflare API Token 即可全自动完成初始化与发布。

### 1. 获取 Cloudflare API Token
1. 登录 [Cloudflare 控制台](https://dash.cloudflare.com/)。

2. 点击右上角 **头像** -> **My Profile** -> **API Tokens** -> **Create Token**。

3. 选择 **Edit Cloudflare Workers** 模板完成创建，并复制生成的 Token 字符串。

   可选配置：

   **`PROXYIP`** ：43.168.16.112,161.33.135.154 （留空直连）

     **`VLESS_UUID_V9`** xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  （留空自动生成）

### 2. 在 GitHub 仓库设置 Secrets
直接进入仓库设置页：[👉 点击前往 Secrets 设置页面](https://github.com/crossgg/cf-sub/settings/secrets/actions)，点击 **New repository secret**：

| Secret 变量名 | 是否必填 | 作用与说明 |
| :--- | :--- | :--- |
| `CF_API_TOKEN` | **必填** | 上一步生成的 Cloudflare API Token |
| `DOMAIN` | **选填** | 你的自定义子域名（例如 `sub.example.com`）。主域名必须已托管在同一个 CF 账号下；若不填则默认使用 workers.dev 域名 |
| `ADMIN_PASSWORD` | **选填** | 管理员密码。若不填，GitHub Actions 会在首次部署时自动生成一个随机强密码并在部署总结（Summary）中输出 |

### 3. 一键部署
* 推送代码到 `main` 分支，或者在仓库页面点击：[👉 前往 Actions 运行工作流](https://github.com/crossgg/cf-sub/actions/workflows/deploy.yml)，点击右侧 **Run workflow**。
* 工作流会自动：
  1. 识别并连接你的 Cloudflare 账户；
  2. 自动检查并创建绑定的 KV 数据库（`XS_SUB_KV`）；
  3. 自动生成并安全注入 `VLESS_UUID_V9` 与 `TOKEN_SECRET_V9`（已存在则保留，不覆盖老配置）；
  4. 首次部署自动生成管理员初始密码，并在 GitHub 运行结果页面的 **Summary** 中展示。

---

## 方式二：本地命令行手动部署

### 1. 前置环境
* **Node.js 20+**（`node -v` 查看）
* 运行目录进入本项目根目录

### 2. 登录与创建 KV
```bash
npx wrangler login
npx wrangler kv namespace create XS_SUB_KV
```
记下终端输出的 `id`（例如 `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`）。

### 3. 配置 wrangler.toml
根据 `wrangler.example.toml`，修改 `wrangler.toml` 中的以下项：
* `name`: Worker 名称（如 `cf-sub-worker`）
* `pattern`: 自定义域名（如 `sub.example.com`）
* `DOMAIN` / `FALLBACK_DOMAIN`: 你的域名
* 取消 `[[kv_namespaces]]` 注释，填入上一步创建的 KV ID

### 4. 设置敏感 Secrets
运行以下命令分别设置密钥（在交互式终端中输入，避免明文落盘）：
```bash
npx wrangler secret put ADMIN_PASSWORD_V9   # 管理员密码（至少12位）
npx wrangler secret put TOKEN_SECRET_V9     # 随机加密字符串（用于Token签名）
npx wrangler secret put VLESS_UUID_V9       # VLESS 节点的 UUID
```

### 5. 执行发布
```bash
npx wrangler deploy
```

---

## 常见问题与说明

1. **什么是 `PROXYIP`？**
   * `PROXYIP` 是 Worker 的备用反代出口。当 Worker 访问的目标也是托管在 Cloudflare 上的站点时，Cloudflare 官方会触发 Error 1000 自身回环拦截。此时 Worker 会通过 `PROXYIP` 转发。
   * 日常访问绝大多数主流网站均走原生直连，因此默认留空即可。
2. **首次登录后台**：
   * 浏览器访问 `https://<你的域名>/`，使用邮箱（`admin@<你的域名>` 或 `admin@example.com`）与初始密码登录。
   * 登录后可以在管理界面中下发客户端订阅 Token、查看网络大屏和优选节点状态。
