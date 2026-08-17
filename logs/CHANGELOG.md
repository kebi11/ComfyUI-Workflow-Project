# CHANGELOG

记录项目级重要变化，不是每一张出图。每次架构、基线、正式版或研究方向变化时追加。

## v0.9 — 2026-08-17

AutoClothes 进入视觉融合阶段，但第一张实验图只做 Small Context。

目的：
解决 P13（新区域像另一图层）。P08 ⊂ P13。Context 只扩大 Crop，不是肤色/风格编码器。

结果：
- 新增设计文档与补丁（开工 Gate、Contract 8/9、EXP-009 验收）
- 从 Phase 3 派生 `workflows/experiments/WAI_AutoClothes_Inpaint_v1-exp009.json`
- 未覆盖 Phase 3；未加入 Pre-fill / 降 Denoise / Harmonization
- P13 写入 `docs/已知问题.md`
- JSON 连线校验 0 错误，尚未 Queue

下一步：
同一输入对照 Phase 3 后 Queue EXP-009。先看 CONTEXT MASK 与 FINAL CROP，再评融合。通过后才做 EXP-010。

## v0.8 — 2026-08-16

AutoClothes Phase 3A：分层 Mask + 人体保护 + 手工覆盖。

目的：
把“删旧衣”和“保护人物”拆开。Final Edit = (衣服 + Manual Add) − (硬保护 + 皮肤核 + Manual Protect)。

结果：
- 新增 `workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase3.json`
- 未覆盖 Phase 2
- 对应 `experiments/2026-08-16/EXP-007.md`
- 采样 / Prompt / Crop / Color Match=0 未改
- 图已建成，尚未 Queue

下一步：
先看 ④ FINAL EDIT MASK，再和 Phase 2 对照腰身与衣缘。

## v0.7 — 2026-08-15

AutoClothes Phase 2：Segformer 自动认衣。

目的：
按 `baseline/3` 文档，用 SegformerB2ClothesUltra 替换手工 Mask，作为自动找衣服边界的第一层。

结果：
- 克隆 `ComfyUI_LayerStyle` 到本机 custom_nodes
- 补齐 LayerStyle Python 依赖（未动 torch）
- 下载 `segformer_b2_clothes` 到 `models/segformer_b2_clothes`
- 新增 `workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase2.json`
- 对应 `experiments/2026-08-15/EXP-005.md`
- 采样与 Phase 1 换装 Prompt 未改
- 需重启 ComfyUI 后 Queue，尚未出图

下一步：
先看自动 Mask 是否盖住薄纱且不吃脸，再决定 Phase 3 保护区。

## v0.6 — 2026-08-15

AutoClothes Phase 1：在 Universal Inpaint 上派生换装实验，不覆盖 baseline。

目的：
先验证「服装 Mask + Crop + Fooocus」能在锁脸、锁已露出皮肤和身材的前提下更换薄纱裙。自动认衣留到 Phase 2。

结果：
- 补齐 `input/顾.jpg` 别名；正式用 `gu_ref.jpg`
- 从旧 clipspace 导出 `gu_clothes_mask_hand_v1.png`
- 新增 `workflows/experiments/WAI_AutoClothes_Inpaint_v1-phase1.json`
- 对应 `experiments/2026-08-15/EXP-004.md`
- Fooocus patch 体积约 1.23GB，视为已下完
- ComfyUI 当时未启动，尚未出图

下一步：
本机 Queue EXP-004。通过后再装 LayerStyle / Segformer。

## v0.5 — 2026-08-15

新主线：WAI Universal Inpaint v1。

目的：
用作者维护的 Crop & Stitch + Fooocus Inpaint 替代整图多 Mask 方案，Mask 外不进 VAE。

结果：
已克隆 lquesada / Acly 原仓库，建成 `workflows/baseline/2/WAI_Universal_Inpaint_v1.json`。方案一 v1–v3 保留为实验备份。

下一步：
确认 `inpaint_v26.fooocus.patch` 下载完整，重启 ComfyUI，跑 EXP-01 单次换装。

## v0.4 — 2026-08-15

方案一 v3：在 v2 成品后增加三个定点修复分支。

目的：
清左腰旧纱残影、融腰线、修裙摆贴片，不再整衣高 denoise。

结果：
新增 `workflows/experiments/01_WAI_人物锁定换装-v03.json`。v2 未覆盖。

下一步：
先只跑分支 A 看残影，再开 B、C。

## v0.3 — 2026-08-15

方案一 v2：防分层节点级重构。

目的：
解决换装贴片感、衣缘分层、第二遍整衣精修破坏主体。

结果：
新增实验工作流 `workflows/experiments/01_WAI_人物锁定换装-v02.json`。v1 基线未覆盖。尚未出图验收。

下一步：
同一 Mask / seed 检查 Composite 后 Mask 外是否锁死，再扫 Seam × Denoise。

## v0.2 — 2026-08-15

加入方案一基线：原人物锁定换装。

目的：
把原整图 Img2Img 改成服装区域 Inpaint，MASK 外尽量保持原图像素。

结果：
建成 `workflows/baseline/01_WAI_人物锁定换装.json`。未跑 5 套服装验收，不能晋升 Release。

下一步：
固定 seed / Mask，单变量扫描第一遍 Denoise 0.22–0.38。

## v0.1 — 2026-08-15

建立仓库框架。

目的：
让后续工作流实验可追踪、可复现，并给 AI Agent 提供稳定入口。

结果：
已创建目录、知识库草稿、Prompt 模板、实验模板和日志索引。尚无 Baseline 工作流，尚未在 ComfyUI 中跑通。

下一步：
补齐 `docs/当前环境.md` 的 ComfyUI / 模型信息，导入第一份 `workflows/baseline` 工作流。
