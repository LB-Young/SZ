# storage

文件型持久化适配层，对应架构中的 **FS**（Session 本地三层）与 **E**（经验池 JSON）。

- `session-files`：读写 `data/sessions/{session_id}/01_raw`、`02_snapshots`、`03_manifest.json`。
- `experience-files`：读写 `data/experience/tmp`、`data/experience/persistent`。
- `paths`：从 `core/config` 读取数据根目录，拼接安全路径。

实现语言可为 TypeScript 或 Python，文件名按仓库约定调整。
