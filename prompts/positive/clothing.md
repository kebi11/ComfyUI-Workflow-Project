# clothing

状态：已写入方案一工作流，未验证。

用途：单独控制服装。换装实验只改这一段，不改 `character-base`。

对应节点：`正向-新服装【每次只改这里】`

```text
wearing a white blouse and a long black skirt, clean clothing folds, accurate garment structure,
natural clothing-body contact, consistent lighting, consistent shadow direction,
consistent color temperature, soft natural garment edges, natural fabric contact and folds
```

## 使用说明

- 目标服装写具体，避免和参考图原服装描述冲突
- 若 IPAdapter 把原衣服带过来，优先调 weight，而不是把服装写得很满来硬盖
- 每次换装保留旧版本，便于对照
