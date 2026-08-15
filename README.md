# ComfyUI 工作流项目

用于管理、开发、测试和迭代 ComfyUI 工作流。当前主线是人物一致性：身份、面部、身材、肤色、风格保持，以及姿势控制、换装和构图实验。

本仓库保存的不只是 `.json` 工作流，还包括目标、环境、实验记录、参考素材和结果对比，方便人和 AI Coding Agent 长期迭代。

## 当前状态

| 项 | 状态 |
|---|---|
| 项目框架 | 已按 `PROJECT_STRUCTURE.md` 搭好 |
| Baseline 工作流 | 尚未录入 |
| Release 工作流 | 尚无正式版本 |
| 推荐正式工作流 | 暂无，请先建立并验证 Baseline |
| 硬件约束 | RTX 4050 Laptop / 约 6 GB VRAM / 16 GB RAM |

开始实际出图前，先补齐：

1. `docs/当前环境.md` 中的 ComfyUI 版本、模型与自定义节点
2. `docs/模型说明.md` 中的本地模型路径
3. `workflows/baseline/` 中的第一份基线工作流

## 如何使用

```text
放入参考图          →  references/<类别>/
记录稳定 Prompt     →  prompts/
导入或新建基线      →  workflows/baseline/
从基线派生实验      →  workflows/experiments/
写实验记录          →  experiments/YYYY-MM-DD/EXP-XXX.md
保存输出            →  results/
验证通过后晋升      →  workflows/release/
```

工作流生命周期：

```text
Baseline → Experiment → 测试 → 调整 → 确认稳定 → Release
```

命名约定：`功能-方案-v版本.json`，例如 `character-ipadapter-v01.json`。正式版使用 `character-consistency-v1.0.json`。

## 文档入口

| 文件 | 用途 |
|---|---|
| [AGENTS.md](AGENTS.md) | AI Agent 工作入口 |
| [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) | 目录、生命周期与开发规则 |
| [docs/项目目标.md](docs/项目目标.md) | 必须保持 / 允许改变 / 优先级 |
| [docs/当前环境.md](docs/当前环境.md) | 真实硬件与 ComfyUI 环境 |
| [docs/开发规范.md](docs/开发规范.md) | 修改与实验规范 |
| [docs/README.md](docs/README.md) | 知识库索引 |

## 当前研究方向

- 人物身份锁定（IPAdapter / 人脸参考）
- 身材与比例保持
- 肤色保持
- 风格保持
- OpenPose / ControlNet 姿势控制
- 换装时减少身份与体型漂移
- 6 GB 显存下的节点与分辨率取舍

## 目录一览

```text
docs/          长期知识库
workflows/     工作流 JSON（baseline / experiments / release / archive）
references/    按职责分类的参考图
prompts/       独立管理的 Prompt
experiments/   实验记录（不是 JSON）
results/       生成结果与对比
logs/          变更日志与实验索引
scripts/       辅助脚本
temp/          临时文件
```
