# ComfyUI 工作流项目结构开发文档

## 1. 项目定位

本项目用于管理、开发、测试和迭代 ComfyUI 工作流。

项目重点不是单纯保存 `.json` 工作流，而是建立一套可长期维护的工作流开发体系，使 AI Coding Agent（如 Grok Build、Codex）能够：

- 理解当前 ComfyUI 环境
- 阅读现有工作流
- 根据需求修改工作流
- 保存不同实验版本
- 管理参考图片
- 记录节点与参数变化
- 对比不同方案
- 避免破坏已经稳定的工作流
- 将验证完成的工作流整理为正式版本

当前主要应用方向包括：

- 人物一致性
- 人脸保持
- 身材比例保持
- 肤色保持
- 风格保持
- 姿势控制
- 换装
- 构图调整
- ControlNet / IPAdapter 等控制方案测试
- ComfyUI 自定义节点测试
- 工作流性能与显存优化

---

# 2. 核心设计原则

## 2.1 工作流与文档分离

`.json` 工作流负责实际执行。

`.md` 文档负责描述：

- 为什么这样设计
- 节点承担什么职责
- 参数为什么这样设置
- 当前存在什么问题
- 下一步准备如何调整

禁止只修改工作流而完全不记录修改原因。

---

## 2.2 稳定版本与实验版本分离

稳定工作流不得直接用于实验性修改。

必须遵循：

```text
Baseline
    ↓
Experiment
    ↓
测试
    ↓
调整
    ↓
确认稳定
    ↓
Release
```

任何实验都应从明确的基线版本或已有实验版本派生。

---

## 2.3 尽量进行单变量实验

调试人物一致性时，禁止无目的地同时修改大量参数。

例如测试 IPAdapter 权重时：

```text
Experiment A
weight = 0.70

Experiment B
weight = 0.75

Experiment C
weight = 0.80
```

其他条件原则上保持一致。

这样才能判断变化究竟来自哪个参数。

---

## 2.4 不覆盖历史版本

除明显错误外，不直接覆盖历史实验文件。

例如：

```text
character-lock-v01.json
character-lock-v02.json
character-lock-v03.json
```

而不是不断覆盖：

```text
character-lock.json
```

---

# 3. 推荐项目目录

```text
ComfyUI-Workflow-Project/
│
├─ AGENTS.md
├─ README.md
├─ PROJECT_STRUCTURE.md
│
├─ docs/
│  │
│  ├─ README.md
│  ├─ 项目目标.md
│  ├─ 当前环境.md
│  ├─ 开发规范.md
│  ├─ 节点说明.md
│  ├─ 模型说明.md
│  ├─ 参数说明.md
│  └─ 已知问题.md
│
├─ workflows/
│  │
│  ├─ baseline/
│  │  └─ README.md
│  │
│  ├─ experiments/
│  │  └─ README.md
│  │
│  ├─ release/
│  │  └─ README.md
│  │
│  └─ archive/
│     └─ README.md
│
├─ references/
│  │
│  ├─ character/
│  ├─ face/
│  ├─ body/
│  ├─ pose/
│  ├─ clothing/
│  ├─ style/
│  └─ composition/
│
├─ prompts/
│  │
│  ├─ README.md
│  ├─ positive/
│  ├─ negative/
│  └─ templates/
│
├─ experiments/
│  │
│  ├─ README.md
│  └─ YYYY-MM-DD/
│
├─ results/
│  │
│  ├─ baseline/
│  ├─ experiments/
│  └─ comparison/
│
├─ logs/
│  │
│  ├─ CHANGELOG.md
│  └─ experiment-log/
│
├─ scripts/
│  ├─ README.md
│  └─ tools/
│
└─ temp/
```

---

# 4. 根目录文件职责

## 4.1 `AGENTS.md`

这是 AI Coding Agent 的项目入口。

主要告诉 Grok Build / Codex：

1. 这个项目是什么
2. 开始工作前应该阅读哪些文件
3. 哪些目录可以修改
4. 哪些文件不能直接覆盖
5. 工作流实验应该如何进行
6. 完成任务后需要输出什么
7. 如何记录实验结果

`AGENTS.md` 应保持简洁。

详细技术内容不要全部堆入 `AGENTS.md`，而应该引用 `docs/` 中对应文档。

---

## 4.2 `README.md`

面向项目使用者。

主要说明：

- 项目用途
- 当前工作流状态
- 推荐正式工作流
- 如何使用
- 文档入口
- 当前主要研究方向

