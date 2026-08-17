# WAI AutoClothes 开发文档补丁

与主文档冲突时，以本补丁为准。

## A. 新增：正式开工 Gate

建议插入到“硬性开发契约”之前。

# 开工前置 Gate

本阶段不得仅依据聊天记录、人工描述或“本地曾经 Queue 过”作为正式对照依据。

Phase 3 必须首先形成仓库内可追踪的正式 Reference。

开始 EXP-009 前必须满足：

```text
[ ] Phase 3 实际 Queue 已完成
[ ] Phase 3 最终成品已保存
[ ] Phase 3 Final Edit Mask 已保存或可以复现
[ ] 输入图已登记
[ ] Workflow 已登记
[ ] Seed 已登记
[ ] EXP-007 已回写真实运行结果
[ ] P13 已写入 docs/已知问题.md
[ ] 已明确记录 P08 ⊂ P13
```

判断依据：

```text
仓库中的正式记录
>
聊天记录
>
人工口头说明
```

如果：

```text
本地已经跑过
```

但：

```text
EXP-007 仍写“尚未 Queue”
```

则 Agent 必须首先完成实验记录同步。

在正式记录完成前：

```text
禁止开始 EXP-009
```

---

# Phase 3 Reference 最低记录要求

至少记录：

```text
Reference ID：
Workflow：
Input Image：
Seed：
Prompt：
Final Edit Mask：
Output Image：
VRAM / 是否 OOM：
主要视觉问题：
对应 Issue：
```

Phase 4 后续所有实验必须与同一个 Phase 3 Reference 对照。

不得在实验中悄悄更换：

```text
输入图
Seed
基础 Prompt
Checkpoint
Sampler
```

然后仍称其为 Phase 3 对照。

---

## B. 修订：Context 的严格定义

建议替换原“Context 的准确含义”章节。

# Context 的严格定义

Context Mask 的直接职责只有：

> 控制 `InpaintCropImproved` 在计算 Crop 时，需要额外包含哪些原图区域。

正式逻辑：

```text
M_FINAL_EDIT
+
M_CONTEXT
        ↓
InpaintCropImproved
        ↓
共同决定 Crop Window
        ↓
Cropped Original Image
```

Context Mask：

```text
不是视觉编码器
不是肤色迁移
不是 Style Adapter
不是 Relighting
不是颜色匹配节点
不是 Reference Conditioning
```

因此：

```text
Context Mask
≠
Skin Tone Transfer
```

```text
Context Mask
≠
Style Encoding
```

```text
Context Mask
≠
Lighting Transfer
```

其真正作用是：

```text
让更多相关原图像素
进入 Generation 可以看到的 cropped_image
```

从而：

```text
模型有机会利用这些信息
```

但必须明确：

> 模型“能够看到”不等于模型“一定会正确利用”。

---

# Context 失败判定

如果已经确认：

```text
Visible Arms
Visible Legs
Face
Nearby Background
```

都合理进入 Crop，

但新生成区域仍然：

```text
明显偏冷
明显偏白
光照方向错误
纹理不一致
```

则不得继续无限扩大 Context。

此时记录：

```text
普通 Spatial Context 不足
```

并保留到后续：

```text
Reference Conditioning
Local Color Transfer
Relighting
```

阶段处理。

---

## C. 修订：Context 初始范围

替换原较大的 Nearby Context 建议。

# Context 第一轮范围

EXP-009 第一轮统一使用：

```text
Nearby Context = 24 px
```

只有在 Crop Preview 明确显示上下文不足时，才允许继续：

```text
24
↓
32
↓
48
```

第一阶段禁止直接使用：

```text
80
120
160
```

---

# Context 停止规则

Context 的目标是：

```text
足够
```

而不是：

```text
越大越好
```

如果 24 px 已经能让：

```text
关键裸露皮肤
人物主体
人物附近环境
```

进入 Crop，

则不允许仅为了“可能更融合”继续扩大到 32 / 48。

