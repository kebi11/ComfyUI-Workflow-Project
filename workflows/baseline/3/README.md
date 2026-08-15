# baseline/3 — 自动认衣换装

本目录三份文档定义的是：

`WAI_AutoClothes_Inpaint_v1`

不是上层的「方案三：大幅姿势 + 原头锁定」。

当前产品目标：自动找出衣服边界并换装，同时保持身材和肤色。姿势先不动。

## 开发状态

| 阶段 | 内容 | 状态 |
|---|---|---|
| Phase 1 | 手工 Mask + Crop + Fooocus + Stitch | 实验图已建，待 Queue |
| Phase 2 | Segformer 自动衣服 Mask | 实验图已建，待重启 Queue |
| Phase 3+ | 保护区 / SAM2 / VITMatte | 未开始 |

可运行文件在实验区，不写进本目录 JSON：

```text
workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase1.json
workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase2.json
experiments/2026-08-15/EXP-004.md
experiments/2026-08-15/EXP-005.md
```

当前请加载 Phase 2。先看自动 Mask 预览，不要先开 SAM2 / Human Parser。
