# 现有 Cloudflare Cron 迁移评估

更新时间：2026-07-06

## 当前结论

可以继续把现有 Cron 合并进 `auto-keepalive-cronhub`，但不要直接用后台登录 Cookie 去调用现有管理接口。推荐给每个目标 Worker 增加一个受保护的内部 HTTP endpoint，例如：

- `POST /api/cron/backup`
- Header：`x-cronhub-token: <secret>`
- endpoint 内部复用原 `scheduled()` 的核心函数

这样迁移后，目标 Worker 删除 `triggers.crons`，统一由 CronHub 发 HTTP 请求触发。

## 迁移可行性表

| Worker | 当前 Cron | 当前逻辑 | 可行性 | 建议做法 |
|---|---:|---|---|---|
| `us-market-daily` | 已移除 | `POST /api/generate?async=0` 已有受保护入口 | 已完成 | 已由 CronHub 每周二到周六 `Asia/Shanghai 06:40` 调用 |
| `sub-proxy-manager` | `30 3 * * *` | `scheduled()` 调用 `runKVBackupToGitHub(env, 'scheduled')` | 高 | 新增 `POST /api/cron/backup`，用独立 secret 校验后调用同一备份函数；CronHub 按 `Asia/Shanghai 11:30` 每天触发；验证后删除该 Worker Cron |
| `nodewarden` | `*/5 * * * *` | `scheduled()` 调用 `runScheduledBackupIfDue(env)` | 中高 | 新增 `POST /api/cron/scheduled-backup`，用独立 secret 校验后调用 `runScheduledBackupIfDue(env)`；CronHub 每 5 分钟调用；验证后删除该 Worker Cron |
| `cloud-mail` | `0 16 * * *`、`*/30 * * * *` | 线上版本有 `fetch`、`email`、`scheduled` handler，来源是 Dashboard | 中 | 先拿到/整理源码，确认两个 Cron 在 `scheduled()` 中分别做什么；再新增 1-2 个受保护 HTTP endpoint；最后迁移到 CronHub。另：检查到有敏感配置以普通变量形式存在，建议迁到 Worker Secret |

## 预期节省

- 当前：`5/5` 个 Cloudflare Cron 已用满。
- 迁移 `sub-proxy-manager` + `nodewarden` 后：预计降到 `3/5`。
- 再迁移 `cloud-mail` 两个 Cron 后：预计降到 `1/5`，只剩 CronHub 自己的 `*/5 * * * *`。

## 推荐迁移顺序

1. `sub-proxy-manager`：逻辑最单一，只是每日 KV 备份，风险最低。
2. `nodewarden`：高频 Cron，释放价值最大，但要确认备份逻辑的幂等与频率判断。
3. `cloud-mail`：先整理源码和敏感变量，再动生产。

## 不建议的做法

- 不建议让 CronHub 模拟登录后台再带 Cookie 调用管理接口：会受会话过期、密码变更、风控影响，不稳定。
- 不建议把目标 Worker 的管理 token 写进 GitHub、README 或 Obsidian。
- 不建议迁移前直接删除目标 Worker 的 Cron；必须先线上测试 HTTP endpoint 成功。
