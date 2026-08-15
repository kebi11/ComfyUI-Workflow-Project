# WAI_AutoClothes_Inpaint_v1-phase3

对应 JSON：`WAI_AutoClothes_Inpaint_v1-phase3.json`  
设计说明书：`WAI AutoClothes Phase 3 开发文档：分层 Mask + 人体保护 + 手工覆盖.md`  
实验记录：`experiments/2026-08-16/EXP-007.md`  
派生自：`WAI_AutoClothes_Inpaint_v1-phase2.json`

状态：**图已建成，未覆盖 Phase 2。** 需重启 ComfyUI 后加载。尚未 Queue。

## 这一版解决什么

只改 Mask 系统，不改采样主干。

```text
FINAL EDIT MASK
=
(AUTO CLOTHES + MANUAL ADD)
-
(HARD PROTECT + SKIN CORE + MANUAL PROTECT)
```

对应：P01 / P06 / P07 / P10 / P11。  
不碰：P08 / P09 / P12。Color Match 保持 0。

## 数据流

```text
原图
 → LoadSegformerModel（只加载一次 B2）
 → Garment Setting + Ultra V3 → Grow 4/4 → M_GARMENT_EXPANDED
 → Hard Setting (face/hair/hat) + Ultra V3 → Grow +2/1
 → Skin Setting (arms/legs) + Ultra V3 → Shrink -2/1
 → Manual Add / Manual Protect（默认全黑）
 → REMOVE - PROTECT
 → ④ FINAL EDIT MASK
 → Crop 1024 CPU → Fooocus + WAI → Stitch
```

## 必须先看的 Preview

1. ① AUTO GARMENT MASK  
2. ② AUTO PROTECTION MASK  
3. ③ MANUAL ADD / MANUAL PROTECT  
4. ④ FINAL EDIT MASK（Crop 之前）

Final Edit 错时禁止改 CFG / Steps / Sampler / Denoise。

## 手工文件

```text
input/manual_add_mask.png        白=补删旧衣
input/manual_protect_mask.png    白=强制保护
```

仓库副本：`references/clothing/`。默认全黑，等于只用自动结果。

## 与 Phase 2 的差异

- 衣服检测和人体保护拆成两路  
- 头发与衣服重叠时，头发从编辑区减去  
- 皮肤保护区向内收缩，衣缘仍留给 Inpaint  
- 手工只修正错误，不再和自动二选一  

采样未改：`24 / 5 / euler_ancestral / 0.95 / Crop 14-16-1.8-1024-CPU / seed 114514`

## 6 GB

同一套 B2 模型跑三次分割。OOM 时把 LoadSegformer `device` 改 cpu，不要先动 Denoise。
