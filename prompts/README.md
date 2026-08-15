# prompts/

不要把重要 Prompt 长期只放在工作流节点里。稳定文本独立存放，方便对比、复用和版本管理。

```text
prompts/
├─ positive/     正向提示
├─ negative/     负向提示
└─ templates/    可拼装模板
```

## 使用约定

- 工作流节点里的 Prompt 应能追溯到本目录中的某个文件
- 已验证 Prompt 不要原地覆盖，新增 `*-v02.md` 或在文件内追加版本节
- 每次 Prompt 实验仍要写 `EXP-XXX.md`，并固定其他变量
- 中英混合可以，但同一轮对照里语言结构保持一致

## 当前草稿

以下文件是起点草稿，**尚未随工作流验证**：

- `positive/character-base.md`
- `positive/clothing.md`
- `positive/quality.md`
- `negative/base-negative.md`
- `negative/anatomy-negative.md`
- `positive/clothing-undress-v1.md`（AutoClothes Phase 1）
- `negative/undress-negative-v1.md`（AutoClothes Phase 1）
- `templates/character-template.md`