---

## 4.3 `PROJECT_STRUCTURE.md`

即本文档。

负责规定整个项目：

- 文件夹结构
- 文件职责
- 工作流生命周期
- AI 开发规则
- 命名规则
- 实验管理方法

原则上不频繁修改。

---

# 5. `docs/` —— 项目知识库

这里保存 AI 开发工作流时需要理解的长期信息。

## 5.1 `项目目标.md`

描述最终希望实现什么。

例如人物一致性工作流可以定义：

### 必须保持

- 人物身份
- 面部结构
- 头部主要特征
- 发型主体结构
- 发色
- 肤色
- 身材比例
- 整体画风

### 允许改变

- 姿势
- 服装
- 表情
- 镜头
- 构图
- 背景

### 优先级示例

```text
人物身份
>
面部一致性
>
身材一致性
>
肤色一致性
>
风格一致性
>
姿势准确性
>
服装准确性
```

具体优先级可以根据实际任务调整。

---

## 5.2 `当前环境.md`

记录当前真实运行环境。

至少包括：

```text
GPU：
VRAM：
RAM：
操作系统：

ComfyUI 版本：
Python：
PyTorch：
CUDA：

Checkpoint：
VAE：

主要 LoRA：

ControlNet：

IPAdapter：

自定义节点：
```

必须特别记录显存限制。

AI 设计工作流时不得无视实际硬件条件。

---

## 5.3 `开发规范.md`

规定工作流修改方式。

例如：

- 不直接覆盖 baseline
- 实验必须创建新版本
- 大规模调整前先分析现有工作流
- 优先解决明确问题
- 避免无意义增加节点
- 避免重复模型加载
- 考虑 VRAM
- 不得随意更换基础模型
- 不得为了一个指标明显破坏其他核心指标

---

## 5.4 `节点说明.md`

记录项目使用过的重要节点。

推荐格式：

```text
节点名称：

节点包：

用途：

输入：

输出：

关键参数：

当前推荐参数：

与其他节点关系：

显存影响：

已知问题：

注意事项：
```

---

## 5.5 `模型说明.md`

记录：

- Checkpoint
- LoRA
- VAE
- ControlNet
- CLIP Vision
- IPAdapter Model
- Embedding
- Upscaler

等模型信息。

每个模型至少说明：

```text
名称
版本
用途
文件位置
适用工作流
推荐权重
已知问题
是否必须
```

---

## 5.6 `参数说明.md`

记录经过验证的重要参数。

例如：

```text
Sampler
Scheduler
Steps
CFG
Denoise
Resolution

IPAdapter Weight
IPAdapter Start
IPAdapter End

ControlNet Strength
ControlNet Start
ControlNet End

LoRA Weight
```

应尽量说明：

> 参数变化 → 实际视觉结果变化

而不是只记录数字。

---

## 5.7 `已知问题.md`

统一维护当前尚未解决的问题。

例如：

```text
[P01] 换装后腰臀比例容易发生变化

[P02] 大幅改变姿势时面部一致性下降

[P03] IPAdapter 权重过高时会继承参考图服装

[P04] 高分辨率时显存不足

[P05] 某自定义节点偶发报错
```

方便 AI 后续针对问题继续开发。

---

# 6. `workflows/` —— 工作流核心目录

## 6.1 `baseline/`

保存经过确认的实验基线。

例如：

```text
baseline/
├─ character-base-v1.json
└─ README.md
```

Baseline 的意义是：

> 后续实验结果与谁比较。

原则上不得直接修改。

---

## 6.2 `experiments/`

保存正在测试的工作流。

例如：

```text
experiments/
├─ character-ipadapter-v01.json
├─ character-ipadapter-v02.json
├─ character-openpose-v01.json
├─ character-body-lock-v01.json
└─ character-clothing-v01.json
```

这里允许快速迭代。

---

## 6.3 `release/`

只保存经过实际验证的正式工作流。

例如：

```text
release/
├─ character-consistency-v1.0.json
├─ character-consistency-v1.1.json
└─ README.md
```

只有满足核心目标并经过测试后才能进入这里。

---

## 6.4 `archive/`

保存：

- 已淘汰工作流
- 旧架构
- 不再使用但可能具有参考价值的方案

不要因为方案失败就立即删除。

失败方案本身也可能具有调试价值。

---

# 7. `references/` —— 参考素材

按照参考信息的职责分类，而不是把所有图片混在一起。

