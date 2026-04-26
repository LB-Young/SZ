# 目录说明：`apps/web/src/lib`

## 本目录职责

- **无 UI 工具**：`api-client`（四 API + `SSE`）、`analytics`（五事件与可选扩展）、`utils`；可引用 `packages/shared` 的 DTO/校验。

## 安全

- 环境变量中读 API 基址；不在前端存密钥用于调用需要特权的服务器能力。
