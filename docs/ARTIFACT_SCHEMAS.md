# Artifact Schemas

Artifacts 是 agents、tools、renderers 和 evals 之间的 contract。它们应是普通 JSON-compatible objects，带版本，并在下一个 stage 运行前持久化。

## Shared Envelope

每个 artifact 使用相同的顶层 metadata。

```yaml
schema_version: "m2m2.artifact.v1"
artifact_type: "scene_spec"
artifact_id: "2026-05-02T140000Z-derivative-scene"
created_at: "2026-05-02T14:00:00Z"
source_run_id: "run_..."
producer:
  stage: "visual_designer"
  model: "provider/model-or-local-tool"
  prompt_hash: "sha256:..."
```

必需规则：

- `schema_version` 只在 breaking changes 时 bump。
- `artifact_type` 必须是下列类型之一。
- `artifact_id` 写入后保持稳定。
- `producer.model` 可以是 `local-tool`，用于 deterministic scripts。

## request_spec

捕获规范化后的 user request。

```yaml
artifact_type: "request_spec"
prompt: "Explain why derivatives are slopes"
audience: "high_school"
duration_seconds: 60
style: "clear"
quality_target: "preview"
constraints:
  render_engine: "manim-ce"
  max_scene_count: 1
  allowed_external_assets: false
```

## concept_plan

定义目标概念和教学目标。

```yaml
artifact_type: "concept_plan"
target_concept: "Derivative as slope of a tangent line"
learning_objectives:
  - "Connect average rate of change to secant slope."
  - "Show tangent slope as the secant limit."
misconceptions:
  - "A tangent line must touch a graph at only one point."
key_terms:
  - "secant line"
  - "tangent line"
  - "limit"
```

## knowledge_tree

表示反向先修知识发现。

```yaml
artifact_type: "knowledge_tree"
root:
  id: "derivative_slope"
  label: "Derivative as slope"
  prerequisites:
    - id: "line_slope"
      label: "Slope of a line"
      prerequisites: []
    - id: "secant_limit"
      label: "Secant lines approaching tangents"
      prerequisites:
        - id: "function_graph"
          label: "Functions and graphs"
          prerequisites: []
depth_limit: 2
ordering: "foundations_to_target"
```

## math_enrichment

存储后续阶段使用的 equations、invariants 和 checks。

```yaml
artifact_type: "math_enrichment"
definitions:
  derivative: "f'(a) = lim_{h -> 0} (f(a+h)-f(a))/h"
equations:
  - id: "difference_quotient"
    latex: "f'(a)=\\lim_{h\\to 0}\\frac{f(a+h)-f(a)}{h}"
    plain_language: "The derivative is the limiting secant slope."
assumptions:
  - "Function is differentiable at the highlighted point."
validation_notes:
  - "Use h values that approach zero from the right in the visual."
```

## visual_spec

描述视觉计划，但不包含可执行代码。

```yaml
artifact_type: "visual_spec"
canvas:
  aspect_ratio: "16:9"
  background: "dark"
visual_elements:
  - id: "graph"
    type: "axes_plot"
    expression: "0.25*x**2 + 0.5"
  - id: "secant_line"
    type: "line"
    relation: "passes through graph at x=a and x=a+h"
beats:
  - id: "introduce_average_slope"
    duration_seconds: 12
    focus: ["graph", "secant_line"]
```

## narrative_spec

定义 narration、captions 和 pacing。

```yaml
artifact_type: "narrative_spec"
tone: "precise"
beats:
  - id: "introduce_average_slope"
    narration: "Start with the slope between two nearby points."
    on_screen_text: "Average slope"
    math_refs: ["difference_quotient"]
```

## scene_spec

Code generation 前的最终 implementation-neutral contract。

```yaml
artifact_type: "scene_spec"
scene_id: "derivative_slope_intro"
scene_class_name: "DerivativeSlopeIntro"
manim_version_target: "CE >=0.18"
imports:
  - "from manim import *"
sections:
  - id: "setup"
    objective: "Show axes, graph, and secant line."
    required_mobjects: ["Axes", "MathTex", "Line", "Dot"]
  - id: "limit"
    objective: "Animate h decreasing until the secant appears tangent."
    required_animations: ["Create", "Transform", "FadeIn"]
acceptance_checks:
  - "One Scene subclass exists with the requested class name."
  - "No network or filesystem writes are used."
  - "MathTex strings are valid LaTeX fragments."
```

## manim_artifact

存储 generated code 和 static validation。

```yaml
artifact_type: "manim_artifact"
scene_class_name: "DerivativeSlopeIntro"
source_path: "generated/derivative_slope_intro.py"
code_hash: "sha256:..."
static_validation:
  syntax_ok: true
  forbidden_imports: []
  scene_classes: ["DerivativeSlopeIntro"]
```

## render_artifact

存储 render output metadata。

```yaml
artifact_type: "render_artifact"
command: "python -m manim -ql generated/derivative_slope_intro.py DerivativeSlopeIntro"
status: "passed"
media:
  video_path: "media/videos/derivative_slope_intro/480p15/DerivativeSlopeIntro.mp4"
  preview_image_path: "media/images/derivative_slope_intro.png"
duration_seconds: 58.4
stderr_summary: ""
```

## study_notes_artifact

捕获配套学习材料。

```yaml
artifact_type: "study_notes_artifact"
formats:
  markdown_path: "generated/derivative_slope_intro.md"
  latex_path: "generated/derivative_slope_intro.tex"
outline:
  - "Average slope"
  - "Limit of secants"
  - "Derivative notation"
```

## eval_record

总结自动化和 human-reviewable 质量信号。

```yaml
artifact_type: "eval_record"
suite: "m2m2_prompt_refactor_v1"
case_id: "derivative_slope_intro"
status: "passed"
scores:
  schema_valid: 1.0
  pedagogy: 0.86
  visual_feasibility: 0.92
  manim_static: 1.0
  render: 1.0
failures: []
```

## Compatibility Notes

- Artifacts 应能序列化为 JSON，也能作为 YAML fixtures 阅读。
- Generated Manim code 通过 path 和 hash 引用；除非 runner 明确需要 inline review，否则不嵌入 eval records。
- Schema 避免 provider-specific prompt fields，因此 Anthropic、Gemini、Kimi、OpenAI 和本地 deterministic stages 都能产出相同 shape。
