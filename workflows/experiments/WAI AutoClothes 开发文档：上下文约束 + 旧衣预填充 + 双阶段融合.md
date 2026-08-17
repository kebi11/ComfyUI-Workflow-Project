# WAI AutoClothes 开发文档：上下文约束 + 旧衣预填充 + 双阶段融合

## 1. 文档定位

本开发阶段建立在现有：

```text
WAI_AutoClothes_Inpaint_v1-phase3.json
```

及其已经完成的实际 Queue 对照结果之上。

Phase 3 已经解决或正在解决的主要问题是：

```text
哪里允许修改
哪里必须保护
旧衣区域如何识别
人物区域如何保护
```

即：

```text
REMOVE
-
PROTECT
=
FINAL EDIT MASK
```

本阶段不再继续把主要精力放在自动服装分割。

当前需要解决的新问题是：

> 新生成的衣服、皮肤和局部人体虽然可以正确贴回原图，但视觉上仍然明显像另一个图层，无法自然融入原图。

因此，本阶段正式进入：

```text
视觉融合
```

而不是继续进行：

```text
Mask 边界优化
```

---

# 2. 当前核心问题

正式登记：

```text
[P13] 新生成区域与原图存在明显视觉层差
```

典型表现：

- 新生成皮肤偏冷或偏白
- 原图为暖色环境，但新生成区域没有相同色温
- 衣服与人体之间缺少自然接触阴影
- 新区域锐度与原图不同
- 新区域纹理过于光滑
- 新区域细节密度与原图不一致
- 新区域像后期贴上去
- Stitch 后虽然没有明显硬边，但仍然可以一眼看出重绘区域

---

# 3. P08 / P09 / P13 的关系

正式定义：

```text
P08 ⊂ P13
```

其中：

```text
P08
=
新生成皮肤的肤色 / 光照无法接上原图
```

属于：

```text
P13
=
新生成区域整体视觉融合失败
```

的一种具体表现。

---

P09 则记录：

```text
当前 Color Match 方案无法有效解决 P08
```

因此：

```text
P09
≠
新的视觉问题
```

而是：

> 一个已经验证效果有限或作用机制不匹配的解决路径。

---

# 4. 本阶段核心原理

整个新流程拆成五个职责。

```text
Final Edit Mask
→ 决定哪里允许修改
```

```text
Context
→ 决定 Crop 中还要额外看到哪些原图内容
```

```text
Pre-fill
→ 先降低旧衣结构残留
```

```text
Generation Pass
→ 负责生成正确的新内容
```

```text
Harmonization Pass
→ 负责把新内容融入原图
```

最后：

```text
Stitch
→ 只负责把最终 Crop 放回原图
```

---

# 5. 总体工作流

```text
                    Original Image
                           │
            ┌──────────────┴───────────────┐
            │                              │
            ▼                              ▼
     Final Edit Mask                  Context Mask
            │                              │
            └──────────────┬───────────────┘
                           ▼
                  InpaintCropImproved
                           │
                           ▼
                     Cropped Image
                           │
                           ▼
                       Pre-fill
                           │
                           ▼
                  Generation Pass
                           │
                           ▼
                Generation Crop Result
                           │
                           ▼
                  Harmonization Pass
                           │
                           ▼
                 Harmonized Crop Result
                           │
                           ▼
                InpaintStitchImproved
                           │
                           ▼
                      Final Image
```

---

# 6. 硬性开发契约

本阶段必须遵守以下规则。

## Contract 1

Context Mask 只影响：

```text
Crop 范围
```

不得在文档或工作流说明中声称：

```text
Context Mask = 肤色迁移
Context Mask = 风格编码
Context Mask = 光照迁移
```

---

## Contract 2

Pre-fill 只负责：

```text
削弱旧衣结构
```

不负责最终生成质量。

---

## Contract 3

第一遍采样：

```text
Generation
```

负责生成。

第二遍：

```text
Harmonization
```

只负责低强度视觉统一。

---

## Contract 4

Harmonization 必须发生：

```text
Crop 内
```

禁止：

```text
第一次 Stitch
→ 整图 img2img
→ 再 Stitch
```

---

## Contract 5

整个流程：

```text
只 Stitch 一次
```

---

## Contract 6

Harmonization 不得修改：

```text
Face
Eyes
Hair 主体
Hat
Manual Protect
```

---

## Contract 7

同一套：

