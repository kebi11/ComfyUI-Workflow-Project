# workflows/release

只保存经过实际验证的正式工作流。

示例：

```text
character-consistency-v1.0.json
character-consistency-v1.1.json
```

## 晋升条件

同时满足才允许放入本目录：

- 覆盖 `docs/项目目标.md` 中当前阶段的核心目标
- 有对照实验和结果图
- 本机（约 6 GB VRAM）可稳定跑通
- 在 `logs/CHANGELOG.md` 写明晋升原因

未测试的实验工作流不得标为 Release，也不得覆盖已有正式版。
