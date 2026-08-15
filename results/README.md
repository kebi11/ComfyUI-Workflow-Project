# results/

生成结果不要和 `references/` 里的参考图混放。

```text
results/
├─ baseline/       基线出图
├─ experiments/    实验出图
└─ comparison/     A/B、参数对比、候选图
```

## 命名建议

```text
{工作流名}_{seed}_{短说明}.png
{EXP号}_{A或B}_{短说明}.png
```

例如：`character-ipadapter-v02_12345_weight075.png`。

大图默认不进 Git（见根目录 `.gitignore`）。需要留档时，把少量关键对比图单独提交，或在实验文档里写清本机路径。