```text
references/
├─ character/
├─ face/
├─ body/
├─ pose/
├─ clothing/
├─ style/
└─ composition/
```

### character

完整人物参考。

### face

主要用于：

- Identity
- 五官
- 面部结构

### body

主要用于：

- 身材
- 比例
- 体型

### pose

主要用于：

- OpenPose
- 姿态控制

### clothing

服装目标参考。

### style

绘画风格参考。

### composition

镜头和构图参考。

---

# 8. `prompts/` —— Prompt 管理

不要长期把重要 Prompt 只保存在工作流节点内部。

建议将稳定 Prompt 独立保存。

例如：

```text
prompts/
├─ positive/
│  ├─ character-base.md
│  ├─ clothing.md
│  └─ quality.md
│
├─ negative/
│  ├─ base-negative.md
│  └─ anatomy-negative.md
│
└─ templates/
   └─ character-template.md
```

这样方便：

- 对比
- 复用
- 修改
- 版本管理

---

# 9. `experiments/` —— 实验记录

注意：

```text
workflows/experiments/
```

保存的是**实验工作流 JSON**。

而：

```text
experiments/
```

保存的是**实验本身的记录**。

例如：

```text
experiments/
└─ 2026-08-15/
   ├─ EXP-001.md
   ├─ EXP-002.md
   └─ EXP-003.md
```

---

# 10. 实验文档标准

每次重要实验建议建立：

```text
EXP-XXX.md
```

内容：

```text
实验编号：

日期：

目标：

基于工作流：

测试工作流：

参考素材：

Seed：

修改内容：

修改前参数：

修改后参数：

保持不变的变量：

测试结果：

优点：

缺点：

是否改善：

结论：

下一步：
```

---

# 11. `results/` —— 输出结果

生成结果不要和参考图片混合。

```text
results/
├─ baseline/
├─ experiments/
└─ comparison/
```

其中 `comparison/` 可以保存：

- A/B 对比
- 参数对比
- 不同工作流对比
- 最终候选结果

---

# 12. `logs/`

## `CHANGELOG.md`

记录项目级的重要变化。

例如：

```text
v0.3

加入 OpenPose 控制支路。

目的：
解决大幅改变人物姿势时结构不稳定的问题。

结果：
姿势准确性明显改善，但人物身份一致性略有下降。
```

---

# 13. `scripts/`

这里保存辅助脚本。

例如未来可以开发：

```text
工作流 JSON 参数提取
工作流 JSON 差异比较
缺失节点检查
模型路径检查
工作流版本生成
Prompt 批量替换
参数批量实验
实验结果整理
```

禁止把这些辅助程序散落在项目根目录。

---

# 14. `temp/`

只允许保存：

- 临时工作流
- 临时图片
- 调试文件
- 中间文件

任何重要内容不得长期只存在于 `temp/`。

该目录可以加入 `.gitignore`。

---

# 15. 工作流命名规范

推荐：

```text
功能-方案-v版本.json
```

例如：

```text
character-base-v01.json

character-ipadapter-v01.json
character-ipadapter-v02.json

character-openpose-v01.json

character-body-lock-v01.json

character-clothing-v01.json
```

正式版本：

```text
character-consistency-v1.0.json
character-consistency-v1.1.json
```

禁止：

```text
最终版.json
最终版2.json
真的最终版.json
最新.json
新工作流.json
111.json
test123.json
```

---

# 16. AI Coding Agent 标准开发流程

Grok Build、Codex 或其他 Coding Agent 接收到 ComfyUI 工作流开发任务后，应按照以下顺序执行。

## Step 1：读取项目规则

首先阅读：

```text
AGENTS.md
PROJECT_STRUCTURE.md
docs/项目目标.md
docs/当前环境.md
docs/开发规范.md
```

---

## Step 2：读取任务相关资料

根据任务进一步阅读：

```text
docs/节点说明.md
docs/模型说明.md
docs/参数说明.md
docs/已知问题.md
```

以及相关：

```text
workflows/
references/
prompts/
experiments/
```

---

## Step 3：分析现有工作流

在修改之前先回答：

1. 当前工作流的数据流是什么？
2. 每个关键节点负责什么？
3. 当前问题可能来自哪个环节？
4. 哪些节点与问题无关？
5. 最小修改方案是什么？
6. 是否会超过当前硬件能力？

禁止在没有理解工作流结构的情况下进行大范围随机修改。

---

# 17. 工作流修改原则

