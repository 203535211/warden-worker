# Warden：一个与 Bitwarden 兼容的 Cloudflare Workers 服务器

[![Powered by Cloudflare](https://img.shields.io/badge/Powered%20by-Cloudflare-F38020?logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Deploy to Cloudflare Workers](https://img.shields.io/badge/Deploy%20to-Cloudflare%20Workers-orange?logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/)

本项目提供了一个可自托管的、兼容 Bitwarden 的服务器，可以免费部署到 Cloudflare Workers。它旨在实现低维护成本，让您“部署后即可高枕无忧”，无需担心服务器管理或后续费用。

## 为什么还要建一个 Bitwarden 服务器？

虽然像 Vaultwarden 这样的项目提供了优秀的自托管解决方案，但它们仍然需要您管理服务器或 VPS。这可能会很麻烦，而且如果您忘记支付服务器费用，您可能会丢失密码访问权限。

Warden 旨在利用 Cloudflare Workers 生态系统解决这个问题。通过将 Warden 部署到 Cloudflare Worker 并使用 Cloudflare D1 进行存储，您可以拥有一个完全免费、无服务器且维护成本低的 Bitwarden 服务器。

## 特征

* 核心 Vault 功能：创建、读取、更新和删除密码和文件夹。
* 文件附件：可选的 Cloudflare KV 或 R2 存储用于存放附件。
* TOTP 支持：存储和生成基于时间的一次性密码。
* 兼容 Bitwarden：可与官方 Bitwarden 客户端配合使用。
* 免费托管：使用 Cloudflare 的免费套餐。
* 维护成本低：部署一次即可，无需后续操作。
* 安全：您的加密数据存储在您的 Cloudflare D1 数据库中。
* 易于部署：使用 Wrangler CLI，几分钟即可启动并运行。

### 附件支持

Warden 支持使用 **Cloudflare KV** 或 **Cloudflare R2** 作为存储后端来保存文件附件：

|特性|KV|R2|
|--|--|--|
|单个文件最大大小|**25 MB**（硬性限制）|100 MB（受 Workers 请求体大小限制）|
|是否需要信用卡|**否**|是|
|流式 I/O|是|是|

**后端选择规则：** R2 优先——如果配置了 R2，则使用 R2；否则使用 KV。

配置细节请参见[部署指南](docs/deployment.md)。R2 可能会产生额外费用；参见 [Cloudflare R2 定价](https://developers.cloudflare.com/r2/pricing/)。

## 当前状态

**本项目尚未实现全部功能**，~~也许永远不会~~。目前支持个人金库（Personal Vault）的核心功能，包括 TOTP。但它 **不支持** 以下功能：

* 共享（Sharing）
* 2FA 登录（除 TOTP 外）
* Bitwarden Send
* 设备与会话管理
* 紧急访问（Emergency access）
* 管理员操作
* 组织（Organizations）
* 其他 Bitwarden 高级功能

目前没有立即实现这些功能的计划。本项目的主要目标是提供一个简单、免费、低维护成本的个人密码管理器。

## 兼容性

* **浏览器扩展：** Chrome、Firefox、Safari 等（在 Chrome 上测试：2025.11.1）
* **Android App：** 官方 Bitwarden Android 应用（测试版本：2025.11.0）
* **iOS App：** 官方 Bitwarden iOS 应用（测试版本：2025.11.0）

## 演示（Demo）

演示实例地址：[warden.qqnt.de](https://warden.qqnt.de)。

您可以使用以 `@warden-worker.demo` 结尾的邮箱注册新账号（邮箱无需验证）。

如果您决定停止使用演示实例，请删除您的账号，为其他人腾出空间。

强烈建议您部署自己的实例，因为演示服务可能会触发速率限制并被 Cloudflare 禁用。

## 快速开始

* 选择部署方式：[CLI 部署](docs/deployment.md#cli-deployment) 或 [GitHub Actions 部署](docs/deployment.md#cicd-deployment-with-github-actions)。
* 按部署文档设置密钥（secrets）以及可选的附件存储。
* 将 Bitwarden 客户端配置为指向您的 Worker URL。

## 前端（Web Vault）

前端通过 [Cloudflare Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/) 与 Worker 一起打包发布。GitHub Actions 工作流会下载一个 **固定版本（pinned）** 的 [bw_web_builds](https://github.com/dani-garcia/bw_web_builds)（Vaultwarden 的 web vault）发布包（默认：`v2025.12.0`），并与后端一起部署。

您可以通过 GitHub Actions Variables 覆盖该版本（生产环境用 `BW_WEB_VERSION`，开发环境用 `BW_WEB_VERSION_DEV`），或者设置为 `latest` 来跟随上游。

**工作原理：**

* 静态文件（HTML、CSS、JS）由 Cloudflare 的边缘网络直接提供。
* API 请求（`/api/*`、`/identity/*`）会路由到 Rust Worker。
* 无需单独部署 Pages，也不需要额外的域名配置。

**UI 覆盖（可选）：**

* 本项目在 `public/css/` 中提供了一小组“轻量自托管”风格的 UI 微调。
* 在 CI/CD（以及可选的本地环境）中，我们会在解压 `bw_web_builds` 后应用它们：
  * `bash scripts/apply-web-vault-overrides.sh public/web-vault`

 > 
 > \[!NOTE\]
 > 从“前后端分离部署”迁移？如果您之前将前端单独部署到 Cloudflare Pages，可以删除 `warden-frontend` 这个 Pages 项目，并重新为 worker 配置路由。现在前端已与 Worker 绑定打包，不再需要单独部署。

 > 
 > \[!WARNING\]
 > Web Vault 前端来自 Vaultwarden，因此 UI 上会显示许多高级功能入口，但其中大部分实际上不可用。请参见[当前状态](#当前状态)。

## 配置自定义域名（可选）

默认的 `*.workers.dev` 域名默认是关闭的，因为它可能会抛出 1101 错误。您可以在 `wrangler.toml` 中设置 `workers_dev = true` 来启用。

如果您想使用自定义域名而不是默认的 `*.workers.dev` 域名，请按以下步骤操作：

### 第 1 步：添加 DNS 记录

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
1. 选择您的域名（例如 `example.com`）
1. 进入 **DNS** → **Records**
1. 点击 **Add record**：
   * **Type：** `A`（或 IPv6 用 `AAAA`）
   * **Name：** 您的子域名（例如 `vault` 对应 `vault.example.com`）
   * **IPv4 address：** `192.0.2.1`（占位地址，实际路由由 Worker 处理）
   * **Proxy status：** **Proxied**（橙色云朵图标——*必须开启*）
   * **TTL：** Auto
1. 点击 **Save**

 > 
 > \[!IMPORTANT\]
 > **Proxy status 必须是 “Proxied”**（橙色云朵）。如果显示 “DNS only”（灰色云朵），Worker 路由将无法工作。

### 第 2 步：添加 Worker Route

1. 进入 **Workers & Pages** → 选择您的 `warden-worker`
1. 点击 **Settings** → **Domains & Routes**
1. 点击 **Add** → **Route**
1. 配置路由：
   * **Route：** `vault.example.com/*`（替换为您的域名）
   * **Zone：** 选择您的域名 Zone
   * **Worker：** `warden-worker`
1. 点击 **Add route**

## 内置限速（Rate Limiting）

本项目包含基于 [Cloudflare Rate Limiting API](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/) 的限速功能。已对敏感端点进行保护：

|端点|限速|Key 类型|目的|
|--|--|------|--|
|`/identity/connect/token`|5 次/分钟|邮箱地址|防止密码暴力破解|
|`/api/accounts/register`|5 次/分钟|IP 地址|防止批量注册与邮箱枚举|
|`/api/accounts/prelogin`|5 次/分钟|IP 地址|防止邮箱枚举|

您可以在 `wrangler.toml` 中调整限速设置：

````toml
[[ratelimits]]
name = "LOGIN_RATE_LIMITER"
namespace_id = "1001"
# Adjust limit (requests) and period (10 or 60 seconds)
simple = { limit = 5, period = 60 }
````

 > 
 > \[!NOTE\]
 > `period` 只能是 `10` 或 `60` 秒。详情参见 [Cloudflare 文档](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/)。

如果缺少该 binding，请求将不进行限速（优雅降级）。

## 配置

### Durable Objects（CPU 卸载）

Cloudflare Workers Free 计划的单次请求 CPU 配额非常小。有两类端点特别“吃 CPU”：

* 导入端点：较大的 JSON 载荷（通常 500kB–1MB）+ 解析 + 批量写入。
* 注册、登录与密码校验端点：服务端使用 PBKDF2 进行密码校验。

为保持主 Worker 响应快速，同时仍然支持这些操作，Warden 可以将指定端点 **卸载到 Durable Objects（DO）**：

* **Heavy DO（`HEAVY_DO`）**：以 Rust 实现为 `HeavyDo`（复用现有 axum 路由），使 CPU 重端点可以在更高的 CPU 配额下运行。

**如何启用/禁用**

是否进行 CPU 卸载取决于 `wrangler.toml` 中是否配置了 `HEAVY_DO` Durable Object binding。

 > 
 > \[!NOTE\]
 > Durable Objects 在免费计划中每次请求有更高的 CPU 配额（30 秒，参见 [Cloudflare Durable Objects 限制](https://developers.cloudflare.com/durable-objects/platform/limits/)），因此可用于卸载 CPU 重端点。
 > 
 > Durable Objects 可能产生两类费用：计算与存储。本项目不使用存储；免费计划每天允许 100,000 次请求与 13,000 GB-s 的计算时长，通常足够大多数用户使用。详情参见 [Cloudflare Durable Objects 定价](https://developers.cloudflare.com/durable-objects/platform/pricing/)。
 > 
 > 如果您选择禁用 Durable Objects，为避免被 Cloudflare 限流，您可能需要订阅付费计划。

### 环境变量

在 `wrangler.toml` 的 `[vars]` 下配置环境变量，或通过 Cloudflare Dashboard 设置：

* **`PASSWORD_ITERATIONS`**（可选，默认：`600000`）：
  * 服务端密码哈希的 PBKDF2 迭代次数。
  * 最小值为 600000。
* **`TRASH_AUTO_DELETE_DAYS`**（可选，默认：`30`）：
  * 软删除条目在清理前保留的天数。
  * 设为 `0` 或负数以禁用。
* **`IMPORT_BATCH_SIZE`**（可选，默认：`30`）：
  * 导入/删除操作的批处理大小。
  * 设为 `0` 以禁用批处理。
* **`DISABLE_USER_REGISTRATION`**（可选，默认：`true`）：
  * 控制客户端 UI 是否显示注册按钮（服务器行为不变）。
* **`AUTHENTICATOR_DISABLE_TIME_DRIFT`**（可选，默认：`false`）：
  * 设为 `true` 以禁用 TOTP 校验的 ±1 时间步漂移。
* **`ATTACHMENT_MAX_BYTES`**（可选）：
  * 单个附件文件最大大小。
  * 例如：`104857600` 表示 100MB。
* **`ATTACHMENT_TOTAL_LIMIT_KB`**（可选）：
  * 每个用户附件总存储上限（KB）。
  * 例如：`1048576` 表示 1GB。
* **`ATTACHMENT_TTL_SECS`**（可选，默认：`300`，最小：`60`）：
  * 附件上传/下载 URL 的 TTL。

### 定时任务（Cron）

Worker 会运行一个定时任务来清理软删除的条目。默认每天 03:00 UTC 执行（`wrangler.toml` 的 `[triggers]` 中 cron 为 `"0 3 * * *"`）。可按需调整；cron 表达式语法请参见 [Cloudflare Cron Triggers 文档](https://developers.cloudflare.com/workers/configuration/cron-triggers/)。

## 数据库操作

* **备份与恢复：** 自动备份与手动恢复步骤请参见 [Database Backup & Restore](docs/db-backup-recovery.md#github-actions-backups)。
* **时间旅行（Time Travel）：** 参见 [D1 Time Travel](docs/db-backup-recovery.md#d1-time-travel-point-in-time-recovery) 以恢复到某个时间点。
* **全局等价域名（Global Equivalent Domains）填充（可选）：** 参见 [docs/deployment.md](docs/deployment.md) 中 CLI 部署与 CI/CD 的相关说明。
* **使用 D1 进行本地开发：**
  * 快速启动：`wrangler dev --persist`
  * 完整栈（含 Web Vault）：按部署文档下载前端资源，然后 `wrangler dev --persist`
  * 在本地导入备份：`wrangler d1 execute vault1 --file=backup.sql`
  * 查看本地 DB：SQLite 文件位于 `.wrangler/state/v3/d1/`

## 使用 D1 的本地开发

使用 Wrangler 在本地运行带 D1 支持的 Worker。

**快速启动（仅 API）：**

````bash
wrangler dev --persist
````

**完整栈（包含 Web Vault）：**

1. 下载前端资源（参见[部署文档](docs/deployment.md#download-the-frontend-web-vault)）。

1. 本地启动：
   
   ````bash
   wrangler dev --persist
   ````

1. 在 `http://localhost:8787` 访问 vault。

**临时使用生产数据：**

1. 下载并解密备份（参见[备份文档](docs/db-backup-recovery.md#restoring-database-to-cloudflare-d1)）。

1. 不使用 `--remote` 在本地导入：
   
   ````bash
   wrangler d1 execute vault1 --file=backup.sql
   ````

1. 启动 `wrangler dev --persist`，并将客户端指向 `http://localhost:8787`。

**查看本地 SQLite：**

````bash
ls .wrangler/state/v3/d1/
sqlite3 .wrangler/state/v3/d1/miniflare-D1DatabaseObject/*.sqlite
````

 > 
 > \[!NOTE\]
 > 本地开发需要 Node.js 与 Wrangler。Worker 会在模拟环境中运行，底层为 [workerd](https://github.com/cloudflare/workerd)。

## 贡献

欢迎提交 Issue 与 PR。提交前请运行 `cargo fmt` 和 `cargo clippy --target wasm32-unknown-unknown --no-deps`。

## 许可证

本项目基于 MIT License 许可。详见 `LICENSE` 文件。
