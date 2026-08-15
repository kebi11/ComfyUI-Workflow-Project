# 01_WAI_人物锁定换装

对应 JSON：`01_WAI_人物锁定换装.json`  
设计说明书：`方案一开发文档：原人物锁定换装.md`  
派生自：本机 `gu_keep_character.json`（已归档到 `workflows/archive/`）

状态：**已建成，未实测**。尚未用 5 套服装跑验收，不能当作 Release。

## 这张图做什么

原图 + 服装 Mask + Inpaint。MASK 外尽量保持原始像素。不改姿势。

```text
原图 800×1216
    +
服装 Mask（Grow 8 / Blur 6）
    +
VAE Encode (for Inpainting)
    +
KSampler 24 / CFG 5 / denoise 1.00
    +
可选第二遍：普通 VAEEncode + Mask，12 / CFG 4.5 / denoise 0.15（不再擦成灰）
```

## 怎么用

1. 在「底图-右键画服装Mask」加载原图。
2. 右键该节点 → Open in MaskEditor，只涂要换的衣服、鞋袜、配饰。
3. 只改「正向-新服装【每次只改这里】」。
4. 跑图。看「预览服装Mask」确认没有涂到脸/头发/皮肤。
5. 第一遍结果会存成 `01_WAI_cloth_p1`；成品存成 `01_WAI_cloth_lock`。
6. 衣缘已经够好时，把第 5 组 Bypass，只用第一遍。

## 默认参数

| 项 | 值 | 来源 |
|---|---|---|
| Checkpoint | `waiIllustriousSDXL_v170.safetensors` | 原工作流，已在本机确认 |
| 尺寸 | 800 × 1216 | 方案一指定，且与 `gu_keep_character` 控件一致 |
| Seed | 114514 / fixed | 便于对照 |
| 第一遍 | 24 / 5 / euler_ancestral / normal / **1.00** | `VAEEncodeForInpaint` 先擦成灰，必须高 denoise 才能长出新衣服 |
| 第二遍 | 12 / 4.5 / euler_ancestral / normal / **0.15** | 方案一推荐 |
| Mask Grow | 8 px | 方案一 4–12 的中值 |
| Mask Blur | 6 px / sigma 1.5 | 方案一 4–10 的中值 |
| VAEEncodeForInpaint grow_mask_by | 6 | ComfyUI 默认，处理 VAE 8px 网格缝 |

## 相对原工作流改了什么

原 `gu_keep_character.json` 是整图两遍 Img2Img，人物会一起被重绘。

| 原节点 | 现在 |
|---|---|
| `VAEEncode` | `VAEEncodeForInpaint` |
| LoadImage 的 MASK 未使用 | 画服装 Mask，并 Grow + Blur |
| 单一混合 Prompt | 人物 / 服装 / 场景拆开，只改服装 |
| 两遍都是整张 latent | 两遍都带同一张服装 Mask |
| 第一遍 denoise 0.18 | 第一遍 1.00（局部 Inpaint 必须填满擦除区） |

## 为什么加 SetLatentNoiseMask

`VAEEncodeForInpaint` 内部会把 Mask `round()` 成硬边。  
先用它擦掉旧衣服，再用 `SetLatentNoiseMask` 写回 Grow+Blur 后的软边，对应方案一第 8 节：服饰和皮肤交界不能完全硬切。

全部是 ComfyUI 0.32 核心节点，没有自定义节点。

## 下一轮该测什么

同一张原图、同一 Mask、同一 seed，衣服能长出来之后再往下扫：

```text
A = 1.00
B = 0.85
C = 0.75
```

不要从 0.22–0.38 扫。那个区间是整图 Img2Img 的习惯，配 Inpaint 擦除只会留下灰块。

再换至少 5 套服装，按方案一第 14 节验收。
