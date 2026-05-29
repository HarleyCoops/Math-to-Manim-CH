# 提升动画质量的领域 Skills

M2M2 把物理和数学 skills 视为贡献者指南和审阅流程，而不是隐藏的运行时依赖。一个 domain skill 应帮助 Hermes、Codex 或其他 operator 把 prompt 转成更好的 `storyboard.json`、`scene_spec.json`、Manim 实现和 review record，同时保留管线规则：先有 typed artifacts，再写 code，只有验证通过后才 render。

## 物理 skill 应包含什么

面向物理的 skill 如果能在写 Manim code 前把物理直觉说清楚，就有价值。它应捕捉这样的约束：

- 选择视觉方案之前，先说清守恒量或变化量；
- 先展示原因，再展示结果，例如先画力箭头再画加速度，先展示场的几何再展示粒子运动；
- 在镜头之间保持 units、axes、labels 和 scale changes 一致；
- 优先使用局部几何证据，而不是符号捷径，例如 slopes、flux、phase、curvature 或 area accumulation；
- 标出不可能的运动、不连续的状态变化、误导性透视，以及暗示错误机制的装饰；
- 说明正在可视化哪些近似，例如 small-angle motion、frictionless motion、point masses、ideal fluids 或 nonrelativistic limits。

这些约束应尽可能进入 planning artifacts。例如，gravity prompt 应生成 storyboard beats，先揭示 curvature、orbit state 和 conservation cues，再让 scene spec 要求 Manim 做 camera move。Quantum prompt 应区分 amplitude、probability、measurement 和 basis choice，而不是把所有 glow 或 randomness 都当作同一种东西。

## 可复用 Manim 模式

Domain skills 也可以维护一组可复用模式，但不应把仓库变成某种 style clone。合适的候选项是小型、可检查 recipes：

- 用于导数和局部线性的 tangent/secant transforms；
- 用于力和流的 vector fields、streamlines 和 field-line density；
- 用于动力学的 phase-space traces 和 energy contours；
- 用于振动主题的 wave superposition、envelopes 和 interference；
- 用于 stochastic processes 的 distribution clouds、histograms 和 highlighted sample paths；
- 用于几何的 camera-safe 3D axes、surface slices 和 projection helpers。

这些内容应写成 constraints 和 examples，让 code generator 能适配当前 `ManimSceneSpec`。它们不应要求从 `math_to_manim` import Hermes 或任何 skill package；package dependencies 留在 `pyproject.toml`，skills 仍然是 operator-side procedure。

## 验证和审阅循环

Domain skill 本身不能保证正确。它改进的是现有 M2M2 循环周围的 prompts、checks 和 review rubric：

1. `IntentAgent` 和 `CurriculumAgent` 识别物理或数学思想、先修知识，以及面向学习者的误解风险。
2. `StoryboardAgent` 记录直觉节拍：什么先出现，什么移动，什么保持不变，labels 或 equations 在哪里进入。
3. `SceneSpecAgent` 把这些节拍转成具体 Manim objects、timing、camera choices 和 validation expectations。
4. `StaticReviewAgent` 在 render 前拦截 unsafe 或 malformed generated Python。
5. Render 和 video review 检查动画是否真的传达预期机制，而不只是生成了一个文件。

在 Hermes/Codex 工作中，skill 应与 `codebase-inspection`、`manim-video` 和 `systematic-debugging` 一起预加载。Operator 可以检查 run bundle，对比 `storyboard.json` 和 `generated_scene.py`，在依赖可用时渲染，并记录最终运动是否满足领域约束。

## 3Blue1Brown 灵感政策

3Blue1Brown 是数学传播的重要参考点：几何第一原则、渐进揭示、谨慎相机运动、可读符号，以及每个节拍只讲一个清晰想法。M2M2 可以把这些一般原则提炼成 skills 和 rubrics。

M2M2 不应复制专有 3Blue1Brown 代码，不应逐镜头复刻视频，也不应把生成场景营销成 3Blue1Brown-style replicas。好的 skill 描述可迁移的教学模式，例如“只有在几何已经可见之后才引入符号”或“当一个量变化时，让一个 invariant 在视觉上保持锚定”。它应避免“copy this scene”“match this exact palette”或“reproduce this animation”这样的指令。

因此，对 issue #39 的实际回答是：是的，domain-specific skills 很适合提升物理直觉和可复用 Manim craft，但它们应作为透明的 Hermes/Codex procedures 和 review rubrics 存在。它们应提炼广泛有用的原则和本仓库模式，而不是私人代码或专有艺术身份。
