# WAI_AutoClothes_Inpaint_v1-phase1

对应 JSON：`WAI_AutoClothes_Inpaint_v1-phase1.json`  
实验记录：`experiments/2026-08-15/EXP-004.md`  
派生自：`workflows/baseline/2/WAI_Universal_Inpaint_v1.json`

状态：**图已建成，待本机 Queue 出图。** 本阶段仍用手绘/已导出 Mask，没有自动认衣。

## 这一版解决什么

用户目标：自动找到衣服边界并换装，同时锁住身材和肤色。

按文档，自动认衣（Segformer）必须等 Inpaint 主干先跑通。Phase 1 只验证：

```text
人物图 + 服装 Mask
→ Crop 1024 CPU
→ Fooocus + WAI
→ Stitch
```

Mask 外（脸、帽、发、手、已露出的腿臂、背景）不进 VAE。

## 输入

| 入口 | 文件 | 位置 |
|---|---|---|
| 底图 | `gu_ref.jpg` | ComfyUI `input/`，仓库副本 `references/character/gu_ref.jpg` |
| 服装 Mask | `gu_clothes_mask_hand_v1.png` | ComfyUI `input/`，仓库副本 `references/clothing/` |

Mask 来源：旧 clipspace 手绘 alpha，已转成白=重绘 / 黑=保留，尺寸 1072×1856，与原图一致。

白区应盖住：整件薄纱、系结、裙摆。  
白区不应盖住：脸、草帽、头发主体、拿雪糕的手、已露出的手臂和腿、背景。

## 采样（沿用 Universal Inpaint）

```text
Seed        114514 / fixed
Steps       24
CFG         5.0
Sampler     euler_ancestral
Scheduler   normal
Denoise     0.95
Crop        expand 14 / blend 16 / context 1.8 / 1024 / CPU
Color Match 0（关）
```

## 与 Universal Inpaint 的差异

- 底图改为已存在的 `gu_ref.jpg`（baseline 里的 `顾.jpg` 原先在 input 中缺失）
- Mask 从 LoadImage 右键涂抹改为独立文件，方便下一阶段用 Segformer 替换
- Prompt 写目标服装，并加上肤色/体型约束
- 负向补：旧衣残影、肤色漂移、腰臀变粗

## 6 GB

与 Universal Inpaint 相同。OOM 时 Crop target 1024 → 896。不要叠 SAM2 / ControlNet。

## 下一阶段（未做）

Phase 1 出图通过后，才装 `ComfyUI_LayerStyle`，用 `SegformerB2ClothesUltra` 替换节点 18/19。
