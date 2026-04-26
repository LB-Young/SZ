# jobs/handlers

异步任务处理器，与 **§4** 数据流一致：

- `persist-assessment`：主链路成功后子 Agent 写 **D**（assessment / advice / evidence / rule_hits / event_logs）。
- `post-close-analyze`：`assessment_closed` 后读 **FS**、补写 **D**、写 **E**（tmp JSON）。
- `experience-promote`：可选定时任务，tmp → persistent 晋升与过期。

由 `jobs/worker` 或等价队列消费者调用；与 `session/orchestrator` 派发解耦。
