# EXP-010A 实验记录

```text
实验编号：        EXP-010A
日期：            2026-08-17
目标：            P06 Mask Audit / Override Fix。不评价 Telea，不降 Denoise
对应问题：        P06 / P11
基于工作流：      workflows/experiments/WAI_AutoClothes_Inpaint_v1-exp010.json
测试工作流：      workflows/experiments/WAI_AutoClothes_Inpaint_v1-exp010a.json
参考素材：        胡.jpg
Seed：            114514 / fixed

修改内容：
- 不覆盖 EXP-010
- Garment Setting：pants = ON（其余类别不变）
- 重写 Final Edit 优先级：
  AUTO_EDIT = Garment − (Hard + Soft)
  EDIT = AUTO_EDIT + Manual Add
  FINAL = EDIT − Hard − Manual Protect
- Manual Add 可覆盖手臂/腿 Soft Protect
- Hard Protect（脸/发/帽）在 Add 之后再减，不能被 Add 打开
- 增加 ②b HARD / ②c SOFT Preview
- 不改 Telea、Denoise、Context、Prompt

修改前参数：
pants=OFF
FINAL = (Garment + Manual Add) − (Hard + Soft + Manual Protect)

修改后参数：
pants=ON
Manual Add 在 Soft Protect 之后，Hard Protect 仍最后挡住脸发

保持不变的变量：
WAI v1.7、Fooocus v26、24/5/euler_ancestral/0.95、
Telea/falloff=0、Context 24、Color Match 0、
胡.jpg、Seed 114514、runtime Prompt

测试结果：
JSON 连线校验 0 错误。
2026-08-18 回查：Comfy Shared output 与 results/baseline 均无 `WAI_autoclothes_exp010a_*.png`。
最新成品仍是 `WAI_autoclothes_exp010_00001_.png`（22:19）。
因此「生成图没改善」目前不能记成 EXP-010A 失败，只说明 EXP-010 成品本来就没改善。

结果路径：
尚无 010A 成品。最新对照仍是 results/baseline/WAI_autoclothes_exp010_00001_.png

优点：
髋侧漏检和手臂勒痕被保护掉，这两个假设可以分开验证

缺点：
一次改了类别和优先级两处，结论必须按四格诊断归因，不能只说「010A 有效」

是否改善：        无法判断
结论：            Mask 审计图已落地。先填四格，再决定要不要 Manual Add 或 EXP-011。

下一步：
1. 看 ① 髋侧蓝布是否进 Garment
2. 看 ②c 是否吃掉左臂勒痕
3. 看 ②b 是否吃掉颈侧蓝带
4. 看 ③ Final 和 ④ Pre-fill 给四格归类
5. 只有 Case 4 才派生 EXP-011
```
