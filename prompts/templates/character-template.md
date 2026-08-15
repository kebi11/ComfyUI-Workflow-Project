# character-template

把稳定模块拼成一次完整正向提示。实际采用的文本以各分文件为准。

```text
{character-base}, {clothing}, {pose_or_composition}, {quality}
```

## 占位说明

| 占位 | 来源 | 实验时 |
|---|---|---|
| `{character-base}` | `../positive/character-base.md` | 身份实验可改；换装实验保持不动 |
| `{clothing}` | `../positive/clothing.md` | 换装实验只改这里 |
| `{pose_or_composition}` | 写在实验文档或单独文件 | 姿势实验只改这里 |
| `{quality}` | `../positive/quality.md` | 默认不动 |

负向默认组合：`base-negative` + `anatomy-negative`。
