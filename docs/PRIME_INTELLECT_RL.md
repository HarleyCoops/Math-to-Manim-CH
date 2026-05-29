# Prime Intellect RL 集成

Math-To-Manim 把简短的教育 prompt 转成渲染后的 Manim 动画：类型化规划产物、生成的 Python 场景、验证报告，以及 MP4/GIF 输出。Prime Intellect RL 是让这些已经可工作的输出随时间变得更好的反馈循环。

目标不是让 RL 从零发明整条视频管线。M2M2 已经能创建 run bundles 和渲染输出。Prime 接收这些 bundles 作为可验证的修复任务，围绕 prompt、scene spec、generated code、validation/render evidence 和 review signals 进行训练，然后奖励那些能让未来生成动画更正确、更可读、更安全、更稳健的修复。

```text
text prompt
  -> Math-To-Manim pipeline
  -> generated Manim scene
  -> MP4/GIF output + validation evidence
  -> Prime RL repair task
  -> better generated animation code
  -> better future MP4/GIF outputs
```

## Prime 运行什么

Prime 的 RL stack 把任务分成三部分：

- **Environment**：一个 Verifiers package，负责提供任务并给 completion 打分。
- **Orchestrator**：从模型采样 rollouts，调用 environment，并把 rewards 转成训练 batches。
- **Trainer/inference**：更新 policy model，并服务最新权重。

对 M2M2 来说，环境是 `harleycooper/math-to-manim`。第一个训练面是 generated-code repair，因为它足够快，可以用于 RL rollouts，同时仍然瞄准最终产品：高质量渲染数学动画。

```text
M2M2 run bundle
  -> prompt + scene_spec + generated_code + render/validation/review evidence
  -> model returns GeneratedCode JSON
  -> fast static reward
  -> Prime RL update
```

完整渲染和视频审阅仍然是较慢的 audit/eval 层。RL loop 默认使用静态 code 和 layout proxies，让 rollouts 保持便宜；随后正常的 Math-To-Manim 管线可以把改进后的代码渲染成 MP4/GIF。

## Environment Contract

独立 package 位于 `environments/math_to_manim/`。

- Package name: `math-to-manim`
- Import package: `m2m2_visual_repair`
- Hub ID: `harleycooper/math-to-manim`
- Entry point: `from m2m2_visual_repair import load_environment`

Verifiers 会通过 import `math_to_manim` 来解析 `math-to-manim`，因此独立 environment package 内含一个小的 `math_to_manim` compatibility shim。安装 `m2m2_visual_repair` 后，主仓库 package 也会暴露一个 lazy `load_environment()` 函数。

模型必须准确返回一个带标签的 JSON block：

```text
<generated_code>{"scene_name":"...","language":"python","code":"..."}</generated_code>
```

默认 reward 使用静态检查：

- 输出含有必需 tag；
- JSON 匹配必需的 `GeneratedCode` shape；
- 生成的 Python 可解析；
- 预期 Manim scene class 有 `construct` method；
- 不存在 unsafe imports/calls；
- 预期数学术语出现在代码中；
- 静态 layout checks 在不渲染的情况下估算文本拥挤风险。

Layout reward 是 proxy，不是视觉 oracle。它会检查返回的 Manim source 中是否存在高字号长 `Text`/`MathTex`、缺少 `scale_to_fit_width`、过密 text-group buffers，以及过多 fixed-frame overlays。这样设计是为了在 RL rollouts 中降低拥挤脚本出现概率，同时让完整渲染继续作为较慢 audit step。

渲染和视频审阅默认不在 reward loop 中，因为 Manim renders 对 RL rollouts 来说太慢、太贵。

## Export Tasks

从现有 M2M2 run bundles 创建 JSONL dataset：

```bash
python -m math_to_manim.cli pi-export-runs \
  --runs-dir runs \
  --output environments/math_to_manim/m2m2_visual_repair/data/repair_tasks.jsonl
```

Exporter 会跳过不完整的 run bundles，并只写入文本产物。它不会复制 videos、media folders、credentials 或 `.env` 文件。

## Local Verification

```bash
uv pip install -e environments/math_to_manim
uv run python -c "from verifiers import load_environment; env = load_environment('math-to-manim'); print(len(env.dataset))"
uv run vf-eval math-to-manim -n 2
```

对于当前 text-crowding target，内置默认 dataset 包含一个来自 QED/Minkowski README GIF run 的 layout-repair task。它要求模型保留 QED 教学弧线，同时让字幕和公式更稀疏、分阶段、缩放合理且可读。

## 发布到 Prime

在已认证的 Prime environment 中运行：

```bash
prime env push --path environments/math_to_manim --name math-to-manim --visibility PUBLIC
```

在 Codex workspace sandbox 内，使用 writable-home wrapper，让 Prime CLI 能更新临时 config/cache，而不用写入真实的 `~/.prime`：

```bash
prime-codex env push --path environments/math_to_manim --name math-to-manim --visibility PUBLIC
```

## Training Templates

模板位于 `environments/math_to_manim/m2m2_visual_repair/configs/`。

- Smoke: `Qwen/Qwen3.5-0.8B`
- Practical repair: `Qwen/Qwen3-30B-A3B-Instruct-2507`
- Follow-up: `Qwen/Qwen3.5-397B-A17B`

先用 smoke model 验证 environment 和 reward wiring。第一次严肃 Manim-code repair run 使用 practical repair model。只有在 reward curves 稳定且 environment 有干净 eval signal 后，才使用 follow-up model。
