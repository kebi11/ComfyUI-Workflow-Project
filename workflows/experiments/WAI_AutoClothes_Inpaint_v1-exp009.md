# WAI_AutoClothes_Inpaint_v1-exp009

对应 JSON：`WAI_AutoClothes_Inpaint_v1-exp009.json`  
设计说明书：`WAI AutoClothes 开发文档：上下文约束 + 旧衣预填充 + 双阶段融合.md`  
补丁：`WAI AutoClothes 开发文档补丁：开工 Gate + Contract 8 + EXP-009 验收契约.md`  
实验记录：`experiments/2026-08-17/EXP-009.md`  
派生自：`WAI_AutoClothes_Inpaint_v1-phase3.json`

状态：**图已建成，未覆盖 Phase 3。** 本版只实现 EXP-009（Small Context）。未加入 Pre-fill / 降 Denoise / Harmonization。尚未 Queue。

## 这一版解决什么

只让 Crop 额外看到人物与近景上下文，验证 P13 是否因此减轻。

```text
M_CONTEXT
=
Hard Protect（脸/发/帽）
+ Visible Skin raw（手臂/腿）
+ Near BG 24px
        ↓
InpaintCropImproved.optional_context_mask
```

Context 只控制 Crop 窗口。它不是肤色迁移、风格编码或光照迁移。

## 必须先看的 Preview

1. ④ FINAL EDIT MASK（与 Phase 3 相同，仍是唯一编辑区）
2. ⑤ CONTEXT MASK
3. ③ FINAL CROP PREVIEW

工程验收先于视觉：先确认 Context 确实改变了 Crop，且没有接近整图。  
不要用 1024 target 预览尺寸去除以原图面积。源窗口近似为 `(Final Edit ∪ Context)` 的包围盒。

## 保持不变

```text
Denoise = 0.95
Pre-fill = OFF
Harmonization = OFF
Color Match = 0
Checkpoint / Seed / Prompt / Sampler / CFG / Steps
Crop expand 14 / blend 16 / context 1.8 / target 1024 / CPU
```

## 与 Phase 3 的差异

- 复用已有 Hard / Skin / Garment Mask，不再跑第四次 Segformer
- Visible Skin 用 Shrink 前的 raw，避免 Context 比保护区还瘦
- Nearby 固定 24px，禁止在本图里改 32 / 48
- Crop.mask 仍只吃 Final Edit

## 6 GB

没有新增大模型。若 Context 把 Crop 拉得接近整图，先看 ⑤ / ③，不要先换主模型。
