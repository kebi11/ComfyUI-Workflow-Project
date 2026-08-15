# WAI AutoClothes Phase 3 开发文档

## 1. 文档目的

本阶段在现有：

```text
workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase2.json
```

基础上继续开发。

Phase 3 不重新设计采样主干，不更换基础模型，不同时解决肤色、光照、姿势和多人问题。

本阶段只解决一个核心问题：

> 将当前单一 Clothes Mask 拆分为“需要删除的衣服区域”和“必须保护的人体区域”，再通过 Mask 运算得到最终可编辑区域。

最终解决或缓解：

- P01：腰身跟随旧衣服剪影变化
- P06：吊带、旧衣边、薄纱残留
- P07：自动衣服 Mask 不够通用
- P10：不同衣服类别需要不同检测组合
- P11：长发遮挡衣服时误伤头发

暂不解决：

- P08：新生成皮肤无法匹配原图光照
- P09：当前 Color Match 对 P08 无效
- P12：多人场景

---

# 2. 开发原则

## 2.1 Phase 3 只修改 Mask 系统

必须保持以下内容不变：

```text
Checkpoint
Fooocus Inpaint
Sampler
Scheduler
Steps
CFG
Denoise
Crop 参数
Stitch 主干
Seed
基础 Prompt
```

Phase 3 第一版沿用当前 Phase 2 参数。

例如：

```text
Checkpoint:
waiIllustriousSDXL_v170.safetensors

KSampler:
Seed = 114514
Steps = 24
CFG = 5
Sampler = euler_ancestral
Scheduler = normal
Denoise = 0.95
```

以及：

```text
Inpaint Crop:
expand = 14
blend = 16
context = 1.8
target = 1024
device = CPU
```

禁止在同一次 Phase 3 实验中同时修改这些参数。

---

# 3. 新工作流

从：

```text
workflows/experiments/
WAI_AutoClothes_Inpaint_v1-phase2.json
```

复制为：

```text
workflows/experiments/
WAI_AutoClothes_Inpaint_v1-phase3.json
```

同时建立说明文件：

```text
workflows/experiments/
WAI_AutoClothes_Inpaint_v1-phase3.md
```

不要覆盖 Phase 2。

---

# 4. Phase 3 核心思想

当前 Phase 2：

```text
原图
↓
Segformer Clothes
↓
Grow / Blur
↓
直接作为 Inpaint Mask
↓
Crop
↓
Fooocus
↓
KSampler
↓
Stitch
```

Phase 3 改成：

```text
                         ┌─ 衣服分割 ─────────────┐
                         │                       │
原图 ─→ Segformer Model ┤                       ├─ Mask 运算
                         │                       │
                         └─ 人体保护分割 ─────────┘
                                     ↑
                              Manual Override
                                     ↓
                            Final Edit Mask
                                     ↓
                              Inpaint Crop
                                     ↓
                           Fooocus + WAI
                                     ↓
                                Stitch
```

核心原则：

> 衣服 Mask 负责“删什么”。

> Protection Mask 负责“什么绝对不能动”。

两者不得继续由同一个 Grow 参数承担。

---

# 5. Phase 3 Mask 数据模型

整个工作流统一使用以下 Mask 名称。

```text
M_GARMENT_RAW
```

自动检测到的原始衣服 Mask。

```text
M_GARMENT_EXPANDED
```

经过 Grow / Blur 后的衣服删除区域。

```text
M_MANUAL_ADD
```

人工补充的需要删除区域。

```text
M_HARD_PROTECT
```

脸、头发、帽子等绝对保护区域。

```text
M_SKIN_CORE
```

手臂、腿等裸露皮肤主体保护区。

```text
M_MANUAL_PROTECT
```

人工指定的绝对保护区域。

```text
M_REMOVE
```

最终准备删除的区域。

```text
M_PROTECT
```

最终人体保护区域。

```text
M_FINAL_EDIT
```

最终真正进入 Inpaint Crop 的 Mask。

---

# 6. 最终 Mask 公式

Phase 3 的正式公式为：

```text
M_REMOVE
=
M_GARMENT_EXPANDED
+
M_MANUAL_ADD
```

保护区：

```text
M_PROTECT
=
M_HARD_PROTECT
+
M_SKIN_CORE
+
M_MANUAL_PROTECT
```

最终：

```text
M_FINAL_EDIT
=
M_REMOVE
-
M_PROTECT
```

即：

```text
Final Edit Mask
=
(Auto Clothes + Manual Add)
-
(Hard Protect + Skin Core + Manual Protect)
```

