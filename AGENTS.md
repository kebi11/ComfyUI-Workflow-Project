# AGENTS.md

本文件是 AI Coding Agent（Grok Build、Codex 等）进入本仓库的入口。详细规则见 `docs/` 与 `PROJECT_STRUCTURE.md`，不要把技术细节全部堆在这里。

## 1. 项目是什么

本仓库用于管理、开发、测试和迭代 ComfyUI 工作流，当前主线是**人物一致性**：身份、面部、身材、肤色、风格保持，以及姿势控制、换装、构图与 ControlNet / IPAdapter 方案验证。

重点不是单纯保存 `.json`，而是让实验可追踪、可复现、可对比。

## 2. 开始工作前必须阅读

按顺序阅读：

1. `AGENTS.md`（本文件）
2. `PROJECT_STRUCTURE.md`
3. `docs/项目目标.md`
4. `docs/当前环境.md`
5. `docs/开发规范.md`

再按任务补读：

- `docs/节点说明.md`
- `docs/模型说明.md`
- `docs/参数说明.md`
- `docs/已知问题.md`
- 相关 `workflows/`、`prompts/`、`experiments/`、`results/`、`logs/`

没有读完上述文件，禁止大规模改工作流。

## 3. 哪些目录可以改

| 目录 / 文件 | 可否修改 | 说明 |
|---|---|---|
| `workflows/experiments/` | 可以 | 实验工作流的唯一写入区 |
| `workflows/archive/` | 可以 | 只归档，不覆盖 |
| `prompts/` | 可以 | 稳定 Prompt 请写新版本，不覆盖已验证稿 |
| `experiments/` | 可以 | 实验记录，按日期新增 |
| `results/` | 可以 | 输出与对比图 |
| `logs/` | 可以 | 追加变更与实验索引 |
| `docs/` | 可以 | 只更新与本次改动相关的事实 |
| `references/` | 可以 | 只新增参考素材，不与结果混放 |
| `scripts/` | 可以 | 辅助工具放这里 |
| `temp/` | 可以 | 临时文件，不得作为唯一存档 |
| `workflows/baseline/` | 原则上禁止直接改 | 派生新实验，不覆盖基线 |
| `workflows/release/` | 原则上禁止直接改 | 仅验证通过后新增正式版本 |
| `PROJECT_STRUCTURE.md` | 原则上不改 | 结构规范，不因单次实验改动 |

## 4. 哪些文件不能直接覆盖

- `workflows/baseline/*.json`
- `workflows/release/*.json`
- 已编号的历史实验工作流与 `experiments/**/EXP-*.md`
- 已验证的正式 Prompt

除明显笔误外，新增版本，不覆盖历史。

## 5. 实验必须怎么做

```text
Baseline / 已有实验
    ↓
创建新实验版本（不覆盖）
    ↓
单变量修改
    ↓
运行并保存结果
    ↓
写 EXP-XXX.md
    ↓
记录结论与下一步
```

- 任何实验都必须从明确基线或已有实验派生。
- 优先单变量；同时改多个参数时必须在实验文档里声明，且不得把结果归因到单个参数。
- 命名：`功能-方案-v版本.json`，禁止「最终版.json」「最新.json」。
- 当前机器约 **6 GB VRAM**，设计时必须考虑能否跑起来。

## 6. 完成后必须交付

每次重要修改至少给出：

1. **改了什么**：节点、连接、参数、Prompt、模型
2. **为什么改**：对应哪个问题或假设
3. **生成了什么**：
   - 新工作流路径
   - 实验文档路径
   - 结果路径
4. **与 Baseline 的差异**
5. **结论**：已解决 / 部分解决 / 未解决 / 新问题
6. **下一步**：下一轮最值得测的变量

## 7. 如何记录实验结果

1. 工作流 JSON 放到 `workflows/experiments/`
2. 实验记录放到 `experiments/YYYY-MM-DD/EXP-XXX.md`（模板见 `experiments/EXP-TEMPLATE.md`）
3. 输出图放到 `results/experiments/`，对比图放到 `results/comparison/`
4. 在 `logs/CHANGELOG.md` 追加项目级变化
5. 在 `logs/experiment-log/INDEX.md` 追加一行索引
6. 新结论同步到 `docs/参数说明.md`、`docs/已知问题.md`、必要时 `docs/节点说明.md`

## 8. 禁止事项（摘要）

不得未经说明覆盖 Baseline / Release；不得删除历史实验；不得无目的重构整图；不得无视 6 GB 显存；不得把未测试工作流标为 Release；不得把参考图和生成结果混放。完整清单见 `PROJECT_STRUCTURE.md` 第 22 节。