---

## D. 新增：Crop 工程验收

# Context Crop 工程验收

EXP-009 必须先通过工程验收，再评价视觉收益。

必须提供：

```text
CONTEXT MASK PREVIEW
```

以及：

```text
FINAL CROP PREVIEW
```

尽可能记录：

```text
Crop Width
Crop Height
Original Width
Original Height
Crop Area Ratio
```

---

# Crop 面积预警

如果：

```text
Crop Area / Original Area
≈ 60%–70% 或更高
```

则标记：

```text
Context Crop Too Large
```

该比例属于工程预警值，不是绝对失败阈值。

Agent 应首先检查：

```text
是否存在无意义 Context
是否 Nearby Grow 过大
是否某个 Context 类别把范围拉远
```

而不是直接继续生成。

---

## E. 新增：Contract 8

建议接在 Contract 7 后。

# Contract 8｜第二遍必须共享 Generation 的 Crop 坐标系

Phase 4 中存在两个坐标空间：

```text
Original Image Space
```

和：

```text
Generation Crop Space
```

Phase 3 中的：

```text
M_FINAL_EDIT
M_HARD_PROTECT
M_MANUAL_PROTECT
```

最初均属于：

```text
Original Image Space
```

而 Harmonization Pass 处理的是：

```text
Generation Crop Space
```

因此第二遍使用的所有 Mask 必须映射到与 Generation 完全相同的 Crop。

---

## Contract 8.1 禁止第二次独立计算 Crop

禁止：

```text
Generation
→ Crop A

Harmonization
→ 再单独计算 Crop B
```

即使：

```text
Crop A
```

和：

```text
Crop B
```

视觉上很接近，也不允许。

因为：

```text
1 px
2 px
resize scale
aspect ratio
```

的差异都可能导致 Protection Mask 与实际人物错位。

---

## Contract 8.2 必须共享同一 Crop Geometry

以下内容必须保持一致：

```text
Crop X
Crop Y
Crop Width
Crop Height
Resize Scale
Target Width
Target Height
```

因此必须得到：

```text
M_FINAL_EDIT_CROP
M_HARD_PROTECT_CROP
M_MANUAL_PROTECT_CROP
```

并保证：

```text
它们和 Generation Result
具有完全一致的宽高和坐标系
```

---

## Contract 8.3 Hard Protect 错位属于阻断错误

如果出现：

```text
保护 Mask 与 Crop 图不对齐
```

禁止继续 Queue Harmonization。

必须先解决：

```text
Mask Crop / Resize / Coordinate Mapping
```

否则可能出现：

```text
原本保护眼睛
↓
Mask 偏移
↓
实际保护到眼睛旁边
↓
第二遍修改人脸
```

这属于：

```text
Blocking Error
```

而不是视觉参数问题。

---

## F. 新增：第二遍 Fooocus Conditioning 契约

# Contract 9｜Harmonization 必须重新建立 Fooocus Inpaint Conditioning

Generation Pass 和 Harmonization Pass 可以共享：

```text
同一个 Checkpoint
同一个 CLIP
同一个 VAE
同一个 Fooocus Inpaint Patch
```

但不能共享同一份：

```text
latent_inpaint
latent_samples
```

原因：

Harmonization 使用的是新的：

```text
Generation Result
+
M_HARMONIZE
```

因此必须重新建立：

```text
Pass B Inpaint Conditioning
```

---

# Generation Pass

第一遍：

```text
Pre-filled Crop
+
M_FINAL_EDIT_CROP
        ↓
INPAINT_VAEEncodeInpaintConditioning
        ↓
latent_inpaint_A
latent_samples_A
```

然后：

```text
latent_inpaint_A
        ↓
INPAINT_ApplyFooocusInpaint
        ↓
MODEL_A
```

同时：

```text
latent_samples_A
        ↓
KSampler A
```

得到：

```text
Generation Crop Result
```

---

# Harmonization Pass

