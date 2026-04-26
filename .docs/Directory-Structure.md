# 项目目录结构建议

> 与 `Architecture-Design.md` **v1.2+** 对齐：四 API、Session 编排与 **FSM**、主/子 Agent、多轮召回、**四类数据落点**（**关系库 D**、**经验 JSON 池 E**、**知识向量 K**、**本地对话 FS 三层**）、异步落库与 **assessment_closed** 后任务。  
> 单仓 **monorepo**；仓库根目录名可为 `SZ` / `shenzhi` 等，下表以 `repo/` 代指根。

---

## 1. 设计原则

| 原则 | 说明 |
|------|------|
| **按边界分目录** | HTTP、编排、Agent、检索、规则、**文件型存储适配器**、关系型 DB 分层。 |
| **介质与代码同构** | **FS / 经验 JSON** 的读写集中在 `storage/`，**向量检索**在 `retrieval/`，**SQL** 在 `db/`，避免混在 Agent 内硬编码路径。 |
| **Agent 与 BFF 同进程起步** | MVP：`apps/api`；若拆队列进程，将 `jobs/` 消费者迁到 `apps/worker` 即可。 |
| **`data/` 不提交** | Session 全量、经验 JSON、本地向量试验等进 `data/`，由 `.gitignore` 忽略。 |
| **共享契约** | `packages/shared`：评估结果 DTO、错误码、五事件 payload、JSON Schema。 |

---

## 2. 仓库根目录总览

```
repo/
├── docs/                          # Feature、Architecture、本文件
├── apps/
│   ├── web/                       # 三页：输入 / 结果 / 历史
│   └── api/                       # 四 API + Session + Agent + storage 适配器
├── packages/
│   └── shared/                    # 跨端类型、常量、zod/json schema
├── data/                          # 本地运行数据根（勿提交）
│   ├── sessions/                  # FS：按 session_id 三层文件（见 §3）
│   └── experience/              # E：经验池 JSON，tmp 与 persistent 分区
├── infra/                         # Compose、镜像、观测样例
├── scripts/                       # 规则导入、知识灌库、冒烟
├── .env.example
├── .gitignore
├── package.json                   # 若用 workspaces
├── pnpm-workspace.yaml            # 可选
└── README.md
```

---

## 3. 前端 `apps/web`

与 **§2.1** 三页对应；技术栈以 Next App Router 为例。

```
apps/web/
├── public/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx               # 入口或重定向到 /assess
│   │   ├── assess/                # 输入 + 流式结果区
│   │   ├── result/[id]/           # 结果详情（可与 assess 合并则删减）
│   │   └── history/
│   ├── components/
│   │   ├── assessment/            # 表单、流式、风险卡、引用列表
│   │   ├── layout/
│   │   └── ui/
│   ├── lib/
│   │   ├── api-client.ts          # 四 API + SSE
│   │   └── analytics.ts           # 五事件埋点
│   └── hooks/
│       ├── use-assessment-stream.ts
│       └── use-session.ts
├── next.config.js
├── package.json
└── tsconfig.json
```

---

## 4. 后端 `apps/api`

目录名对 **Node / Python** 通用；实现文件扩展名按语言替换。

```
apps/api/
├── src/
│   ├── main.ts                    # 或 main.py：入口、生命周期、挂载路由
│   ├── core/
│   │   ├── config.ts              # 数据根路径、向量服务 URL、重试 N_max 等
│   │   └── logging.ts             # 结构化日志、trace_id = session/query
│   ├── api/
│   │   ├── deps.ts
│   │   └── routes/
│   │       ├── assessments.ts     # 提交评估（SSE）、获取结果
│   │       ├── history.ts
│   │       └── contact.ts         # 协同请求
│   ├── session/                   # 控制面：单写、FSM、CTX 装配（§3 SessionOrch）
│   │   ├── orchestrator.ts        # 入队、与 storage 协作写 FS
│   │   ├── fsm.ts                 # Idle → … → Submitted → ResultReady / Failed
│   │   ├── errors.ts              # 可重试 vs 不可重试映射（与 §5 一致）
│   │   ├── context-loader.ts      # 热上下文 vs 压缩视图
│   │   └── types.ts
│   ├── storage/                   # 数据面：文件与路径约定（§4 D/E/K/FS 中的 FS+E）
│   │   ├── session-files.ts       # FS 三层：01_raw / 02_snapshots / manifest
│   │   ├── experience-files.ts    # E：tmp / persistent JSON 读写与晋升路径
│   │   └── paths.ts               # 统一拼 dataRoot/sessions|experience
│   ├── agents/
│   │   ├── main/                  # 主 Agent：多轮召回 + tool + LLM
│   │   │   ├── graph.ts           # 或 loop：感知-召回-决策-生成
│   │   │   ├── prompts/
│   │   │   └── tools.ts           # search_experience / search_knowledge / evaluate_rules
│   │   ├── subagents/
│   │   │   ├── persist.ts         # 子 Agent：结构化落库 → D
│   │   │   └── experience.ts      # 子 Agent：对话结束后写 E（tmp）与晋升
│   │   └── compression.ts         # 上下文压缩任务（亦可挂 session/）
│   ├── retrieval/                 # 读 K + 读 E（经 storage 或直接向量化服务）
│   │   ├── experience.ts          # 对 JSON 池的检索与可选侧路索引
│   │   ├── knowledge.ts           # 向量检索 + 重排
│   │   ├── merge.ts
│   │   └── recall-policy.ts       # K_max、早停、覆盖度
│   ├── rules/
│   │   ├── engine.ts
│   │   └── definitions/           # YAML/JSON 规则版本
│   ├── db/                        # 关系库 D：ORM、仓储
│   │   ├── models.ts
│   │   ├── session-factory.ts
│   │   └── repositories/
│   ├── observability/
│   │   ├── events.ts              # 五事件 + 可选 assessment_failed
│   │   └── tracing.ts
│   ├── jobs/                      # 异步：子 Agent、关闭后队列、死信
│   │   ├── enqueue.ts
│   │   ├── handlers/
│   │   │   ├── persist-assessment.ts
│   │   │   ├── post-close-analyze.ts   # assessment_closed：读 FS、写 D/E
│   │   │   └── experience-promote.ts  # 可选：晋升与过期扫描
│   │   └── worker.ts              # 单机队列或 Redis Stream 消费者
│   └── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── contract/
├── Dockerfile
├── package.json                   # 或 pyproject.toml
└── .env.example
```

