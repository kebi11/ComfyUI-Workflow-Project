# workflows/archive

保存已淘汰、旧架构、不再使用但可能仍有参考价值的工作流。

失败方案不要立刻删除。归档时在下方登记：

```text
文件：
原路径：
淘汰原因：
仍有价值的点：
对应实验：
```

当前归档：

```text
文件：     gu_keep_character.json
原路径：   ComfyUI/user/default/workflows/gu_keep_character.json
淘汰原因： 整图两遍 Img2Img，无法做到 MASK 外像素锁定；方案一已替换为局部 Inpaint
仍有价值的点： WAI v1.7 采样习惯、800×1216、顾.jpg 人物描述、两遍采样骨架
对应实验： 方案一基线由此派生
```
