# 项目目录结构建议

> 与 `Architecture-Design.md` 对齐：三页前端、四 API、Session 编排、主/子 Agent、多轮召回、经验池、事件与 Session 冷存储。  
> 采用 **单仓 monorepo** 便于共享类型与契约；若团队更熟 **双仓**，可将 `apps/web` 与 `apps/api` 拆成两个 Git 仓库，子目录含义不变。

---

## 1. 设计原则

| 原则 | 说明 |
|------|------|
| **按边界分目录** | HTTP 入口、领域服务、Agent 编排、检索、规则、持久化分层，避免「一个大 `src` 全堆在一起」。 |
| **Agent 与 API 同进程起步** | MVP 将 BFF 与 Agent 放在 `apps/api`，后续再拆 `apps/worker` 仅当队列/算力需要。 |
| **Session 全量与向量数据不入库** | 放 `data/` 或配置的数据根目录，并由 `.gitignore` 忽略（见 §3）。 |
| **共享契约** | `packages/shared` 放 API 类型、评估结果 DTO、错误码；OpenAPI 可由后端生成后同步到前端。 |

---

## 2. 仓库根目录总览

```
shenzhi/
├── docs/                          # 需求、架构、本文件
├── apps/
│   ├── web/                       # 前端（输入 / 结果 / 历史）
│   └── api/                       # 后端：四 API + Session + Agent + 子 Agent
├── packages/
│   └── shared/                    # 可选：跨端类型、常量、校验 schema
├── data/                          # 本地开发：session 冷存储、可选本地向量/缓存（勿提交）
├── infra/                         # Docker Compose、部署片段、观测配置样例
├── scripts/                       # 种子数据、规则导入、一次性维护脚本
├── .env.example                   # 环境变量模板（勿放密钥）
├── .gitignore
├── package.json                   # 若用 pnpm/npm workspaces / turbo
├── pnpm-workspace.yaml            # 可选：monorepo 工作区声明
└── README.md                      # 如何启动 web + api、依赖服务
```

---

## 3. 前端 `apps/web`

以 **Next.js App Router** 为例；若用 **Vue 3 + Vite**，将 `app/` 换成 `src/views` + `src/router` 即可，逻辑对应关系不变。

```
apps/web/
├── public/
├── src/
│   ├── app/                       # 或 pages/（Pages Router）
│   │   ├── layout.tsx
│   │   ├── page.tsx               # 输入页（或重定向到 /assess）
│   │   ├── assess/                # 评估流：输入 + 同页或子路由结果区
│   │   │   └── page.tsx
│   │   ├── result/
│   │   │   └── [id]/page.tsx      # 结果详情（可选，与 assess 合并则简化）
│   │   └── history/
│   │       └── page.tsx
│   ├── components/
│   │   ├── assessment/            # 表单、流式输出、风险卡、引用列表
│   │   ├── layout/
│   │   └── ui/                    # 通用 UI
│   ├── lib/
│   │   ├── api-client.ts          # 四 API + SSE 封装
│   │   ├── analytics.ts           # 五事件埋点封装
│   │   └── utils.ts
│   └── hooks/
│       ├── use-assessment-stream.ts
│       └── use-session.ts
├── next.config.js                 # 或 vite.config.ts
├── package.json
└── tsconfig.json
```

**与需求映射**：`assess` + 可选 `result` 覆盖「输入页 + 结果页」；`history` 为历史记录页。若产品选择单页完成输入到结果，可只保留 `assess/page.tsx` 内嵌结果区。

---

## 4. 后端 `apps/api`（推荐布局，语言可二选一）

下列结构对 **Python（FastAPI）** 与 **Node（Hono/Fastify）** 均适用：将 `py` 文件换成 `ts` 即可，目录名保持一致以便团队统一心智。