**要点**：

- **`api/routes`**：仅四组对外能力；SSE 从 `session/orchestrator` 拉取与 **主 Agent** 绑定的 async 流。  
- **`storage`**：实现架构 **§4** 中 **FS** 与 **E** 的落盘，**不**把大段对话塞进 PostgreSQL CLOB。  
- **`retrieval/knowledge`**：对接 **K**（pgvector / Qdrant 等），与 **E** 文件池解耦。  
- **`db`**：仅 **D**（assessment、advice、evidence、rule_hits、event_logs、contact_requests）；经验正文以 **文件为准**，库内可有可选轻量索引表（见架构 §11）。  
- **`jobs/handlers/post-close-analyze`**：`assessment_closed` 后读 **FS**、写 **D/E**，与「每轮后 persist」并存时用幂等键。  
- **`session/errors`**：主链路异常分类，驱动 **同态重试** 或 **Failed** 响应（架构 §5）。

---

## 5. `packages/shared`

```
packages/shared/
├── src/
│   ├── types/
│   │   ├── assessment.ts          # 四字段 + rule_source + generated_at + 引用 id
│   │   └── events.ts
│   ├── constants/
│   │   └── risk-levels.ts
│   └── schemas/
│       └── assessment-result.json
└── package.json
```

---

## 6. `data/`（开发机，勿提交）

### 6.1 Session **FS** 三层（与架构 §9 / §4 一致）

```
data/sessions/
└── {session_id}/
    ├── 01_raw/                    # 第一层：全量不可变追加
    │   ├── messages.jsonl
    │   └── tool_calls.jsonl
    ├── 02_snapshots/              # 第二层：压缩版本 v{n}.json
    │   └── v000001.json
    └── 03_manifest.json           # 第三层：指针、compress_versions、热加载策略
```

### 6.2 经验池 **E**（JSON 文件）

```
data/experience/
├── tmp/                           # 可冗余、可过期；单条一条 JSON 或 JSONL
└── persistent/                    # 晋升后只读优先检索分区
```

### 6.3 其他

```
data/dev/                          # 本地试验：sqlite、向量导出缓存等（按需）
```

**知识库 K** 通常由 **Docker 内 Postgres+pgvector** 或 **Qdrant** 承载，路径在 `infra/docker/`；一般**不**把向量主存放到 `data/` 除非做离线 dump。

---

## 7. `infra/` 与 `scripts/`

```
infra/docker/
├── docker-compose.yml             # postgres、可选 qdrant/redis
└── Dockerfile.api
infra/otel/
    └── otel-collector.example.yaml
```

```
scripts/
├── seed-rules.ts                  # 规则导入 definitions + DB
├── import-knowledge.ts            # 切分文档、写向量库 K
└── smoke-api.sh
```

---

## 8. `.gitignore` 建议

```
data/
.env
.env.local
**/__pycache__/
**/node_modules/
**/.next/
**/dist/
*.sqlite
```

---

## 9. 架构模块速查（含数据面）

| 架构模块 | 目录落点 |
|----------|----------|
| 三页 | `apps/web/src/app/...` |
| 四 API | `apps/api/src/api/routes/` |
| Session 单写 + FSM + 错误策略 | `apps/api/src/session/` |
| 主 Agent | `apps/api/src/agents/main/` |
| 子 Agent 落库 / 经验 | `apps/api/src/agents/subagents/` + `jobs/handlers/` |
| 多轮召回 | `apps/api/src/retrieval/` |
| 规则引擎 | `apps/api/src/rules/` |
| **FS + 经验 JSON 文件** | `apps/api/src/storage/` + `data/sessions/` + `data/experience/` |
| **关系库 D** | `apps/api/src/db/` + `migrations/` |
| **知识向量 K** | `retrieval/knowledge.ts` + `infra/docker` 内向量服务 |
| 五事件 | `observability/events.ts` + 前端 `lib/analytics.ts` |
| 异步与关闭后任务 | `apps/api/src/jobs/` |

---

*若精简为单应用：将 `apps/web` 与 `apps/api` 改为仓库根下 `frontend/` 与 `backend/` 即可，子目录名保持不变便于对照架构文档。*