```text
MODEL
CLIP
VAE
```

必须在 Generation 与 Harmonization 中复用。

禁止为了第二遍重新加载同一个 Checkpoint。

---

# 7. Context 的准确含义

Context 的直接作用只有：

> 让 InpaintCropImproved 在确定 Crop 时额外包含某些重要原图区域。

正确理解：

```text
Context Mask
↓
扩大 / 调整 Crop
↓
这些原图像素进入 cropped_image
↓
扩散模型能够看到这些像素
```

---

但：

```text
模型看到
≠
模型一定会正确利用
```

因此必须明确：

> Context 增加可用视觉信息，但不保证肤色、光照或风格自动迁移。

---

# 8. Context 第一版目标

优先让 Crop 包含：

```text
Face
Hair
Neck
Visible Arms
Visible Legs
人物附近背景
```

其中真正重要的不是类别数量，而是：

```text
能够体现原图人物和环境视觉状态的区域
```

---

# 9. Nearby Context 范围

禁止第一版使用过大的：

```text
80–160 px
```

第一轮采用：

```text
24 px
```

第二档：

```text
32 px
```

最大第一阶段建议：

```text
48 px
```

---

测试顺序：

```text
24
→
32
→
48
```

不得直接：

```text
24
→
120
```

---

# 10. Context Mask 组成

建议：

```text
M_CONTEXT_CORE
=
Face
+
Hair
+
Visible Skin
```

人物附近背景：

```text
M_NEAR_BG
```

最终：

```text
M_CONTEXT
=
M_CONTEXT_CORE
+
M_NEAR_BG
```

---

# 11. Context 不允许无限增长

Context 的目的不是：

> 把整张图强行塞进 Crop。

如果最终：

```text
Crop ≈ 整张原图
```

说明 Context 设计失败。

---

# 12. 必须增加 Crop 预览

至少提供：

```text
CONTEXT MASK PREVIEW
```

以及：

```text
FINAL CROP PREVIEW
```

并尽量记录：

```text
Crop Width
Crop Height
```

如果方便，可同时记录：

```text
Crop Area / Original Area
```

---

# 13. Context 面积预警

如果 Crop 已经接近原图：

```text
约 60%–70%+
```

则暂停继续扩大 Context。

该比例只作为：

```text
工程预警值
```

不是绝对验收阈值。

---

# 14. 6GB 环境下 Context 的处理顺序

如果：

```text
Context
↓
Crop 明显增大
↓
OOM
```

必须按照：

```text
① Context 48 → 32
② Context 32 → 24
③ 删除无价值 Context 区域
④ 再考虑 Crop Target 1024 → 896
```

---

禁止第一反应：

```text
拿掉 Context
```

也禁止：

```text
更换低质量基础模型
```

---

# 15. Pre-fill 的目的

当前高 Denoise 的根本原因之一是：

```text
模型必须先忘掉旧衣
```

如果旧衣轮廓仍然存在：

```text
低 Denoise
→
容易继承旧衣
```

所以之前不得不使用：

```text
0.95
```

甚至考虑：

```text
1.00
```

---

但：

```text
高 Denoise
```

也会降低对原图局部特征的继承。

因此新流程改为：

```text
先清旧衣
↓
再降低 Denoise
```

---

# 16. Pre-fill 输入

```text
Cropped Original
+
Cropped Final Edit Mask
```

进入：

```text
Pre-fill
```

---

# 17. Pre-fill 输出要求

Pre-fill 输出只要求：

- 旧衣大轮廓削弱
- 旧裙轮廓削弱
- 吊带等旧结构减少
- 不保留特别强的旧衣纹理

不要求：

- 人体完整
- 解剖完美
- 皮肤漂亮
- 最终可直接使用

---

# 18. Pre-fill 节点优先级

正式定义：

```text
Level A
LaMa
```

如果当前环境：

```text
已经存在可运行 LaMa 节点
+
模型已经存在
```

则优先使用。

---

# 19. LaMa 回退策略

如果本机：

```text
没有 LaMa
```

不得为了这一阶段强制安装大型模型或大量新依赖。

回退：

```text
Level B
Telea
```

如果已有 OpenCV / Telea 类填充节点。

---

仍不可用：

```text
Level C
Blur / Neighbor Fill
```

作为最低成本占位方案。

---

# 20. Pre-fill 方案定位

质量关系：

