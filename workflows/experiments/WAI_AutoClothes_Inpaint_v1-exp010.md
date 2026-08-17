# WAI_AutoClothes_Inpaint_v1-exp010

对应 JSON：`WAI_AutoClothes_Inpaint_v1-exp010.json`  
实验记录：`experiments/2026-08-17/EXP-010.md`  
派生自：`WAI_AutoClothes_Inpaint_v1-exp009.json`  
runtime Prompt：`experiments/2026-08-17/runtime-prompt-exp007-exp009.md`

状态：**图已建成，未覆盖 EXP-009。** 本版只加 Pre-fill。尚未 Queue。

## 唯一变量

```text
Cropped Original + Cropped Final Edit Mask
        ↓
INPAINT_MaskedFill  fill=telea  falloff=0
        ↓
VAE Encode & Inpaint Conditioning.pixels
```

本机没有 `big-lama.pt`，按文档回退 Level B Telea。不要为这一张去装 LaMa。

Pre-fill 不是最终图。只要求旧衣大轮廓减弱。

## 冻结

```text
输入：胡.jpg
Seed：114514
Denoise：0.95
Harmonization：OFF
Context：EXP-009 原样（Hard∪Skin∪NearBG 24）
Color Match：0（reference 仍是未填充的 Crop）
Prompt：与 EXP-007 / EXP-009 runtime 相同
```

## 先看

1. ④ PRE-FILL RESULT：旧衣轮廓有没有糊掉
2. 再看成品，不要用融合好坏评价 Pre-fill