第二遍必须重新：

```text
Generation Crop Result
+
M_HARMONIZE
        ↓
INPAINT_VAEEncodeInpaintConditioning
        ↓
latent_inpaint_B
latent_samples_B
```

然后重新：

```text
Base WAI MODEL
+
同一 Fooocus Patch
+
latent_inpaint_B
        ↓
INPAINT_ApplyFooocusInpaint
        ↓
MODEL_B
```

再：

```text
MODEL_B
+
latent_samples_B
+
Harmonization Conditioning
        ↓
KSampler B
```

---

# 重要区别

允许：

```text
Checkpoint Loader = 1
```

禁止：

```text
Checkpoint Loader A
Checkpoint Loader B
```

因此：

```text
重新 Apply Fooocus
≠
重新 Load Checkpoint
```

---

# 最终正确结构

```text
CheckpointLoaderSimple
        ↓
     WAI MODEL
        │
        ├───────────────────────────┐
        │                           │
        ▼                           ▼
Generation                    Harmonization
Conditioning                  Conditioning
        │                           │
latent_inpaint_A             latent_inpaint_B
        │                           │
        ▼                           ▼
Apply Fooocus A              Apply Fooocus B
        │                           │
MODEL_A                      MODEL_B
        │                           │
KSampler A                   KSampler B
```

---

## G. 修订：Harmonize Mask 正式公式

替换原 Harmonization Mask 公式。

# Harmonization Mask 正式定义

首先在：

```text
Generation Crop Space
```

中得到：

```text
M_FINAL_EDIT_CROP
```

然后：

```text
M_HARMONIZE_RAW
=
Grow(M_FINAL_EDIT_CROP, +8)
```

再进行：

```text
Blur = 8~12
```

得到柔化区域。

最终：

```text
M_HARMONIZE
=
M_HARMONIZE_RAW
-
M_HARD_PROTECT_CROP
-
M_MANUAL_PROTECT_CROP
```

---

# Hard Protect 必须覆盖到第二遍

至少包括：

```text
Face
Eyes
Hair 主体
Hat
```

这些区域：

```text
Generation Pass
```

不能修改，

```text
Harmonization Pass
```

也不能修改。

---

# Skin 的处理

不默认从 Harmonization 中完全删除所有：

```text
arms
legs
waist skin
```

因为第二遍可能需要轻微调整：

```text
局部肤色色温
衣服接触阴影
边缘亮度
```

因此这些区域默认属于：

```text
Soft Protection
```

如果实测发现第二遍明显改变裸露皮肤主体，

再建立：

```text
M_SKIN_CORE_PROTECT_CROP
```

并作为独立实验变量。

---

## H. 新增：Harmonization 接法禁止项

# Harmonization 接法硬限制

唯一允许的流程：

```text
Original
↓
One Crop
↓
Pre-fill
↓
Generation
↓
Harmonization
↓
One Stitch
```

禁止：

```text
Generation
↓
Stitch
↓
Full Image Harmonization
```

禁止：

```text
Generation
↓
Stitch A
↓
Harmonization
↓
Stitch B
```

禁止：

```text
Pass B 重新计算 Crop
```

禁止：

```text
Pass B 使用原始 Full-size Protect Mask
```

禁止：

```text
Pass B 直接复用 Pass A latent
```

---

## I. 修订：EXP-009

替换原 EXP-009 定义。

# EXP-009｜Small Context Only

## 实验目的

回答：

> 在不修改生成模型、Denoise、Pre-fill 和 Harmonization 的情况下，只让 Crop 看到更合理的原图上下文，是否改善生成结果？

---

## 唯一主动变量

新增：

```text
M_CONTEXT
```

并接入：

```text
InpaintCropImproved.optional_context_mask
```

---

## 第一轮 Context

固定：

```text
Nearby Context = 24 px
```

不得同时测试：

```text
24 / 32 / 48
```

EXP-009 第一张正式对照只使用：

```text
24 px
```