```text
LaMa
>
Telea
>
Blur
```

但三者都只是：

```text
旧结构清除工具
```

不是：

```text
最终 Inpaint 模型
```

---

# 21. 第一遍：Generation Pass

第一遍的目标：

```text
Generate Correct Content
```

检查：

- 新衣服结构
- 衣服类型
- 人体结构
- 腰身
- 衣服与身体基本关系

---

不要求第一遍直接达到：

```text
完全融入原图
```

---

# 22. Generation 模型

继续使用现有：

```text
WAI Illustrious v1.7
```

以及现有：

```text
Fooocus Inpaint
```

不更换 Checkpoint。

---

# 23. Generation 基础参数

Phase 3 对照：

```text
Steps = 24
CFG = 5
Sampler = euler_ancestral
Scheduler = normal
Denoise = 0.95
```

本阶段先保持作为实验起点。

---

# 24. 降低 Denoise 不能与 Pre-fill 同一个实验

必须分别验证。

禁止：

```text
加 Pre-fill
+
0.95 → 0.78
```

一次完成。

原因：

无法判断改善来自：

```text
Pre-fill
```

还是：

```text
Lower Denoise
```

---

# 25. Denoise 初始目标

当 Pre-fill 验证有效后，再独立测试：

```text
0.95
→
0.78
```

如果需要扫描：

```text
0.70
0.75
0.78
0.82
0.85
```

---

# 26. 第二遍：Harmonization Pass

第二遍只负责：

```text
视觉统一
```

不负责重新设计衣服。

---

# 27. Harmonization 目标

重点改善：

```text
色温
肤色
明度
局部阴影
接触阴影
锐度
纹理
细节密度
边缘软硬
颗粒感
```

最终目标：

> 降低“新内容贴在旧图上的感觉”。

---

# 28. Harmonization 输入

必须使用：

```text
Generation Pass 生成的 Crop Result
```

作为第二遍输入。

不是：

```text
Original Crop
```

也不是：

```text
Stitch 后整图
```

---

# 29. Harmonization 必须发生在 Crop 内

固定流程：

```text
Generation Crop Result
↓
重新 VAE Encode
↓
Harmonization KSampler
↓
Decode
↓
Harmonized Crop
↓
最终 Stitch
```

---

# 30. 只 Stitch 一次

禁止：

```text
Generation
↓
Stitch
↓
第二遍
```

正确：

```text
Generation
↓
Harmonization
↓
Stitch
```

---

# 31. Harmonize Mask

首先：

```text
M_HARMONIZE_RAW
=
Grow(M_FINAL_EDIT)
```

建议起点：

```text
Grow = +8
```

并加入柔化：

```text
Blur = 8~12
```

---

# 32. Harmonization 必须减 Hard Protect

最终：

```text
M_HARMONIZE
=
M_HARMONIZE_RAW
-
M_HARD_PROTECT
-
M_MANUAL_PROTECT
```

---

# 33. Hard Protect 内容

至少包括：

```text
Face
Eyes
Hair 主体
Hat
```

即：

> 第二遍融合也没有资格修改这些区域。

---

# 34. Skin 不全部 Hard Protect

手臂、腿、腰侧皮肤原则上属于：

```text
Soft Protection
```

而不是全部加入：

```text
Hard Protect
```

原因：

Harmonization 有时需要轻微调整：

```text
皮肤边缘色温
衣服接触阴影
局部明度
```

---

但如果某一素材第二遍明显改变裸露皮肤，则可增加：

```text
Skin Core Protect
```

作为实验变量。

---

# 35. Harmonization 参数起点

建议：

```text
Steps = 10
CFG = 4
Denoise = 0.20
Sampler = euler_ancestral
Scheduler = normal
```

这些参数全部标记：

```text
未验证起点
```

---

# 36. Harmonization 扫描

只有正式实验确认第二遍方向有效以后再测试：

```text
0.15
0.20
0.25
```

必要时：

```text
0.12
0.30
```

---

# 37. Denoise 太高的风险

若：

```text
Harmonization Denoise >= 0.35~0.40
```

可能开始重新设计：

```text
衣服
身体
头部边缘
人物结构
```

这违背本阶段目标。

---

# 38. Harmonization Prompt

原则：

```text
不要增加新的服装设计
```

只增加融合约束。

例如：

```text
consistent local lighting,
same local color temperature,
natural contact shadows,
consistent texture,
consistent detail level,
seamless integration with surrounding image
```

