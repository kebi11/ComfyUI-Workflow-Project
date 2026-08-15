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

```text
01_WAI_人物锁定换装-v02.json
说明： 方案一防分层，三套 Mask + Composite
对应： experiments/2026-08-15/EXP-001.md

01_WAI_人物锁定换装-v03.json
说明： 在 v2 成品后串 A/B/C 定点修复
对应： experiments/2026-08-15/EXP-002.md

WAI_Universal_Inpaint_v1.json
说明： Crop + Fooocus 单次换装主干
对应： experiments/2026-08-15/EXP-003.md

WAI_AutoClothes_Inpaint_v1-phase1.json
说明： 上述主干 + 独立服装 Mask + 换装 Prompt
对应： experiments/2026-08-15/EXP-004.md

WAI_AutoClothes_Inpaint_v1-phase2.json
说明： Segformer 自动认衣替换手工 Mask
对应： experiments/2026-08-15/EXP-005.md
```
