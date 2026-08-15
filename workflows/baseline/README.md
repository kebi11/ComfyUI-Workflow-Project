# workflows/baseline

保存经过确认的实验基线。后续所有结果都和这里的版本比较。

```text
baseline/
├─ README.md
└─ character-base-v01.json   ← 第一份基线放这里
```

## 规则

- 原则上不得直接修改已有 baseline JSON
- 需要改基线时：先在 `experiments/` 验证，再新增 `character-base-v02.json`
- 每个 baseline 应在本页登记来源、用途和限制

## 当前基线

尚无。导入或搭建第一份可运行工作流后，按下面格式追加：

```text
文件：
来源：
用途：
已验证分辨率：
已知限制：
对应实验：
```
