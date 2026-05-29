# M2M2 文档

本目录记录 Codex/OpenAI Agents SDK 重构相关的主要文档。

## 文档

- [架构](ARCHITECTURE.md)：说明目标 agent 管线、运行形态和 worker 边界。
- [路线图](ROADMAP.md)：说明当前可编辑视频工作流状态，以及计划中的 prompt/spec/code 迭代循环。
- [部署路线图](DEPLOYMENT_ROADMAP.md)：给出在云端部署 Manim 动画引擎的实用公开指南。
- [动效展厅](showcase/README.md)：展示本地保留的 legacy Math-To-Manim GIF，以及生成场景需要追赶的视觉标准。
- [Artifact schemas](ARTIFACT_SCHEMAS.md)：定义规划、生成、渲染和 eval 阶段之间传递的 JSON/YAML contracts。
- [Eval strategy](EVAL_STRATEGY.md)：解释 prompt、artifact、code 和 render 检查如何配合。
- [Domain skills](DOMAIN_SKILLS.md)：说明物理/数学 skills 如何改进动画直觉、Manim 模式、审阅循环和伦理灵感。
- [迁移笔记](MIGRATION_NOTES.md)：总结从公开 Math-To-Manim 到本次重构的迁移。
- [Prime Intellect RL](PRIME_INTELLECT_RL.md)：说明如何把 run bundle 作为可验证修复任务，接入 Prime Intellect RL。

## 当前 fixtures

- `evals/prompt_suite.yaml` 包含初始 prompt-level eval suite。
- `examples/reference/limit_tangent_reference.py` 是一个小型 Manim CE 参考场景，用于 renderer 和 style sanity checks。

## 中文镜像说明

这个仓库保留代码、命令、路径、artifact 名称和 package 名称不变，只翻译面向人的说明文本。Legacy 历史材料会优先保持原样，除非它们属于当前公开文档入口。
