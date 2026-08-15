# experiments/

注意区分：

| 路径 | 内容 |
|---|---|
| `workflows/experiments/` | 实验工作流 JSON |
| `experiments/`（本目录） | 实验记录文档 |

## 目录约定

```text
experiments/
├─ README.md
├─ EXP-TEMPLATE.md
└─ YYYY-MM-DD/
   ├─ EXP-001.md
   ├─ EXP-002.md
   └─ EXP-003.md
```

- 按**实验当天日期**建子目录
- 编号全局递增，不要每天从 `EXP-001` 重新开始
- 下一号以 `logs/experiment-log/INDEX.md` 为准
- 复制 `EXP-TEMPLATE.md` 再填，不要缺字段

当前还没有正式实验。建立 Baseline 后从 `EXP-001` 开始。
