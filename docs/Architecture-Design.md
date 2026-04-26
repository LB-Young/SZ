## 架构图
**Web 前端 ↔ API 网关/应用服务 ↔ 会话与编排器 ↔ 主 Agent + 子 Agent ↔ 数据与可观测**。

```mermaid
graph TB
  subgraph Client["客户端"]
    UI_Input["输入页"]
    UI_Result["结果页"]
    UI_History["历史页"]
  end

  subgraph BFF["后端应用层 BFF API"]
    API_Submit["提交评估 流式"]
    API_GetResult["获取结果"]
    API_History["获取历史"]
    API_Contact["创建协同请求"]
  end

  subgraph SessionOrch["会话与编排 Session Orchestrator"]
    SM["Session 单写入口与并发控制"]
    ST["Query 状态机 FSM"]
    CTX["上下文装配 原始对话与压缩视图"]
  end

  subgraph Agents["智能体层"]
    MA["主Agent 多轮召回 规则与工具 LLM"]
    SA_Persist["子 Agent: 结构化落库 抽取 assessment 等"]
    SA_Exp["子 Agent: 经验提取与入 tmp 池"]
  end

  subgraph Data["数据与知识"]
    DB[(关系库PostgreSQL业务表)]
    KB["知识库 向量+元数据"]
    ExpTmp["经验 tmp 池"]
    ExpPer["经验持久化池"]
    SessStore["Session 全量与压缩快照 本地/对象存储"]
  end

  subgraph Obs["可观测性"]
    Metrics["指标"]
    Tracing["链路追踪"]
    LogAgg["事件与审计日志"]
  end

  UI_Input --> API_Submit
  UI_Result --> API_GetResult
  UI_History --> API_History
  UI_Result --> API_Contact

  API_Submit --> SM
  SM --> ST
  SM --> CTX
  CTX --> MA
  MA --> KB
  MA --> DB
  MA --> ExpTmp
  MA --> ExpPer

  MA -->|一轮结束| SA_Persist
  MA -->|一轮结束| SA_Exp
  SA_Persist --> DB
  SA_Exp --> ExpTmp
  ST --> LogAgg
  API_Submit --> LogAgg
```


## 请求与数据流（端到端）

本节按 **控制面**（请求/编排/状态）与 **数据面**（各类数据最终落在哪种介质）描述一条评估请求的端到端路径。介质分工与 §3 中 **DB / KB / Exp* / SessStore** 一致：**关系型业务 → 数据库 `D`**，**经验条目 → JSON 文件池 `E`**（tmp + 持久分区），**知识 chunk → 向量库 `K`**，**完整对话与 tool 轨迹 → 本地文件 `FS`**（实现上可对应 §9 全量/快照/manifest 三层，此处序列图统称 `FS`）。


```mermaid
sequenceDiagram
  participant UI as Web 前端
  participant API as BFF API
  participant SM as Session 单写入口
  participant FSM as Query 状态机
  participant CTX as 上下文装配
  participant MA as 主 Agent
  participant KB as 知识库 K
  participant DB as 数据库 D
  participant FS as 会话文件 FS
  participant TMP as 经验 tmp 池 E/tmp
  participant PER as 经验持久池 E/per
  participant SP as 结构化落库子 Agent
  participant SE as 经验提取子 Agent
  participant OBS as 可观测性

  UI->>API: 提交评估请求
  API->>SM: 创建或恢复 Session
  SM->>FSM: 进入 running 状态
  SM->>FS: 写入原始输入与请求元数据
  SM->>CTX: 装配上下文
  CTX->>FS: 读取历史对话与工具轨迹
  CTX->>DB: 读取业务对象与历史结果
  CTX->>TMP: 读取临时经验候选
  CTX->>PER: 读取持久经验
  CTX-->>MA: 返回压缩后的运行上下文

  MA->>KB: 检索相关知识 chunk
  KB-->>MA: 返回知识片段与元数据
  MA->>DB: 查询或写入业务过程数据
  MA->>FS: 追加模型消息与工具调用轨迹
  MA-->>API: 流式返回阶段性结果
  API-->>UI: 推送进度与中间输出

  MA->>SP: 一轮结束后发送结构化抽取任务
  SP->>DB: 写入 assessment、评分、结论等结构化结果
  SP-->>SM: 返回落库状态

  MA->>SE: 一轮结束后发送经验提取任务
  SE->>TMP: 写入候选经验条目
  SE->>PER: 晋升高置信经验
  SE-->>SM: 返回经验处理状态

  SM->>FSM: 根据主流程与子任务结果进入 completed 或 failed
  SM->>FS: 写入 Session 快照与最终 manifest
  SM->>OBS: 记录状态迁移、耗时、错误与审计事件
  API-->>UI: 返回最终评估结果
```
