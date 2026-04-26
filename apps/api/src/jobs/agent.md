# 目录说明：`apps/api/src/jobs`

## 本目录职责

- **后台任务入口**：子 Agent 消费、重试、定时清理 tmp 经验、可选导出审计包；与 `main` 进程可同进程或独立 worker 进程。

## 队列

- MVP 可用内存队列；多实例时换 Redis Stream 等，接口抽象在本目录内。
