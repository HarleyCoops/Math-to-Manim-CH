# Hermes 学习 Manim

这是一个 launch concept：把 M2M2 展示为 Hermes Agent 使用 native tools 制作数学动画的活体 demo。

## 一句话 launch frame

Hermes 学习 Manim：一个 agent 阅读仓库、规划课程、写入 typed artifacts、生成 Manim code、运行 CLI、审阅渲染，并把最好的 motion beat 转成 showcase GIF。

## Repo operator model

Hermes 提供 developer/operator tools；M2M2 提供动画管线。Hermes 会阅读和搜索仓库、patch 文件、运行终端检查、用 vision 审阅 frames/GIFs、委派较大工作、跟踪 todos/session state，并加载 task skills。M2M2 给这些工具提供具体操作面：`m2m2` CLI、`math_to_manim/tools/`、typed stage artifacts、render/review helpers，以及 `runs/<run_id>/` bundles。

## 我们应该在 X/Twitter 上展示什么

重点不只是“AI 做了一个 Manim 视频”。重点是 Hermes 能原生使用开发环境：

1. 加载 repo instructions 和 skills。
2. 检查 codebase 和 docs。
3. 运行 setup、tests 和 CLI smoke checks。
4. 通过 M2M2 生成或修复 Manim code。
5. 根据依赖情况选择 render 或 no-render。
6. 检查 generated artifacts 和 media。
7. 用 vision/contact sheets 判断一个 GIF 是否真的好。
8. 只有验证通过后才 commit docs/assets。

这才是故事：Hermes 不是给 Manim 建议的 chatbot；Hermes 是一个使用工具、操作仓库的协作者。

## 可展示的 native Hermes tool moments

| Moment | Hermes 做什么 | 为什么在视觉上重要 |
| --- | --- | --- |
| Skill load | 从 `agents-md`、`manim-video`、`codebase-inspection` 或 `systematic-debugging` 开始 | 说明这是程序化知识，不是 vibes。 |
| Repo inspection | 读取 `pyproject.toml`、`README.md`、`AGENTS.md`、`docs/ARCHITECTURE.md` 和 CLI help | 说明 agent 扎根于真实项目。 |
| Terminal run | 执行 `pytest`、CLI help 和 deterministic smoke generation | 在创意声明前证明 repo 能运行。 |
| Artifact trail | 打开 `runs/<run_id>/*.json`、`generated_scene.py` 和 `manifest.json` | 让管线可读。 |
| Render path | 可用时使用 Manim/FFmpeg，或记录为什么跳过 render | 展示诚实验证。 |
| Visual review | 构建 contact sheet 或用 vision tools 检查 GIF | 展示超越“文件存在”的媒体验证。 |
| GitHub workflow | Commit/push README/showcase assets，并验证 remote PR | 闭合从 idea 到 published repo 的循环。 |

## 建议 X/Twitter thread

1. “Hermes learns Manim.”
   - 发布 hero image。
   - Message：M2M2 把 prompts 转成 typed planning artifacts、Manim code、renders 和 review bundles。

2. “重要的是：Hermes 原生使用工具。”
   - 截图或短片展示 Hermes 运行 repo commands，而不只是聊天。
   - 提到 skills、terminal、file inspection、vision review 和 GitHub verification。

3. “每个动画都是 artifact trail。”
   - 展示 pipeline diagram 或 run folder list。
   - 强调 `intent.json -> storyboard.json -> scene_spec.json -> generated_scene.py -> render_result.json -> manifest.json`。

4. “美术方向目标。”
   - 分享本地 showcase GIF grid。
   - 解释 legacy Math-To-Manim GIFs 已被本地复制为视觉标准。

5. “下一步：新的 M2M2-native animations。”
   - 分享下方 slate 中的 3-5 个 upcoming concepts。
   - 问大家希望 Hermes 先学哪个。

## Launch copy options

Short:

> Hermes learns Manim.
>
> 我们围绕 typed artifacts、agent skills、native tooling 和可验证 render loops 重建了 Math-To-Manim。目标是一个能规划、写代码、运行、检查、审阅和发布数学运动的 AI collaborator，而不只是描述它。

More technical:

> “Hermes learns Manim” 是我们的 M2M2 demo：prompt -> typed curriculum -> scene spec -> generated Manim -> render/review bundle。
>
> Hermes 使用 native repo tools：skills、file inspection、terminal commands、tests、CLI smoke runs、visual GIF validation 和 GitHub PR verification。

Thread opener:

> Hermes learns Manim.
>
> 我们正在把 Math-To-Manim 变成 agent-native animation lab。Hermes 可以检查 repo、加载 skills、运行 tests、调用 CLI、审阅生成 media，并把验证过的 GIF 推进 showcase。
>
> 不是 chatbot。是使用工具的协作者。

## Animation slate：新的 M2M2-native showcase 候选

这些应由 rewrite pipeline 生成，而不是从 legacy repo 复制。每一个都设计用来展示 Hermes + M2M2 的不同强项。

### 1. The Agent Learns a Tangent

Prompt:

```text
Create a cinematic Manim explainer showing an AI agent learning that derivatives are slopes: a secant line slides along a curve, the two points collapse into one, the tangent locks into place, and the derivative notation appears only after the geometric reveal.
```

