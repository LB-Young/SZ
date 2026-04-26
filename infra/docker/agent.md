# 目录说明：`infra/docker`

## 本目录职责

- **Docker 相关**：`docker-compose.yml` 定义依赖服务、网络与卷；`Dockerfile` 可放在 `apps/api` 下构建镜像，本目录引用。

## 开发流程

- `docker compose up -d` 后本地 `api` 连 `localhost` 服务名。