最终只有：

```text
M_FINAL_EDIT
```

允许进入：

```text
InpaintCropImproved.mask
```

---

# 7. Segformer 架构

Phase 3 推荐升级为：

```text
LoadSegformerModel
       ↓
Segformer Model
       │
 ┌─────┴─────┐
 │           │
 ↓           ↓
Garment    Protection
Setting    Setting
 │           │
 ↓           ↓
Segformer  Segformer
Ultra V3   Ultra V3
```

目的：

- 同一个 Segformer 模型只加载一次
- 衣服识别和人体保护共享模型
- 减少显存浪费
- 后续更容易增加类别 Preset

如果当前本机 LayerStyle 版本不具备这一结构，则允许第一版继续使用现有 Segformer 节点实现逻辑验证。

但必须保证：

> Phase 3 逻辑先成立，再考虑节点版本优化。

---

# 8. Garment Mask 分支

## 8.1 职责

只回答：

> 原来的衣服在哪里？

不得负责人物保护。

---

## 8.2 第一版类别

继续沿用 Phase 2：

```text
dress = ON
upper_clothes = ON
skirt = ON
belt = ON
```

其他默认关闭。

---

## 8.3 Grow

Phase 3 第一轮建议：

```text
Grow = 4
Blur = 4
```

但注意：

> Grow=4 是实验参数，不是长期固定值。

必须通过 Mask Preview 判断。

---

## 8.4 验收原则

衣服 Mask 应做到：

```text
旧衣主体全部覆盖
+
旧衣边缘尽量覆盖
+
吊带 / 腰带 / 衣领等不要漏
```

同时：

```text
尽量不要大量侵入裸露皮肤
```

但裸露皮肤保护最终由 Protection 分支负责，因此 Garment Mask 可以比 Phase 2 更积极。

---

# 9. Hard Protect 分支

## 9.1 保护对象

第一版至少包括：

```text
face
hair
hat
```

根据当前素材可扩展：

```text
shoe
bag
glasses
```

原则：

> 用户没有要求修改的高优先级区域，应优先进入 Hard Protect。

---

## 9.2 Grow

建议：

```text
Grow = +1 ~ +2
Blur = 1 ~ 2
```

目的：

稍微扩大保护边界。

---

## 9.3 Hard Protect 的性质

Hard Protect 是：

```text
绝对不可修改
```

例如：

```text
脸
眼睛
主要发型
草帽
```

哪怕因此减少一点衣服边界的生成自由度，也优先保护。

---

# 10. Skin Core Protect

裸露皮肤不能完全采用 Hard Protect 的处理方式。

原因：

衣服与皮肤接触处通常正是：

```text
吊带
领口
袖口
腰边
裙边
```

最容易产生残影的位置。

如果裸露皮肤整个区域全部强保护，旧衣边缘可能无法被清除。

---

## 10.1 第一版

检测：

```text
arms
legs
```

如果当前 Segformer 能稳定识别身体其他裸露皮肤，可后续扩展。

---

## 10.2 收缩保护区

建议：

```text
Grow = -2
Blur = 1
```

即把皮肤 Mask 向内部收缩。

形成：

```text
Skin Full Mask
↓
Shrink
↓
Skin Core Protect
```

这样：

```text
皮肤内部
→ 保护

皮肤 / 衣服交界
→ 留给 Inpaint 重建
```

---

# 11. Manual Add Mask

必须保留一个人工补充入口。

文件可以暂时使用：

```text
input/manual_add_mask.png
```

正式仓库参考素材可放：

```text
references/clothing/
```

---

## 11.1 用途

Manual Add 专门补：

```text
细吊带
透明薄纱
遗漏衣袖
腰带
领口
围巾
Segformer 漏掉的衣服边缘
```

---

## 11.2 运算

```text
M_GARMENT_EXPANDED
+
M_MANUAL_ADD
=
M_REMOVE
```

不要为了补一根吊带去：

```text
Grow 4 → 10
```

---

# 12. Manual Protect Mask

再增加：

```text
input/manual_protect_mask.png
```

用途：

```text
误识别的头发
手指
裸露腰侧
腿部
饰品
帽子
特殊人物区域
```

公式：

```text
M_PROTECT
=
M_AUTO_PROTECT
+
M_MANUAL_PROTECT
```

---

# 13. 自动与手动的关系

Phase 3 不采用：

```text
自动 Mask
OR
手工 Mask
```

而采用：

