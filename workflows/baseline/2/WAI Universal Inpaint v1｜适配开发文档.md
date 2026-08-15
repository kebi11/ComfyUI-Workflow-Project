# WAI Universal Inpaint v1｜适配开发文档

## 一、开发目标

环境：

```text
Checkpoint:
waiIllustriousSDXL_v170.safetensors

GPU:
RTX 4050 Laptop 6GB
```

目标：

> 只修改服装及必要遮挡区域；头部、脸、肤色、身材和无关背景尽可能保持原图。

不再继续把旧方案扩展为：

```text
Generation Mask
+ Composite Mask
+ Seam Mask
+ 多遍 KSampler
+ 多次 Composite
```

新的 baseline 使用：

```text
Crop & Stitch
+
Fooocus Inpaint
```

---

## 二、核心架构

```text
Load Image
│
├── IMAGE
└── MASK
     ↓
✂ Inpaint Crop
│
├── STITCH
├── cropped_image
└── cropped_mask
        │
        ↓
VAE Encode & Inpaint Conditioning
│
├── positive
├── negative
├── latent_inpaint
└── latent_samples
        │
        ├──────────────┐
        ↓              │
Apply Fooocus Inpaint  │
↑                      │
WAI MODEL              │
Fooocus Patch          │
        │              │
        ↓              │
   patched MODEL       │
        │              │
        └────→ KSampler
                 ↑
           latent_samples
                 ↓
             VAEDecode
                 ↓
        Color Match（可选）
                 ↓
          ✂ Inpaint Stitch
                 ↓
             SaveImage
```

---

# 三、需要安装

### 1. Crop & Stitch

ComfyUI Manager 搜索：

```text
ComfyUI-Inpaint-CropAndStitch
```

或者：

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/lquesada/ComfyUI-Inpaint-CropAndStitch.git
```

### 2. Inpaint Nodes

ComfyUI Manager 搜索：

```text
ComfyUI Inpaint Nodes
```

或者：

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/Acly/comfyui-inpaint-nodes.git
```

### 3. Fooocus Inpaint

放入：

```text
ComfyUI/models/inpaint/
```

首选：

```text
fooocus_inpaint_head.pth
inpaint_v26.fooocus.patch
```

---

# 四、Crop & Stitch 参数

第一版建议：

```text
mask_fill_holes = true

mask_expand_pixels = 12

mask_blend_pixels = 16

context_from_mask_extend_factor = 1.7

output_resize_to_target_size = enabled

target size = 1024 × 1024

output_padding = 32
```

Crop & Stitch 作者当前明确建议 SDXL 使用约 `1024×1024` 的目标尺寸，而且 Mask 外区域不会经过 VAE。

### RTX 4050 6GB

第一阶段：

```text
device_mode = CPU
```

让 VRAM 尽量留给：

```text
WAI
Fooocus Patch
KSampler
VAE
```

工作流稳定后再测试 GPU Crop/Stitch。

---

# 五、Mask 规则

这套工作流不需要非常复杂的三层 Mask。

原始 Mask 本身必须正确。

例如：

```text
透明长裙
→
白衬衫 + 黑短裙
```

应该覆盖：

```text
旧衣服完整区域
旧透明裙摆
腰侧透明纱
旧系结
新衣服需要占据的位置
需要重新露出的部分皮肤
少量服装—皮肤接触边界
```

不要默认覆盖：

```text
脸
头发主体
无关手臂
无关腿部
背景大面积区域
```

原则：

> Mask 决定模型有权重建什么。

---

# 六、Fooocus Inpaint 核心

Load Fooocus Inpaint：

```text
head:
fooocus_inpaint_head.pth

patch:
inpaint_v26.fooocus.patch
```

Apply Fooocus Inpaint：

```text
MODEL
← WAI MODEL

patch
← Fooocus Patch

latent
← latent_inpaint
```

输出 MODEL：

```text
→ KSampler MODEL
```

---

# 七、VAE Encode & Inpaint Conditioning

使用 Acly 的：

```text
VAE Encode & Inpaint Conditioning
```

输入：

```text
positive
negative
VAE
cropped_image
cropped_mask
```

输出：

```text
positive
→ KSampler

negative
→ KSampler

latent_inpaint
→ Apply Fooocus Inpaint

latent_samples
→ KSampler latent_image
```

这样就可以同时做到：

```text
Fooocus Inpaint Patch
+
低于 1.0 的 denoise
```

---

# 八、两种运行模式

## Mode A：真正换装

适合：

```text
透明裙 → 黑裙
长袖 → 吊带
长裙 → 短裙
泳装 → 制服
```

建议：

```text
Denoise:
0.85~1.00
```

第一组：

```text
0.90
0.95
1.00
```

旧服装还有残影：

```text
先检查 Mask
↓
再提高 Denoise
```

不要直接增加第二遍工作流。

---

## Mode B：服装小改 / Refine

例如：

```text
黑裙 → 红裙
改领口
改材质
增加图案
改变褶皱
```

