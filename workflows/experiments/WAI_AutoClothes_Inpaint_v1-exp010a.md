# WAI_AutoClothes_Inpaint_v1-exp010a

对应 JSON：`WAI_AutoClothes_Inpaint_v1-exp010a.json`  
实验记录：`experiments/2026-08-17/EXP-010A.md`  
派生自：`WAI_AutoClothes_Inpaint_v1-exp010.json`

状态：**图已建成，未覆盖 EXP-010。** 不改采样。尚未 Queue。

## 这张修什么

1. `pants = ON`（`胡.jpg` 下装）
2. Manual Add 可以覆盖 Soft Protect（手臂/腿），不能覆盖 Hard Protect（脸/发/帽）

```text
AUTO_EDIT = Garment − (Hard + Soft)
EDIT      = AUTO_EDIT + Manual Add
FINAL     = EDIT − Hard Protect − Manual Protect
```

优先级：

```text
Manual Protect  >  Manual Add  >  Soft Protect  >  Auto Garment
Hard Protect 在 Add 之后再减一次
```

## 冻结

Telea / falloff 0、Denoise 0.95、Context 24、Color Match 0、Seed 114514、runtime Prompt、`胡.jpg`。

## Queue 前先填四格

① AUTO GARMENT　② AUTO PROTECT（②b Hard / ②c Soft）　③ FINAL EDIT　④ PRE-FILL

只有「Final 有、Pre-fill 已清、成品又画回」才谈 EXP-011。