```text
Auto
+
Manual Add
-
Manual Protect
```

即：

```text
自动完成主体工作
+
人工只修正错误
```

目标是：

> 95% 自动 + 5% 快速修正。

---

# 14. Mask 运算节点

优先使用现有 ComfyUI / LayerStyle 中稳定可用的：

```text
MaskComposite
```

实现：

```text
ADD
SUBTRACT
```

如果 LayerStyle 已存在更稳定的 Mask Math 节点也可以使用，但不得为了简单加减运算额外引入复杂依赖。

---

# 15. 必须增加的 Preview

Phase 3 必须至少提供四个明显的 Preview。

## Preview 1

标题：

```text
① AUTO GARMENT MASK
```

显示：

```text
M_GARMENT_EXPANDED
```

---

## Preview 2

标题：

```text
② AUTO PROTECTION MASK
```

显示：

```text
M_HARD_PROTECT + M_SKIN_CORE
```

---

## Preview 3

标题：

```text
③ MANUAL OVERRIDE
```

展示：

```text
M_MANUAL_ADD
M_MANUAL_PROTECT
```

可以两个 Preview。

---

## Preview 4

最重要：

```text
④ FINAL EDIT MASK
```

显示：

```text
M_FINAL_EDIT
```

这个 Preview 必须放在：

```text
Inpaint Crop
```

之前。

---

# 16. 强制调试规则

在 Phase 3 中：

> Final Edit Mask 错误时，禁止调整采样参数。

错误排查顺序必须是：

```text
Final Edit Mask
↓
Garment Mask
↓
Protection Mask
↓
Manual Override
↓
确认 Mask 正确
↓
才允许 Queue
```

禁止：

```text
Mask 错
→ 调 CFG
→ 调 Steps
→ 调 Prompt
→ 调 Sampler
```

---

# 17. 衣服 Preset 体系

Phase 3 开始引入衣服类型 Preset。

第一版不要求复杂 UI，可以先通过 Note 节点和类别开关实现。

---

## Preset A：Dress

```text
dress
skirt
belt
```

根据具体图可开启 upper_clothes。

---

## Preset B：Top + Skirt

```text
upper_clothes
skirt
belt
```

---

## Preset C：Pants / Shorts

```text
upper_clothes
pants
belt
```

---

## Preset D：Special

```text
自动类别
+
Manual Add
```

适合：

```text
透明薄纱
复杂礼服
异形服装
细吊带
特殊配饰
```

---

# 18. P11：长发覆盖衣服

若：

```text
Clothes Mask
```

与：

```text
Hair Mask
```

发生重叠：

必须以 Hair Protect 为高优先级。

公式天然保证：

```text
Final Edit
=
Clothes
-
Hair
```

因此：

```text
头发优先保留
```

而不是随衣服一起重新生成。

---

# 19. Phase 3 不解决 P08 / P09

Phase 3 保持：

```text
Color Match strength = 0
```

不继续调整。

P08：

```text
新生成皮肤与原图日落暖光不一致
```

应单独实验。

Phase 3 不增加：

```text
肤色迁移
肤色匹配
光照匹配
```

节点。

---

# 20. Phase 3 不解决 P12

当前版本继续明确：

```text
Single Person Only
```

双人、多人图暂不支持。

多人架构未来应先做：

```text
Person Instance Detection
↓
Target Person Selection
↓
Person ROI
↓
Clothes Segmentation
```

之后再接 Phase 3。

不得在本阶段加入。

---

# 21. Phase 3A 节点结构

建议按照以下分组排列。

```text
00｜说明 / Preset
```

↓

```text
01｜输入
- 原图
- Manual Add Mask
- Manual Protect Mask
```

↓

```text
02｜Segformer Shared Model
```

↙　　　　　　　　　↘

```text
03｜Garment Detection
```

```text
04｜Protection Detection
```

↓

```text
05｜Mask Processing
- Garment Grow
- Hard Protect Grow
- Skin Shrink
```

↓

```text
06｜Manual Override
```

↓

```text
07｜Mask Math
```

↓

```text
08｜FINAL EDIT MASK PREVIEW
```

↓

```text
09｜Inpaint Crop
```

↓

```text
10｜Fooocus Inpaint
```

↓

```text
11｜KSampler
```

↓

```text
12｜VAE Decode
```

↓

```text
13｜Stitch
```

↓

```text
14｜Final Output
```

---

# 22. 工作流逻辑图

