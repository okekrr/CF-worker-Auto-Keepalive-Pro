# CF-worker-Auto-Keepalive-Pro / CronHub

基于 `anlish01/Auto-Keepalive` 改造的 Cloudflare Worker 统一 Cron 托管项目。核心目标是：用 **1 个 Cloudflare Cron Trigger** 轮询/触发多个 HTTP 定时任务，节省免费账号 Cron 配额。

## 当前部署

- Worker 名称：`auto-keepalive-cronhub`
- 线上地址：`https://auto-keepalive-cronhub.okekrr.workers.dev`
- Cron：`*/5 * * * *`
- 存储：Cloudflare KV，binding 为 `DB`

## 已改造能力

- 保留原版可视化面板、KV 配置、运行日志、通知渠道。
- 支持任务级 `method`，例如 `POST`。
- 支持任务级 `headers`，用于调用受保护的 Worker endpoint。
- 支持 `runAt + timeZone + weekdays + windowMinutes` 的固定时间窗口调度。
- 支持每天只执行一次的 `lastRunKey` 防重复逻辑。
- 保留原版 `interval` 间隔保活能力。

## 当前接入任务

- 美股盘后日报：每周二到周六 `Asia/Shanghai 06:40` 左右触发一次。
- 目标：`POST https://us-market-daily.okekrr.workers.dev/api/generate?async=0`
- 鉴权 Header 存在 Cloudflare KV 的任务配置中，不写入仓库。

## 本地使用

```bash
npm install
npm run deploy:dry
npm run deploy
```

首次部署需要创建 KV 并把 namespace id 写入 `wrangler.jsonc`：

```bash
npx wrangler kv namespace create DB
```

后台登录密码使用 Cloudflare Secret：

```bash
npx wrangler secret put ADMIN_PASS
```

## 安全

仓库不保存任何密钥；`ADMIN_PASS`、目标任务 token、`Authorization`、`x-admin-token` 等只放 Cloudflare Secret/KV 或本机临时文件。