---

# 39. 肤色 Prompt

允许辅助：

```text
same skin tone as the visible original skin,
same lighting as the visible arms and legs
```

但必须在文档中说明：

> Prompt 只是语言约束，不是真正的肤色采样或颜色迁移。

---

# 40. Harmonization Negative

可加入：

```text
pasted look,
collage look,
different lighting,
different skin tone,
hard compositing edge,
inconsistent texture,
detached appearance
```

---

# 41. Color Match

继续：

```text
strength = 0
```

保持关闭。

---

# 42. 当前不再扫描 Color Match

禁止继续：

```text
0.3
0.5
0.8
1.0
```

除非未来出现：

> 整个 Crop 存在统一全局色偏。

---

# 43. Stitch

第一阶段继续：

```text
Blend = 16
```

不动。

---

# 44. Stitch 的职责

Stitch 只解决：

```text
空间贴回
+
边缘混合
```

不能解决：

```text
整块新区域颜色错误
纹理不一致
光照方向错误
```

---

# 45. 什么时候允许调 Stitch

只有：

```text
新区域内部已经融合
```

但仍存在：

```text
局部接缝
```

时才测试：

```text
16
→
20
→
24
```

---

# 46. 推荐节点布局

```text
00｜说明
```

↓

```text
01｜Original Image
```

↓

```text
02｜Phase 3 Final Edit Mask
```

↓

```text
03｜Context Mask
```

↓

```text
04｜Context / Crop Preview
```

↓

```text
05｜Inpaint Crop
```

↓

```text
06｜Pre-fill
```

↓

```text
07｜Generation Conditioning
```

↓

```text
08｜Generation Pass
```

↓

```text
09｜Generation Preview
```

↓

```text
10｜Harmonize Mask
```

↓

```text
11｜Hard Protect Subtract
```

↓

```text
12｜Harmonization Conditioning
```

↓

```text
13｜Harmonization Pass
```

↓

```text
14｜Harmonization Preview
```

↓

```text
15｜Final Stitch
```

↓

```text
16｜Final Output
```

---

# 47. 必须提供的 Preview

至少包括：

```text
① FINAL EDIT MASK
```

```text
② CONTEXT MASK
```

```text
③ FINAL CROP
```

```text
④ PRE-FILL RESULT
```

```text
⑤ GENERATION RESULT
```

```text
⑥ HARMONIZATION MASK
```

```text
⑦ HARMONIZED CROP
```

```text
⑧ FINAL STITCH RESULT
```

---

# 48. 实验路线

Phase 3 对照已经建立。

因此下一轮正式开始：

---

## EXP-009 — Context Only

基于：

```text
Phase 3
```

只新增：

```text
Context Mask
→ optional_context_mask
```

保持：

```text
Pre-fill = OFF
Denoise = 0.95
Harmonization = OFF
```

---

实验问题：

> 让 Crop 明确包含人物和环境上下文后，生成结果是否有所改善？

---

重点观察：

```text
肤色
光照
阴影方向
整体协调
```

---

# 49. EXP-010 — Pre-fill Only

基于：

```text
EXP-009
```

只增加：

```text
Pre-fill
```

Generation：

```text
Denoise 继续 = 0.95
```

Harmonization：

```text
OFF
```

---

实验问题：

> Pre-fill 是否能明显减少旧衣结构残留？

---

不以最终融合效果作为主要评价。

---

# 50. EXP-011 — Lower Generation Denoise

基于：

```text
EXP-010
```

仅：

```text
Generation Denoise
0.95 → 0.78
```

---

实验问题：

> 在旧衣已经预清理以后，是否可以降低生成破坏性并保留更多原图视觉特征？

---

# 51. EXP-012 — Harmonization Pass

基于：

```text
EXP-011
```

加入：

```text
Harmonization Pass
```

参数：

```text
Steps = 10
CFG = 4
Denoise = 0.20
```

---

实验问题：

> 第二遍低强度局部扩散能否明显降低 P13 的不同图层感？

---

# 52. EXP-013 — Harmonization Strength Scan

只有：

```text
EXP-012
```

确认第二遍确实有效以后才做。

扫描：

```text
0.15
0.20
0.25
```

其他参数固定。

---

# 53. 实验链

最终必须形成：