```text
                          ┌───────────────────────┐
                          │ Load Original Image   │
                          └──────────┬────────────┘
                                     │
                     ┌───────────────┴───────────────┐
                     │                               │
                     ▼                               ▼
            Clothes Segmentation           Protection Segmentation
                     │                               │
                     ▼                               ├─────────────┐
              Garment Raw Mask                      │             │
                     │                              Face/Hair      Skin
                     ▼                               │             │
              Grow 4 / Blur 4                        ▼             ▼
                     │                         Hard Protect     Shrink -2
                     ▼                               │             │
         M_GARMENT_EXPANDED                          └──────┬──────┘
                     │                                      │
            ┌────────┴────────┐                             ▼
            │                 │                       Auto Protect
            ▼                 ▼                             │
      Manual Add          Garment                           │
            │                 │                             │
            └────── ADD ──────┘                             │
                     │                                      │
                     ▼                                      │
                 M_REMOVE                                   │
                                                            │
                                                 Manual Protect
                                                            │
                                                     ADD ────┘
                                                            │
                                                            ▼
                                                       M_PROTECT
                                                            │
                     ┌──────────────────────────────────────┘
                     ▼
              REMOVE - PROTECT
                     │
                     ▼
                M_FINAL_EDIT
                     │
                     ▼
             FINAL MASK PREVIEW
                     │
                     ▼
             InpaintCropImproved
                     │
                     ▼
              Fooocus Inpaint
                     │
                     ▼
                  KSampler
                     │
                     ▼
                VAE Decode
                     │
                     ▼
            InpaintStitchImproved
                     │
                     ▼
                 Final Image
```

---

# 23. Phase 3A 初始参数

第一轮固定：

```text
Garment Grow:
4

Garment Blur:
4
```

```text
Hard Protect Grow:
+2

Hard Protect Blur:
1
```

```text
Skin Core Shrink:
-2

Skin Blur:
1
```

其他全部沿用 Phase 2。

---

# 24. Phase 3 验收标准

必须先看 Mask，再看生成结果。

---

## Mask 验收

### Garment Mask

- [ ] 原衣主体完全覆盖
- [ ] 原衣边缘基本覆盖
- [ ] 吊带没有明显漏检
- [ ] 薄纱主体尽量覆盖
- [ ] 腰带覆盖

### Protection

- [ ] 脸完整保护
- [ ] 眼睛完整保护
- [ ] 头发主体完整保护
- [ ] 草帽完整保护
- [ ] 手臂主体保护
- [ ] 腿主体保护

### Final Edit Mask

- [ ] 不包含脸
- [ ] 不包含眼睛
- [ ] 不明显包含头发
- [ ] 不明显包含草帽
- [ ] 能覆盖旧衣
- [ ] 能覆盖衣服与皮肤接触边
- [ ] 不大量吞掉正常裸露皮肤

---

# 25. 成品验收

Queue 后检查：

- [ ] 人脸无可见变化
- [ ] 主要发型无可见变化
- [ ] 草帽无明显变化
- [ ] 已露出手臂基本保持
- [ ] 已露出腿部基本保持
- [ ] 腰宽不明显跟随旧裙变化
- [ ] 腿长视觉比例不明显变化
- [ ] 旧吊带明显减少
- [ ] 薄纱残影明显减少
- [ ] 旧衣边缘不明显残留
- [ ] 背景基本保持
- [ ] 6 GB VRAM 可稳定运行

---

# 26. P01 判断规则

如果腰身仍跟随旧衣：

首先检查：

```text
FINAL EDIT MASK
```

判断：

### 情况 A

Mask 吃掉大量正确腰侧皮肤：

```text
Grow 4
↓
尝试 3
```

或加强 Skin Protect。

### 情况 B

旧裙轮廓仍在 Mask 外：

不能继续减 Grow。

应该：

```text
Manual Add
```

补掉旧裙边。

### 情况 C

Mask 已正确，但生成仍形成错误腰型：

此时才认定：

> P01 已从 Mask 问题转为生成模型 / Conditioning 问题。

不要提前加 IPAdapter。

---

# 27. P06 判断规则

如果旧衣残影存在：

顺序：

```text
① Final Mask 是否覆盖残影位置？
```

否：

```text
补 Manual Add
```

是：

```text
② denoise 0.95 是否仍保留旧衣信息？
```

若是：

下一独立 EXP 才允许：

```text
0.95 → 1.00
```

不得在 Phase 3A 同时修改。

---

# 28. Phase 3B：双 Mask 架构