优先采用：

```text
发现问题
↓
提出假设
↓
确定变量
↓
创建实验版本
↓
修改
↓
运行
↓
比较结果
↓
记录结论
↓
继续下一轮
```

而不是：

```text
发现问题
↓
同时修改十几个参数
↓
图片似乎变好了
↓
不知道为什么
```

---

# 18. 人物一致性工作流的模块化思想

人物工作流尽量按照职责理解：

```text
Checkpoint
    ↓
决定基础生成能力
```

```text
Identity / IPAdapter
    ↓
人物身份
```

```text
Face Reference
    ↓
面部特征
```

```text
Body Reference
    ↓
身材与比例
```

```text
OpenPose
    ↓
姿势
```

```text
Prompt / Clothing Reference
    ↓
服装
```

```text
Style
    ↓
整体视觉风格
```

```text
Sampler
    ↓
最终生成
```

不同控制模块尽量职责分离。

目标是：

> 锁定需要保持的变量，释放需要改变的变量。

---

# 19. 人物工作流修改优先级

针对人物一致性任务，默认优先保证：

```text
身份
↓
脸部
↓
头发
↓
肤色
↓
身材
↓
风格
```

然后允许：

```text
姿势
服装
构图
镜头
背景
```

如果某个方案能够很好地改变服装，但导致人物身材或身份明显变化，则原则上不能直接认定为更优方案。

---

# 20. 硬件约束

工作流设计必须以真实硬件为基础。

对于低显存环境尤其需要关注：

- 同时加载的模型数量
- ControlNet 数量
- CLIP Vision
- IPAdapter
- 图像分辨率
- Batch Size
- VAE Decode
- Upscale
- High-Res 流程
- 模型重复加载

不能单纯追求理论效果而忽略工作流是否能够实际运行。

---

# 21. AI 修改工作流后的交付要求

每次重要修改完成后，至少汇报：

## 修改了什么

列出：

- 新增节点
- 删除节点
- 修改连接
- 修改参数
- 修改 Prompt
- 修改模型

## 为什么修改

说明每个关键修改对应解决什么问题。

## 生成了什么

给出：

```text
新工作流路径
实验文档路径
相关结果路径
```

## 与 Baseline 的差异

明确指出：

```text
Baseline
→
Experiment
```

发生了哪些变化。

## 当前结论

说明：

- 已解决
- 部分解决
- 未解决
- 新发现的问题

## 下一步

提出下一轮最值得测试的变量。

---

# 22. 禁止事项

AI Agent 不得：

1. 未经说明直接覆盖 Baseline。
2. 未经说明覆盖 Release。
3. 删除历史实验记录。
4. 为解决一个小问题无目的重构整个工作流。
5. 随意更换基础模型。
6. 忽略显存限制。
7. 同时修改大量变量后声称某个单独参数有效。
8. 将参考图片和生成结果混为一谈。
9. 使用无法确认存在的节点或模型而不做说明。
10. 将未经测试的工作流直接标记为 Release。
11. 删除失败实验而不保留必要结论。
12. 在没有阅读相关文档的情况下直接开始大规模修改。

---

# 23. 推荐的长期工作模式

最终形成：

```text
项目目标
   ↓
Baseline
   ↓
发现问题
   ↓
建立 EXP
   ↓
生成实验 Workflow
   ↓
运行测试
   ↓
保存 Results
   ↓
记录结果
   ↓
继续迭代
   ↓
达到目标
   ↓
Release
```

整个项目应形成完整链路：

```text
为什么修改
+
修改了什么
+
产生什么结果
+
结果是否更好
+
下一步做什么
```

---

# 24. 最终目标

本结构的目的不是单纯让项目文件看起来整齐。

真正目标是让任何新的 AI Coding Agent 进入项目以后，都能够通过：

```text
AGENTS.md
        ↓
docs/
        ↓
workflows/
        ↓
experiments/
        ↓
results/
        ↓
logs/
```

快速恢复完整开发上下文。

即使经过几十轮甚至上百轮 ComfyUI 工作流实验，也能够回答：

- 当前最好的工作流是哪一个？
- 它基于哪个版本？
- 为什么加入这些节点？
- 哪些参数已经测试过？
- 哪些方案失败过？
- 为什么失败？
- 哪些参数最敏感？
- 当前还存在什么问题？
- 下一步最值得测试什么？

以此实现 ComfyUI 工作流的可重复、可追踪、可迭代开发。