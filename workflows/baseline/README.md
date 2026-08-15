# workflows/baseline

保存经过确认的实验基线。后续所有结果都和这里的版本比较。

```text
baseline/
├─ README.md
├─ 方案一开发文档：原人物锁定换装.md
├─ 01_WAI_人物锁定换装.json
└─ 01_WAI_人物锁定换装.md
```

## 规则

- 原则上不得直接修改已有 baseline JSON
- 需要改基线时：先在 `experiments/` 验证，再新增 `character-base-v02.json`
- 每个 baseline 应在本页登记来源、用途和限制

## 当前基线

```text
文件：     01_WAI_人物锁定换装.json
说明：     01_WAI_人物锁定换装.md
设计：     方案一开发文档：原人物锁定换装.md
来源：     本机 gu_keep_character.json（已归档）
用途：     只换服装，MASK 外尽量保持原图像素，不改姿势
分辨率：   800 × 1216
状态：     已建成，未实测 5 套服装验收
已知限制： 必须手绘服装 Mask；姿势不能变；第二遍默认开启但可 Bypass
对应实验： 尚未开始，下一轮先扫 Denoise 0.22–0.38
```
