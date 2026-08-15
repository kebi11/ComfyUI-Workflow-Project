# WAI_AutoClothes_Inpaint_v1-phase2

对应 JSON：`WAI_AutoClothes_Inpaint_v1-phase2.json`  
实验记录：`experiments/2026-08-15/EXP-005.md`  
派生自：`WAI_AutoClothes_Inpaint_v1-phase1.json`

状态：**图已建成。** 需要重启 ComfyUI 后加载 LayerStyle。尚未出图。

## 这一版解决什么

文档 Phase 2：自动识别衣服边界，去掉手绘 Mask 作为正式入口。

```text
原图
→ SegformerB2ClothesUltra（dress + upper + skirt + belt）
→ MaskGrow 6 / Blur 4
→ Crop 1024 CPU
→ Fooocus + WAI
→ Stitch
```

单变量：只换 Mask 来源。采样、Prompt、Crop 与 Phase 1 相同。

## 默认勾选

开：`dress` / `upper_clothes` / `skirt` / `belt`  
关：`face` / `hair` / `hat` / `arms` / `legs` / `shoe` / `bag` / `scarf`

原图是薄纱裹裙，可能被判成 dress 或 skirt。四类并集更不容易漏纱。  
凉鞋、草帽、脸、已露出的腿臂不进编辑区。

## 6 GB

- `detail_method = GuidedFilter`，不开 VITMatte
- `max_megapixels = 1.0`
- Crop 仍走 CPU
- Segformer B2 约百兆，先于 WAI 跑完

OOM：Segformer `device` 改 cpu，或 Crop target 改 896。

## 手工 Mask

左侧灰色组仍保留 Phase 1 的 `gu_clothes_mask_hand_v1.png`，默认 Bypass。  
自动 Mask 明显错时再打开对照，不要和自动路同时接到 Crop。

## 依赖

```text
custom_nodes/ComfyUI_LayerStyle
models/segformer_b2_clothes/     mattmdjaga/segformer_b2_clothes
```

重启 ComfyUI 后，节点应出现在 `😺dzNodes / LayerMask`。
