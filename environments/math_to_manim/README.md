# Math-To-Manim Visual Repair

Math-To-Manim 把简短教育文本 prompts 转成渲染后的 Manim animations：typed planning artifacts、generated Python scenes、validation reports，以及 MP4/GIF outputs。

这个 Prime Intellect environment 负责闭合改进循环。它不是训练抽象 coding tasks，而是用真实 Math-To-Manim run bundles 训练模型：original prompt、scene specification、generated Manim code、render/validation evidence 和 review signals。模型的任务是修复或改进已经可工作的输出，让未来 generations 更正确、更可读、更视觉稳健。

简写如下：

```text
text prompt
  -> Math-To-Manim pipeline
  -> generated Manim scene
  -> MP4/GIF output + validation evidence
  -> Prime RL repair task
  -> better generated animation code
```

当前 environment 专注于快速 generated-code repair rewards，因为 full video rendering 对每个 RL rollout 来说太慢。它奖励 valid `GeneratedCode` JSON、可解析且安全的 Manim Python、预期 scene structure、保留的 math intent，以及在昂贵 render audits 前减少拥挤文本和公式的 static layout improvements。

## Environment

- Hub ID: `harleycooper/math-to-manim`
- Package: `math-to-manim`
- Import package: `m2m2_visual_repair`
- Task: single-turn generated-code repair for text-prompt-to-animation runs

模型接收 M2M2 prompt、scene spec、current generated code，以及 validation/render/review evidence。它必须准确返回一个 `<generated_code>...</generated_code>` block，其中 JSON 至少包含 `scene_name` 和 `code`。

默认 dataset 包含来自 README GIF 的 QED/Minkowski layout-repair task。它的 reward 包含 static text-crowding checks，用于检查没有 `scale_to_fit_width` 的长公式、过密 text grouping，以及过多 fixed-frame overlays。Full rendering 仍然是 eval/audit step，不是 per-rollout reward。

## Local Use

```bash
uv pip install -e environments/math_to_manim
uv run python -c "from verifiers import load_environment; env = load_environment('math-to-manim'); print(len(env.dataset))"
uv run vf-eval math-to-manim -n 2
```

## Export M2M2 Runs

从 Math-To-Manim repo 运行：

```bash
python -m math_to_manim.cli pi-export-runs \
  --runs-dir runs \
  --output environments/math_to_manim/m2m2_visual_repair/data/repair_tasks.jsonl
```

## Publish

在已认证 Prime environment 中运行：

```bash
prime env push --path environments/math_to_manim --name math-to-manim --visibility PUBLIC
```

在 Codex workspace sandbox 内，使用 writable-home wrapper：

```bash
prime-codex env push --path environments/math_to_manim --name math-to-manim --visibility PUBLIC
```

## Training Templates

Config snippets 位于 `m2m2_visual_repair/configs/`。

- Smoke: `Qwen/Qwen3.5-0.8B`
- Practical repair: `Qwen/Qwen3-30B-A3B-Instruct-2507`
- Follow-up: `Qwen/Qwen3.5-397B-A17B`
