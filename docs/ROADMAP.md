# 路线图：可编辑动画工作流

M2M2 暂时不打算把渲染出来的 MP4 本身作为主要编辑界面。当前设计把视频变得可编辑，是通过保留生成视频的管线产物：prompt、概念意图、先修知识图、课程计划、数学包、分镜、场景规格、生成的 Manim 代码、验证报告、渲染结果、审阅报告和 manifest。

## 当前进展

- 中间产物已经是编辑 contract。一次 deterministic run 会把可检查的 JSON 和 `generated_scene.py` 写入 `runs/<run_id>/`，因此审阅者可以看到哪个 prompt、storyboard、scene spec 和 code 生成了输出。
- Prompt-level 编辑已经支持：用修改后的 prompt、style、quality 或 render option 重新运行 CLI。
- Spec/code-level 编辑是近期工作流：编辑 `scene_spec.json` 或 `generated_scene.py`，然后针对同一个 run bundle 重新运行验证和渲染。
- 静态验证已经是渲染前的 gate。验证失败时应在 Manim 运行前停止。
- Render repair 是架构的一部分：失败渲染可以把 Manim stderr/stdout 和冻结的 `scene_spec` 交回代码修复路径，而不需要重新计算整条规划链。
- Hermes 和 Codex 是 agent-assisted 迭代层。Hermes 可以检查产物、修补 prompts/specs/code、运行 CLI smoke checks、审阅 frames 或 GIF，并保持编辑历史明确；选择本地 Codex CLI provider 时，Codex 可用于 generated-code 和 repair 阶段。

## 计划中的编辑循环

```text
prompt
  -> typed planning artifacts
  -> storyboard / scene_spec edits
  -> generated_scene.py edits or Codex-assisted repair
  -> static validation
  -> Manim rerender
  -> video review artifacts
  -> next edit
```

关键产品方向是 artifact-first editing：

- 讲解目标、受众、风格或时长不对时，编辑 prompt。
- 教学顺序或视觉对象不对时，编辑 storyboard 或 `scene_spec.json`。
- Manim 实现细节不对时，编辑 `generated_scene.py`。
- 验证或渲染失败时，使用 repair loop。
- 使用 Hermes/Codex 让这些编辑可重复，而不是把每段视频都当成一次性渲染。

## 暂不在范围内

- 完整浏览器视频时间线编辑器。
- 直接逐帧编辑 MP4。
- 用大型 UI 在画布上拖拽 Manim objects。

这些以后可能会出现，但第一个有用的可编辑工作流已经和 M2M2 架构一致：保留产物，回到问题引入的阶段进行修改，然后重新渲染。

## 中文 issue 回复草稿

目前“视频可编辑”的进展主要在工作流和中间产物层面，而不是已经完成一个大型可视化剪辑器。

现在的方向是：每次生成都会保留 prompt、知识图谱、课程顺序、数学包、storyboard、`scene_spec.json`、`generated_scene.py`、验证报告、渲染报告和 review 结果。这样视频不是一个黑盒 MP4，而是可以回到对应阶段修改：想改讲解目标就改 prompt，想改画面节奏就改 storyboard/scene spec，想改 Manim 细节就改生成代码，然后通过静态验证、repair loop 和重新渲染得到新版视频。

Hermes/Codex 会作为辅助迭代层：检查这些产物、修改 spec 或代码、运行验证/渲染、查看帧或 GIF，再继续下一轮。短期计划是把这个“中间产物可编辑 + 重新渲染/修复”的流程打磨清楚；完整的浏览器时间线编辑器不在当前最小范围内。
