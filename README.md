# celld-lzcapp

[celld](https://github.com/denoland/celld) 的懒猫微服打包（**仅私有商店、镜像模式**）：自托管的分布式 Durable Objects —— 在自己机器上跑 Cloudflare Workers 应用（Workers、Durable Objects、KV、Queues、D1、R2、Workflows、Cron Triggers、静态资源）。

- **镜像**：`ghcr.io/denoland/celld`，经 `ghcr.1ms.run` 拉取（`delivery.mode: mirror`）。官方商店要求 `lazycat` 模式的懒猫镜像源，所以本包只发私有商店。
- **路由**：`/` → `celld:8080`（Worker 流量）。8081 是 operator/peer 监听，上游要求留在内网，这里不配置，让它沿用默认的回环临时端口。
- **配置全部走环境变量**（不用命令行参数，方便走安装向导）：
  - `CELLD_BUCKET`、`S3_ENDPOINT`、`AWS_REGION`、`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_SESSION_TOKEN`（celld 自带这些环境回退）
  - `CELLD_ADDR=0.0.0.0:8080`（不设只监听回环，微服入口进不来）
  - `CELLD_WATCH=/var/lib/celld/state`（本地 SQLite/复制工作目录）
  - `CELLD_TRUST_FORWARDED_HEADERS=true`（入口是反向代理）
- **持久化**：`/lzcapp/var/celld:/var/lib/celld`。长期状态（部署、cell 数据）在你自己填的 bucket 里。
- **镜像本身以 root 运行**（Dockerfile 没有 USER 指令），持久目录由平台以 root 创建，正好可写，因此 manifest 不设 `user`。
- **图标**：仓库里没有 logo，`icon.png` 是按「cell 网格」意象生成的 512×512 图。

## 已知限制

- **单节点**：上游说单个节点没有别的节点可以同步，每次写入都要等 bucket，比多节点 fleet 慢。多节点需要 8081 在节点间可达，当前打包方式做不到。
- **部署在别处做**：`celld deploy` 直接把部署写进 bucket（不在 HTTP 上），所以本应用没有部署接口暴露在外；用同一个 bucket 从你的电脑部署即可，节点会从 bucket 拿到部署。
- `public_path: [/]`：Worker 流量本来就是给外部访问的（webhook、分享链接）；8080 上没有运维面。若你只想让自己访问，把这段删掉即可走微服账号鉴权。
