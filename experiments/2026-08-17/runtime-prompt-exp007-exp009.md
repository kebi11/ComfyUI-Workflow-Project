# EXP-007 / EXP-009 真实 runtime Prompt

从成品 PNG 元数据读取，不是仓库 JSON 绿框默认稿。

```text
输入图：    胡.jpg
分辨率：    1872 × 1056
Seed：      114514 / fixed
Checkpoint：waiIllustriousSDXL_v170.safetensors
Denoise：   0.95
Color Match：0
```

正向：

```text
masterpiece, best quality, amazing quality,
nsfw, uncensored, nude,
pale skin, same skin tone, consistent skin tone,
same body type, slim waist, long legs, delicate proportions,
natural anatomy,
consistent lighting
```

负向（按原样，含全角标点）：

```text
：clothes, dress, fabric, ghost fabric（
```

来源：

```text
results/baseline/WAI_autoclothes_p3_00004_.png
results/baseline/WAI_autoclothes_exp009_00002_.png
```

EXP-010 必须沿用这一组，才能和 Phase 3 / EXP-009 对照。