Phase 3A 验证通过后，再考虑 Phase 3B。

Phase 3B 将：

```text
Generation Mask
```

和：

```text
Composite Mask
```

分离。

---

## Composite Mask

严格控制：

> 最终哪些像素允许贴回原图。

使用：

```text
M_FINAL_EDIT
```

---

## Generation Mask

允许比最终贴回区域稍大：

```text
M_FINAL_EDIT
↓
Grow +4
↓
Blur 4
↓
Generation Mask
```

模型获得更大的边缘生成空间。

最终 Stitch 仍只使用 Composite 范围。

---

## Phase 3B 目的

减少：

```text
衣服边缘接缝
硬边
新旧皮肤交界突兀
```

但不得在 Phase 3A 首次验证时加入。

---

# 29. 推荐实验安排

## EXP-006

对应：

```text
P08
```

只修改绿色 Prompt。

不得修改 Phase 3 Mask。

---

## EXP-007

对应：

```text
P01
P07
P10
P11
```

建立：

```text
WAI_AutoClothes_Inpaint_v1-phase3.json
```

只增加：

```text
Garment Mask
Hard Protect
Skin Core Protect
Manual Add
Manual Protect
Final Mask Math
```

Denoise 保持：

```text
0.95
```

---

## EXP-008

对应：

```text
P06
```

基于 EXP-007 已确认 Mask 正确的版本。

仅修改：

```text
Denoise:
0.95
→
1.00
```

验证吊带、薄纱和旧衣残影。

---

# 30. Phase 3 失败回退

如果工作流出现异常：

```text
OOM
→ 暂时只保留一组 Segformer
→ Protection 使用已有 Mask 对照
→ 不先降低生成质量
```

如果：

```text
脸被改
```

检查：

```text
Face Protect
→ Final Edit Mask
```

如果：

```text
头发被改
```

检查：

```text
Hair Protect 是否正确从 Clothes Mask 中减去
```

如果：

```text
旧衣残影
```

检查：

```text
Manual Add
```

而不是先加第二遍采样。

如果：

```text
腰型变化
```

先检查：

```text
Final Edit Mask 是否吞腰
```

不得直接增加 IPAdapter。

---

# 31. 工作流中的 Note 节点

在 Phase 3 JSON 顶部增加 Note：

```text
AutoClothes Phase 3｜REMOVE / PROTECT / FINAL MASK

目标：
把“删除旧衣”和“保护人物”彻底拆开。

核心公式：

FINAL EDIT MASK
=
(AUTO CLOTHES + MANUAL ADD)
-
(HARD PROTECT + SKIN CORE + MANUAL PROTECT)

调试规则：
1. 先看 Final Edit Mask
2. Mask 错时禁止调 CFG / Steps / Sampler / Denoise
3. 漏衣服使用 Manual Add
4. 误伤人物使用 Manual Protect
5. Phase 3A Denoise 固定 0.95
6. Color Match 保持 0
7. Single Person Only
```

---

# 32. 文档同步

完成 Phase 3 后必须同步：

```text
docs/节点说明.md
docs/参数说明.md
docs/已知问题.md
```

以及对应：

```text
experiments/YYYY-MM-DD/EXP-007.md
```

---

## `docs/已知问题.md`

更新：

```text
P01
P06
P07
P10
P11
```

并明确：

```text
Phase 3 是否改善
```

---

## `docs/参数说明.md`

只有实际跑过后才能把：

```text
Grow 4
Protect +2
Skin -2
```

标记为：

```text
观察中
```

不得在未测试情况下标记：

```text
已验证
```

---

# 33. Phase 3 完成条件

只有满足以下条件才认为 Phase 3A 完成：

```text
Final Edit Mask 可解释
+
衣服和人体职责分离
+
Manual Add 可用
+
Manual Protect 可用
+
Face/Hair Protection 可用
+
至少一张实际图片 Queue 成功
+
6GB VRAM 可运行
+
实验记录完整
```

之后才能考虑：

```text
Phase 3B
```

---

# 34. 最终架构原则

Phase 3 完成后，整个 AutoClothes 主线应遵循：

```text
检测衣服
≠
决定人物哪里能改
```

而是：

```text
检测衣服
↓
REMOVE

检测人物
↓
PROTECT

REMOVE - PROTECT
↓
FINAL EDIT
```

以后所有衣服识别、人体保护、手工修正和类别 Preset 都只影响：

```text
FINAL EDIT MASK
```

生成主干保持稳定。

这是 Phase 3 的核心设计目标。