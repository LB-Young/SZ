# 目录说明：`apps/api/src/agents/main`

## 本目录职责

- **主 Agent**：`graph`/`loop` 实现「感知-决策-执行」、多轮 `search_experience` / `search_knowledge`、**规则 tool**、**流式**输出四字段+引用+时间。  
- `tools.py` 将 `retrieval`、`rules` 包装为可调用 tool；`prompts/` 为 system/模板片段。

## 与 retrieval

- 每轮轮次、子查询、`query_variant` 写入 trace，供子 Agent 落 `evidence`。
