# 从公开 Math-To-Manim 迁移的笔记

这些笔记描述 M2M2 如何从公开 `HarleyCoops/Math-To-Manim` 项目迁移思想，同时避免复制其 provider-specific 形态。

## 观察到的 baseline

公开项目描述了一条管线：从小 prompt 开始，构建 reverse knowledge tree，丰富数学内容，设计视觉，写 narrative，生成 Manim code，验证或修复语法，用 Manim 渲染，并输出 video 或 GIF artifacts。

公开 README 还列出多个实验性 provider 路径：维护中的 Claude/Anthropic pipeline、Gemini/Google ADK pipeline，以及 Kimi/Moonshot swarm-style pipeline。

Source: https://github.com/HarleyCoops/Math-To-Manim

## 继承什么

- 反向先修知识发现仍是核心 pedagogy pattern。
- 输出应同时包含 animation code 和 study notes。
- Generated Manim 必须在 render 前验证。
- Demo prompts 应保持足够小，以展示管线如何扩展 intent。
- Examples 应优先追求数学清晰，而不是视觉过量。

## 改变什么

- Stage outputs 从隐式 in-memory state 变成 versioned artifacts。
- Provider-specific agents 移到通用 stage-runner interface 后面。
- OpenAI Agents SDK primitives 可以表达 specialist agents、deterministic tools、handoffs、guardrails、sessions 和 tracing。
- Evals 从手动 demo inspection 变成一等 fixtures。
- Generated media paths 是 `render_artifact` 中的 metadata，不是主要 source of truth。

## 建议迁移映射

| Public Math-To-Manim idea | M2M2 artifact or stage |
| --- | --- |
| Simple prompt | `request_spec` |
| ConceptAnalyzer | `concept_plan` |
| PrerequisiteExplorer | `knowledge_tree` |
| MathematicalEnricher | `math_enrichment` |
| VisualDesigner | `visual_spec` |
| NarrativeComposer | `narrative_spec` |
| Manim CodeGenerator | `scene_spec` then `manim_artifact` |
| Syntax validation and repair | `static_validation` inside `manim_artifact` |
| Manim render | `render_artifact` |
| Study notes | `study_notes_artifact` |
| Demo inspection | `eval_record` |

## 兼容性风险

- 公开 examples 可能假设 `src/`、`media/` 或 provider-specific demo scripts 这类本地 folder names。除非 package owner 采用这些路径，M2M2 不应保留它们。
- Generated Manim 可能又长又脆。重构应偏好紧凑的 `scene_spec`，然后从这个稳定 contract 重新生成代码。
- Multi-agent demos 如果每次 retry 都重跑所有 stage，可能会隐藏失败。持久化 artifacts 会让 repair 更便宜、更可审计。
- Manim、LaTeX 和 FFmpeg 是外部系统依赖；evals 应在 render failures 中报告环境细节。

## 第一批里程碑

1. 从 `docs/ARTIFACT_SCHEMAS.md` 定义 artifact dataclasses 或 JSON Schemas。
2. 构建能在每个 artifact stage 后停止的 runner。
3. 把 prompt eval suite 通过 `scene_spec` 接到 runner。
4. Render 前加入 static Manim checks。
5. 只有在 media storage policy 确定后，才把一个已渲染 reference scene 提升为 golden example。