---

## 保持不变

```text
Pre-fill = OFF
Generation Denoise = 0.95
Harmonization = OFF
Color Match = 0
Checkpoint 不变
Seed 不变
Prompt 不变
Sampler 不变
CFG 不变
Steps 不变
```

---

# EXP-009 第一阶段验收：工程层

必须先检查：

```text
[ ] Context Mask 正常
[ ] Context 确实影响 Crop
[ ] 重要参考区域进入 Crop
[ ] Crop 没有明显接近整图
[ ] Crop 长宽比正常
[ ] Target=1024 下可运行
[ ] 无 OOM
```

第一阶段不评价：

```text
好不好看
```

只评价：

```text
Context / Crop 机制是否正确
```

---

# EXP-009 第二阶段验收：视觉层

工程验收通过后，再比较：

```text
Phase 3 Reference
vs
EXP-009
```

评价：

```text
肤色
色温
阴影
人物与环境关联
整体图层感
```

---

# EXP-009 停止规则

如果 24 px 已经让重要上下文进入 Crop：

```text
不得自动升级到 32 / 48
```

如果 24 px 明显不足：

建立新的单变量实验：

```text
24 → 32
```

如果 32 仍不足：

再：

```text
32 → 48
```

---

## J. 修订：EXP 链路

正式实验顺序改为：

```text
Phase 3 Reference
        ↓
EXP-009
Small Context Only
        ↓
先验 Crop
        ↓
再验 Visual
        ↓
EXP-010
Pre-fill Only
        ↓
EXP-011
Lower Generation Denoise
        ↓
EXP-012
Harmonization Pass
        ↓
EXP-013
Harmonization Strength Scan
```

每轮只回答一个问题。

---

# EXP-010

只新增：

```text
Pre-fill
```

保持：

```text
Generation Denoise = 0.95
Harmonization = OFF
```

回答：

> Pre-fill 是否有效削弱旧衣结构？

---

# EXP-011

只修改：

```text
Generation Denoise
0.95 → 0.78
```

回答：

> 在 Pre-fill 已经存在的情况下，降低破坏性是否可以更多保留原图视觉特征？

---

# EXP-012

只新增：

```text
Harmonization Pass
```

回答：

> 第二遍低强度 Crop 内融合是否降低 P13？

---

# EXP-013

仅扫描：

```text
0.15
0.20
0.25
```

回答：

> Harmonization 最合适强度区间在哪里？

---

## K. 修订：6 GB OOM 策略

如果 Context 导致：

```text
Crop 过大
```

或：

```text
OOM
```

正式回退顺序：

```text
Context 48 → 32
↓
Context 32 → 24
↓
删除无价值 Context
↓
重新检查 Crop
↓
Target 1024 → 896
```

不得首先：

```text
删除 Context
```

不得首先：

```text
更换主模型
```

不得首先：

```text
删除 Harmonization 设计
```

因为这些属于本阶段核心实验变量。

---

# L. 开工后 Agent 执行规则

Agent 开始派生下一版工作流前必须：

```text
1. 读取 Phase 3 Reference
2. 读取 EXP-007
3. 读取 P13
4. 读取本开发文档
5. 检查当前 phase3.json
```

然后：

```text
复制 phase3
↓
建立新实验工作流
↓
只实现 EXP-009
```

第一版禁止提前加入：

```text
Pre-fill
Lower Denoise
Harmonization
```

即使这些节点最终一定会需要。

原因：

> 先验证 Context/Crop 契约本身。

EXP-009 通过后再进入 EXP-010。

---

# M. 当前开工边界

在本补丁生效后：

```text
架构设计阶段结束
```

下一步不再继续扩展概念模块。

正式进入：

```text
Phase 3 Reference
↓
EXP-009
↓
EXP-010
↓
EXP-011
↓
EXP-012
```

除非实际实验暴露新的结构性阻断问题，否则不得在 EXP-009 前继续增加新的大模块。