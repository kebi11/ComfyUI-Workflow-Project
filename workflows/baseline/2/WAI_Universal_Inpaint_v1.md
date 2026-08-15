# WAI_Universal_Inpaint_v1

对应 JSON：`WAI_Universal_Inpaint_v1.json`  
设计说明书：`WAI Universal Inpaint v1｜适配开发文档.md`  
状态：**已建成，未实测**。装好节点和 Fooocus 权重后需要重启 ComfyUI。

## 仓库来源（作者原仓库，不用旧镜像）

| 组件 | 地址 | 本机路径 |
|---|---|---|
| Crop & Stitch | https://github.com/lquesada/ComfyUI-Inpaint-CropAndStitch | `ComfyUI/custom_nodes/ComfyUI-Inpaint-CropAndStitch` |
| Inpaint Nodes | https://github.com/Acly/comfyui-inpaint-nodes | `ComfyUI/custom_nodes/comfyui-inpaint-nodes` |
| Fooocus 模型 | https://huggingface.co/lllyasviel/fooocus_inpaint | `ComfyUI/models/inpaint/` |

节点类型按当前作者代码：

- `InpaintCropImproved` / `InpaintStitchImproved`
- `INPAINT_LoadFooocusInpaint`
- `INPAINT_ApplyFooocusInpaint`
- `INPAINT_VAEEncodeInpaintConditioning`（才能 denoise < 1.0）
- `INPAINT_ColorMatch`（默认 strength=0，关闭）

## 数据流

```text
LoadImage + 一张服装 Mask
    ↓
Inpaint Crop（扩 14 / blend 16 / context 1.8 / 1024 / CPU）
    ↓
VAE Encode & Inpaint Conditioning
    ├─ latent_inpaint → Apply Fooocus Inpaint
    └─ latent_samples → KSampler
    ↓
KSampler 24 / CFG 5 / euler_ancestral / denoise 0.95
    ↓
Decode → Color Match(0) → Inpaint Stitch
    ↓
WAI_universal_inpaint_v1
```

Mask 外不进 VAE。第一版不加第二遍、Seam、整图 Composite。

## 当前黑裙建议

- 手绘 Mask 必须盖住旧透明纱、旧裙摆、旧系结
- denoise `0.95`，还有残影先加大 Mask，再试 `1.00`
- 小改/换色用 Mode B：`0.65`

## 6GB

Crop `device_mode = cpu (compatible)`。仍 OOM 把 target 改成 `896×896`。不要叠 IPAdapter / ControlNet / 整图高清。

## 对照实验（文档第十五节）

1. 单次 Crop + Fooocus + Stitch 能不能换装
2. Denoise 0.85 / 0.90 / 0.95 / 1.00
3. Mask Expand 8 / 12 / 16 / 20
4. Blend 8 / 12 / 16 / 24
5. Euler a+normal 对 DPM++ 2M SDE GPU + Karras
