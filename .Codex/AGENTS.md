# CF-worker-Auto-Keepalive-Pro 项目记忆

## 项目信息
- 项目名：CF-worker-Auto-Keepalive-Pro / CronHub
- 本地路径：`/Users/okekrr/Documents/0_项目/27_CF-worker-Auto-Keepalive-Pro`
- GitHub：`https://github.com/okekrr/CF-worker-Auto-Keepalive-Pro`
- 线上 Worker：`https://auto-keepalive-cronhub.okekrr.workers.dev`

## 项目定位
- 基于 `anlish01/Auto-Keepalive` 思路改造的 Cloudflare Worker 统一 Cron Hub。
- 目标：用 1 个 Cloudflare Cron Trigger 托管多个定时 HTTP 任务，节省免费账号 Cron 配额。

## 安全规则
- 不要把 ADMIN_PASS、目标任务 token、Authorization Header、x-admin-token 写入 GitHub、README 或 Obsidian。
- 任务配置中的敏感 Header 只允许写入 Cloudflare KV 或本机临时文件，不提交到仓库。

## 当前首个任务
- 美股盘后日报：调用 `https://us-market-daily.okekrr.workers.dev/api/generate?async=0`
- 方法：POST
- 认证：`x-admin-token` Header，值只存在 KV/临时文件，不进仓库。
- 调度：Asia/Shanghai 每周二到周六 06:40 左右执行一次，对应美股周一到周五盘后。