```text
Phase 3 Reference
        ↓
EXP-009 Context
        ↓
EXP-010 Pre-fill
        ↓
EXP-011 Lower Denoise
        ↓
EXP-012 Harmonization
        ↓
EXP-013 Strength Scan
```

---

# 54. 问题定位规则

## 旧衣仍存在

检查：

```text
Final Edit Mask
Pre-fill
Generation Denoise
```

不要调 Harmonization。

---

## 新区域整体偏冷白

检查：

```text
Context
Generation
Harmonization
```

---

## 脸或头发被第二遍改变

立即检查：

```text
M_HARMONIZE
-
M_HARD_PROTECT
```

不得通过降低所有 Harmonization 强度掩盖 Mask 错误。

---

## Crop 过大 / OOM

顺序：

```text
Context 48 → 32
↓
Context 32 → 24
↓
减少无关 Context
↓
Target 1024 → 896
```

---

## 第二遍重新改变衣服

说明：

```text
Harmonization Denoise 太高
```

优先：

```text
0.20 → 0.15
```

---

# 55. 6GB 显存开发规则

必须：

```text
Batch = 1
```

---

禁止重复加载：

```text
Checkpoint
VAE
CLIP
```

---

Segmentation 如果已经完成并能释放显存，则不要为了 Preview 保持不必要的大模型常驻。

---

Pre-fill 优先 CPU 时：

```text
允许 CPU
```

---

出现 OOM：

```text
先处理 Context / Crop
```

不是先删掉整个设计核心。

---

# 56. P13 验收

至少比较：

```text
Phase 3 Reference
vs
EXP-009
vs
EXP-010
vs
EXP-011
vs
EXP-012
```

---

从以下方面评分。

### A. 色温

```text
新旧区域是否属于同一个环境光？
```

### B. 肤色

```text
新皮肤与可见手臂 / 腿是否明显色差？
```

### C. 纹理

```text
新区域是否明显更光滑或更锐利？
```

### D. 光照

```text
阴影方向是否与原图一致？
```

### E. 接触关系

```text
衣服是否像真正穿在人身上？
```

### F. 图层感

核心判断：

> 第一眼是否仍然明显看出某一块是后来重新生成的？

---

# 57. 第一阶段成功标准

相比 Phase 3：

- [ ] 新生成区域冷白感明显降低
- [ ] 肤色色差明显降低
- [ ] 衣服与皮肤的接触更自然
- [ ] 新旧区域纹理差异减小
- [ ] 锐度差异减小
- [ ] 明显剪贴感减小
- [ ] 脸和主要头发不被 Harmonization 改动
- [ ] 旧衣不会因为降低 Denoise 再次明显恢复
- [ ] 6GB VRAM 可完成整条流程

---

# 58. 失败判定

如果：

```text
Context 正确
+
Pre-fill 有效
+
Generation Denoise 已降低
+
Harmonization 正常运行
```

但 P13 仍然非常明显，

则停止继续围绕：

```text
Grow
Blur
Stitch
Prompt
```

进行小修小补。

此时应进入下一层问题：

```text
Reference Conditioning
```

或：

```text
Explicit Color / Lighting Transfer
```

---

# 59. 下一阶段预留方向

后续可能研究：

```text
Skin Reference Conditioning
```

```text
Reference IPAdapter
```

```text
Local Color Transfer
```

```text
Relighting
```

```text
Grain / Frequency Matching
```

```text
Depth / Normal Conditioning
```

但这些全部不进入当前实验。

---

# 60. 最终原则

整个工作流必须始终区分：

```text
Mask
→ 哪里允许改
```

```text
Context
→ Crop 需要看到哪里
```

```text
Pre-fill
→ 如何先忘掉旧衣
```

```text
Generation
→ 要生成什么
```

```text
Harmonization
→ 如何让生成内容融入原图
```

```text
Stitch
→ 如何最终贴回
```

任何一个模块都不能被当作另一个模块的替代品。

---

# 61. 本阶段核心链路

最终目标架构：

```text
Phase 3 Final Mask
        ↓
Context-aware Crop
        ↓
Old Clothing Pre-fill
        ↓
Lower-damage Generation
        ↓
Hard-Protected Harmonization
        ↓
Single Final Stitch
        ↓
Integrated Final Image
```

本阶段真正要验证的问题不是：

> 能不能生成新衣服。

而是：

> 能不能让新生成内容看起来原本就属于这张图。
```
