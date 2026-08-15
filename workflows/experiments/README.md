# workflows/experiments

正在测试的工作流 JSON 放这里。允许快速迭代，但不要覆盖旧版本。

示例：

```text
character-ipadapter-v01.json
character-ipadapter-v02.json
character-openpose-v01.json
character-body-lock-v01.json
character-clothing-v01.json
```

## 规则

- 从 baseline 或已有实验复制后改，不要凭空覆盖正式版
- 文件名：`功能-方案-v版本.json`
- 每次重要实验同时写 `../../experiments/YYYY-MM-DD/EXP-XXX.md`
- 失败版本不要删，确认无用后再移到 `../archive/`

## 当前实验

暂无。