```
apps/api/
├── src/
│   ├── main.py                    # 或 main.ts：应用入口、生命周期
│   ├── core/
│   │   ├── config.py              #  settings / pydantic BaseSettings
│   │   └── logging.py             # 结构化日志、trace_id
│   ├── api/                       # HTTP 层：仅参数解析、依赖注入、状态码
│   │   ├── deps.py
│   │   └── routes/
│   │       ├── assessments.py     # 提交评估（流式）、获取结果
│   │       ├── history.py         # 历史列表
│   │       └── contact.py         # 创建协同请求
│   ├── session/                   # Session 单写队列、FSM、与 session_id 绑定
│   │   ├── orchestrator.py
│   │   ├── fsm.py
│   │   ├── context_loader.py      # 热上下文 vs 压缩视图
│   │   └── storage.py             # 读写 data/sessions 全量 JSONL / 分文件
│   ├── agents/
│   │   ├── main/                  # 主 Agent：多轮召回 + LLM 循环
│   │   │   ├── graph.py           # 或 loop.py / LangGraph 定义
│   │   │   ├── prompts/
│   │   │   └── tools.py           # tool 注册：search_*、evaluate_rules
│   │   ├── subagents/
│   │   │   ├── persist.py         # 子 Agent：抽取落库
│   │   │   └── experience.py     # 子 Agent：经验 tmp 池写入
│   │   └── compression.py         # 上下文压缩任务（也可放 session/）
│   ├── retrieval/                 # 多轮召回实现细节
│   │   ├── experience.py          # tmp + 持久化池检索
│   │   ├── knowledge.py         # 知识库向量检索 + 重排
│   │   ├── merge.py               # 合并、去重、轮次元数据
│   │   └── recall_policy.py       # K_max、早停、覆盖度启发
│   ├── rules/                     # 风险规则层（可解释、可审计）
│   │   ├── engine.py
│   │   └── definitions/           # YAML/JSON 规则集，或按版本分子目录
│   ├── db/
│   │   ├── models.py              # ORM 模型
│   │   ├── session_factory.py
│   │   └── repositories/          # 按聚合根分文件（可选）
│   ├── migrations/                # Alembic；Node 可用 drizzle/knex
│   ├── observability/
│   │   ├── events.py              # 五事件 + 扩展 recall 事件
│   │   └── tracing.py
│   └── jobs/                      # 可选：后台队列消费者
│       └── queue_worker.py
├── tests/
│   ├── unit/
│   ├── integration/
│   └── contract/
├── pyproject.toml                 # 或 package.json + tsconfig
├── Dockerfile
└── .env.example
```

**要点**：

- **`api/routes`**：只保留四组路由对应四个对外能力；流式从 `session` 或 `agents/main` 拉取 async generator。  
- **`retrieval`**：与架构文档 §6「多轮经验/知识召回」一一对应，避免把检索细节塞满 `agents/main`。  
- **`rules`**：与 LLM 解耦，便于单测与监管解释。  
- **`session/storage`**：`data/sessions/` 的实际路径由 `core/config` 注入，**全量对话**落盘规则见架构 §9。

---

## 5. 共享包 `packages/shared`（可选）

```
packages/shared/
├── src/
│   ├── types/
│   │   ├── assessment.ts          # 四字段 + rule_source + generated_at
│   │   └── events.ts              # 五事件 payload 形状
│   ├── constants/
│   │   └── risk-levels.ts
│   └── schemas/                   # JSON Schema 或 zod，与前后端共用
│       └── assessment-result.json
└── package.json
```

若全栈 Python（如 Streamlit 仅内部用），可改为 `packages/shared_py` 仅放 Pydantic 模型，无前端包。

---

## 6. 数据与基础设施

### 6.1 `data/`（开发机，不提交）

```
data/
├── sessions/                      # session 全量、压缩快照（架构 §9）
│   └── {session_id}/
│       ├── messages.jsonl
│       ├── tool_calls.jsonl
│       └── snapshots/
└── dev/                           # 本地向量库文件、sqlite 试验（按需）
```

### 6.2 `infra/`

```
infra/
├── docker/
│   ├── docker-compose.yml           # postgres、可选 qdrant/redis/minio
│   └── Dockerfile.api
└── otel/
    └── otel-collector.example.yaml
```

---

## 7. `scripts/`

```
scripts/
├── seed_rules.py                  # 导入初始规则到 DB 或 definitions/
├── import_knowledge.py            # 知识库切分与灌库（离线）
└── smoke_api.sh                   # 健康检查与四 API 冒烟
```

---

## 8. 根目录 `.gitignore` 建议片段

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

## 9. 与架构模块速查

| 架构模块 | 目录落点 |
|----------|----------|
| 三页 | `apps/web/src/app/...` |
| 四 API | `apps/api/src/api/routes/` |
| Session 单写 + FSM | `apps/api/src/session/` |
| 主 Agent | `apps/api/src/agents/main/` |
| 子 Agent 落库 / 经验 | `apps/api/src/agents/subagents/` |
| 多轮召回 | `apps/api/src/retrieval/` |
| 规则引擎 | `apps/api/src/rules/` |
| 五事件 | `apps/api/src/observability/events.py` + 前端 `lib/analytics.ts` |
| 关系型业务表 | `apps/api/src/db/` + `migrations/` |
| Session 冷存储 | `data/sessions/` + `session/storage.py` |

---

*实现时若精简为「单应用无 monorepo」，可将 `apps/web` 与 `apps/api` 压成同一仓库根下 `frontend/` 与 `backend/`，上表路径做一级平移即可。*
