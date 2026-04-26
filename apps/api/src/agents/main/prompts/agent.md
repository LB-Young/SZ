# 目录说明：`apps/api/src/agents/main/prompts`

## 本目录职责

- **Prompt 资源**：system 安全与输出 JSON schema、分阶段 instructions、**动态槽位**（用户原文、多轮合并后的检索块）。  
- 可用纯文本、Jinja2、或 `langchain` 类模板；**禁止**在模板里硬编码具体患者数据样例作默认。
