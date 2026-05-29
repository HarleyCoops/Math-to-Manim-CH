# Eval 策略

M2M2 evals 应衡量从 prompt 到有用动画的整条路径，同时让失败能归因到单个阶段。

## Eval 层

| 层 | 问题 | Gate |
| --- | --- | --- |
| Prompt eval | 管线是否推断出正确教育意图？ | code generation 前必需 |
| Schema eval | Artifacts 是否有效且完整？ | 每个阶段必需 |
| Pedagogy eval | 概念顺序是否从基础走向目标？ | narrative approval 前必需 |
| Visual feasibility eval | 视觉计划能否在 Manim CE 中构建？ | code generation 前必需 |
| Static code eval | 生成的 Python 是否可解析并定义预期 Scene？ | render 前必需 |
| Render eval | Manim 是否无错误生成 media？ | shipping 前必需 |
| Regression eval | 改动后的 prompt 或 stage 是否降低已知 cases？ | package code 存在后在 CI 中必需 |

## 初始 YAML Suite

`evals/prompt_suite.yaml` 是 starter prompt-level suite。它刻意保持 runner-neutral：package owners 之后可以把它绑定到 OpenAI Evals、pytest、Inspect 或自定义 runner。

每个 case 包含：

- `input.prompt`：自然语言请求。
- `expected`：关键概念、artifact requirements 和 disallowed shortcuts。
- `rubric`：grader 可应用到 stage artifacts 的加权 checks。

## 建议执行流程

1. 让 prompt cases 通过 stage pipeline 运行到 `scene_spec`。
2. 对照 schema docs 或已有的 generated JSON Schema 验证每个 artifact。
3. 先用 deterministic checks 评分 pedagogy 和 visual feasibility。
4. 只有 explanation quality 这类主观标准使用 judge model，并在 `eval_record` 中保存完整 judge prompt/version。
5. 调用 Manim 前运行 static Python checks。
6. 常规 CI 使用低质量渲染，只有 release candidates 或 golden examples 使用更高质量。

## Local Runner

无需 Manim 运行 deterministic structural suite：

```bash
./.venv/bin/python -m math_to_manim.cli eval-suite evals/prompt_suite.yaml --runs-dir /tmp/m2m2-evals
```

安装了 render dependencies 且 eval 需要 Manim output 时，加上 `--render --quality l`。Runner 会写正常 run bundles，并检查 artifact completeness、scene-name sanity、generated Python parsing、static validation、render status，以及可选的 `expected.acceptance_terms`。

## 最小 CI Gates

最小 gates 应包括：

- 所有 YAML suites 可解析。
- 每个 generated artifact 都有有效 shared envelope。
- 每个 generated `scene_spec` 命名一个 scene class。
- Generated Manim files 能通过 `python -m py_compile` 解析。
- Reference examples 能用 `python -m manim -ql` 渲染。

## 评分指导

使用分开的 scores，而不是单一混合 pass/fail。一个 scene 可以数学正确但视觉不可行，也可以可渲染但教学很薄。

推荐 score fields：

- `schema_valid`: 0 或 1。
- `concept_coverage`: 0 到 1。
- `prerequisite_ordering`: 0 到 1。
- `visual_feasibility`: 0 到 1。
- `narrative_alignment`: 0 到 1。
- `manim_static`: 0 或 1。
- `render`: 0 或 1。

Shipping threshold：所有 binary checks 通过，没有 critical failures，且主观平均分至少为 0.8。

## OpenAI Eval 对齐

OpenAI 的 agent eval guidance 强调 agent workflows 的可复现 evals，以及 workflow errors 的 trace-level grading。M2M2 应把 run traces 和 artifact IDs 一起存储，这样失败评分能映射回负责的 agent stage。

Source: https://platform.openai.com/docs/guides/agent-evals
