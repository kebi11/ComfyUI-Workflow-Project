# baseline/3 — 自动认衣换装

本目录三份文档定义的是：

`WAI_AutoClothes_Inpaint_v1`

不是上层的「方案三：大幅姿势 + 原头锁定」。

当前产品目标：自动找出衣服边界并换装，同时保持身材和肤色。姿势先不动。

## 开发状态

| 阶段 | 内容 | 状态 |
|---|---|---|
| Phase 1 | 手工 Mask + Crop + Fooocus + Stitch | 实验图已建，待 Queue |
| Phase 2 | Segformer 自动衣服 Mask | 实验图已建 |
| Phase 3A | 分层 Mask + 人体保护 + 手工覆盖 | 实验图已建，待 Queue |
| Phase 3B+ | Generation/Composite 双 Mask、SAM2 | 未开始 |

可运行文件在实验区，不写进本目录 JSON：

```text
workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase1.json
workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase2.json
workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase3.json
experiments/2026-08-15/EXP-004.md
experiments/2026-08-15/EXP-005.md
experiments/2026-08-16/EXP-007.md
```

当前请加载 Phase 3。先看 ④ FINAL EDIT MASK，不要先开 SAM2 / 不要改 Denoise。