为什么展示：

- 熟悉的微积分概念。
- 强视觉 aha moment。
- 与 legacy hero GIF 直接连续。

关键节拍：

- `Δx` 明显缩小，直到割线变成切线。

### 2. Fourier as a Drawing Machine

Prompt:

```text
Create a cinematic Manim explainer showing Fourier epicycles as a drawing machine: rotating vectors attach head-to-tail, their endpoint traces a luminous curve, and the viewer sees how adding frequencies sharpens the drawing.
```

为什么展示：

- 对 Manim 受众来说极具标志性。
- 很适合 Twitter loop。
- 展示运动、累积和近似。

关键节拍：

- 三个圆变成十二个；endpoint 突然解析出可识别曲线。

### 3. Gradient Descent as Terrain Navigation

Prompt:

```text
Create a Manim animation explaining gradient descent as a glowing particle moving over a loss landscape: contour lines appear first, the gradient arrow points downhill, the step size changes the trajectory, and overshooting is contrasted with stable convergence.
```

为什么展示：

- 连接数学、ML 和 agent training。
- 让 Hermes 展示参数直觉。
- 容易比较“坏”和“好”的 update rules。

关键节拍：

- 过大的 learning rate 疯狂反弹，然后调好的 rate 螺旋进入 basin。

### 4. Attention as Moving Light

Prompt:

```text
Create a Manim explainer of transformer attention as moving light: tokens appear as nodes, query/key similarity lights up weighted edges, values flow along the brightest paths, and one output token forms from a weighted mixture.
```

为什么展示：

- 适合 Hermes launch 的 AI-native topic。
- 强视觉 metaphor。
- 把 agent tooling 和 model internals 连接起来。

关键节拍：

- 一个 token 发问；只有相关 context tokens 发光并流入它。

### 5. The Pipeline Becomes a Scene

Prompt:

```text
Create a meta Manim animation of the M2M2 pipeline itself: a user prompt enters as a spark, becomes typed JSON cards, transforms into a storyboard, compiles into Manim code, renders into a video frame, and ends as a manifest-backed showcase GIF.
```

为什么展示：

- 最适合“Hermes learns Manim”，因为它展示 toolchain 本身。
- 可以映射 README pipeline diagram。
- 让 typed artifacts 在情绪上变得可读。

关键节拍：

- JSON cards 吸附成 filmstrip，然后 filmstrip 活起来。

### 6. Brownian Motion Becomes Finance

Prompt:

```text
Create a Manim animation showing Brownian motion becoming a finance model: many random paths bloom from one point, their distribution widens over time, a single price path is highlighted, and expectation/variance appear as visual summaries.
```

为什么展示：

- 建立在已有 legacy showcase subject 之上。
- 很适合概率直觉。
- 视觉丰富但数学扎实。

关键节拍：

- 一团 paths 变成围绕 highlighted trajectory 的 shaded probability distribution。

## 推荐优先制作的前三个

1. The Pipeline Becomes a Scene：最能表达 launch thesis。
2. The Agent Learns a Tangent：最清楚的教育 aha moment。
3. Attention as Moving Light：最契合 Hermes/AI 受众。

## Launch GIF 成功标准

只有全部通过时，新的 GIF 才值得 featured：

- 没有音频也能理解核心概念。
- Loop 有一个明显 aha moment。
- 文本在 README 和 Twitter 尺寸下可读。
- Palette 与 dark neon M2M2 visual language 一致。
- Generated run 有 manifest 和保存下来的 artifact trail。
- Media 被视觉检查过，而不只是渲染成功。
- README/showcase description 解释教学瞬间。

## Launch narrative 中可展示的命令

Hermes setup:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
hermes setup
hermes doctor
hermes tools list --summary
hermes skills list
```

启动 repo-aware Hermes session:

```bash
cd M2M2
hermes --skills agents-md,manim-video,codebase-inspection,systematic-debugging
```

One-shot repo inspection:

```bash
hermes -z "Inspect this M2M2 repo and verify the README, AGENTS.md, pyproject entry points, and CLI smoke command agree." \
  --skills codebase-inspection,agents-md
```

M2M2 deterministic smoke:

```bash
./.venv/bin/python -m pytest
./.venv/bin/python -m math_to_manim.cli generate "Explain why derivatives are slopes" --deterministic --no-render --runs-dir /tmp/m2m2-smoke
```

安装依赖后的 render path:

```bash
python -m pip install -e ".[dev,render]"
./scripts/bootstrap-render.sh
m2m2 generate "Create a cinematic Manim explainer showing the M2M2 pipeline becoming a scene" --quality l
```

GIF extraction recipe:

```bash
ffmpeg -y -ss 8 -t 12 -i "$MP4" \
  -vf "fps=12,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=96[p];[s1][p]paletteuse=dither=bayer:bayer_scale=5" \
  docs/showcase/assets/hermes-learns-manim-pipeline.gif
```

## Guardrail

除非 run 确实执行过且 resulting media 已被检查，否则不要声称“Hermes rendered it”。Launch 应强调可验证 tooling：exact commands、artifacts、tests、renders、visual review 和 committed assets。