建议：

```text
Denoise:
0.50~0.75
```

默认：

```text
0.65
```

---

# 九、KSampler

因为这里同时存在：

- Fooocus Inpaint 官方示例参数；
- WAI 自己的推荐采样习惯；

不要直接假定一个采样器就是最终答案。

## WAI Baseline

先用：

```text
Steps = 24

CFG = 5.0

Sampler =
euler_ancestral

Scheduler =
normal
```

换装：

```text
Denoise = 0.90~0.95
```

---

## 对照实验

再测试：

```text
Steps = 20

CFG = 5

Sampler =
dpmpp_2m_sde_gpu

Scheduler =
karras
```

固定：

```text
Seed
Prompt
Mask
Denoise
```

只比较 sampler。

---

# 十、Prompt

不再用大量：

```text
same face
same person
same body
same hairstyle
...
```

试图锁角色。

人物锁定主要由：

```text
Mask
+
Crop
+
Stitch
```

完成。

Positive 示例：

```text
masterpiece, best quality, amazing quality,

white sleeveless blouse,
high-waisted opaque black pleated skirt,

clean garment silhouette,
natural clothing folds,
natural clothing-body contact,
soft contact shadows,
consistent lighting
```

Negative：

```text
bad quality,
worst quality,
worst detail,

double clothing,
ghost fabric,
floating fabric,
blurry hemline
```

如果目标服装本来就是透明材质，则不要写：

```text
transparent cloth
```

之类的负面词。

---

# 十一、针对当前黑裙案例

当前最大的历史问题是：

```text
原透明纱
→
残留在新黑裙周围
```

第一轮参数：

```text
mask_expand_pixels = 14

mask_blend_pixels = 16

context factor = 1.8

target = 1024×1024

Steps = 24

CFG = 5

Euler a

Denoise = 0.95
```

如果仍然有旧纱：

### 第一步

扩大**实际手绘 Mask**。

尤其检查：

```text
腰左侧
原透明裙摆
原裙边
旧系结
```

### 第二步

再：

```text
0.95 → 1.00
```

不要第一时间做第三遍精修。

---

# 十二、Color Match

默认：

```text
OFF
```

只有出现：

```text
Crop 内明显偏亮
Crop 内明显偏暖
Crop 内明显偏冷
```

再打开。

参考：

```text
reference
← cropped_image

target
← VAEDecode 输出

exclude mask
← cropped_mask
```

然后：

```text
Color Match
→ Inpaint Stitch
```

---

# 十三、最终 Stitch

```text
STITCH
← Inpaint Crop

inpainted_image
← VAEDecode
```

或者：

```text
← Color Match
```

输出直接：

```text
SaveImage
```

第一版不再增加：

```text
ImageCompositeMasked
第二遍整衣 KSampler
Seam 环带
第三遍局部修复
```

---

# 十四、4050 6GB 原则

```text
Batch = 1
```

v1 暂时禁止：

```text
IPAdapter
FaceDetailer
ControlNet
整图二次高清采样
```

只让 SDXL 处理：

```text
Crop 后的 1024 局部图
```

如果 OOM：

```text
第一步：
Crop device → CPU
```

仍然 OOM：

```text
第二步：
target → 896×896
```

但 `1024×1024` 应作为正常 SDXL baseline。

---

# 十五、验证实验

严格按顺序：

### EXP-01

验证单次：

```text
Crop
+
Fooocus
+
Stitch
```

是否已经能够自然换装。

### EXP-02

固定 Seed 测：

```text
Denoise

0.85
0.90
0.95
1.00
```

### EXP-03

固定最佳 denoise：

```text
Mask Expand

8
12
16
20
```

### EXP-04

测试 Blend：

```text
8
12
16
24
```

### EXP-05

Sampler A/B：

```text
Euler a + normal
```

对：

```text
DPM++ 2M SDE GPU + Karras
```

---

# 十六、验收标准

必须满足：

- [ ] 旧衣服完全删除
- [ ] 没有透明纱残影
- [ ] 新衣服结构正常
- [ ] 衣服与皮肤自然接触
- [ ] 无明显 Stitch 接缝
- [ ] 头部没有可见变化
- [ ] 发型主体没有可见变化
- [ ] Mask 外肤色没有可见变化
- [ ] Mask 外身材没有可见变化
- [ ] Mask 外背景没有可见变化
- [ ] 单次 Inpaint 已达到可用质量

---

# 十七、后续扩展

只有这个 baseline 验证通过以后才继续：

```text
方案一
Crop & Stitch
+ Fooocus Inpaint
```

↓

```text
方案二
+ DWPose
+ OpenPose
```

↓

```text
方案三
+ 人物身份控制
```

不要同时开发。

---

# 十八、最终定位

新的：

```text
WAI_Universal_Inpaint_v1
```

应该成为整个项目新的局部编辑 baseline。

旧的：

```text
多 Mask
多 KSampler
多 Composite
```

方案暂时保留为实验备份，而不是继续作为主线开发。