---
name: hermes-learns-manim
description: "用 Hermes 操作 Math-To-Manim：检查 artifacts、运行 deterministic checks、使用 Codex-backed codegen、渲染/审阅 Manim outputs，并避免提交 generated media。"
version: 1.0.0
author: HarleyCoops + Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [math-to-manim, manim, m2m2, hermes, codex, animation, artifacts]
---

# Hermes 学习 Manim

当用 Hermes 操作 Math-To-Manim repo 来检查、生成、验证、渲染或审阅教育 Manim animations 时，使用这个 skill。

## Operating Contract

- 编辑前阅读 `README.md` 和 `AGENTS.md`。
- 把 Hermes 视为 contributor tooling，而不是 Python runtime dependency。
- 保持 M2M2 artifacts 在 `runs/<run_id>/` 下可检查。
- 保留 pipeline contract：story before symbols、geometry before algebra、artifacts before side effects。
- 在 model-backed 或 render-heavy runs 前，优先使用 deterministic no-render checks。
- 不要 commit generated `runs/`、`media/`、temporary renders、secrets 或 local caches。
- 对 showcase/media changes，声称成功前要 visually inspect representative frames/GIFs。

## Quick Verification

从 repo root 运行：

```bash
./.venv/bin/python -m math_to_manim.cli --help
./.venv/bin/python -m math_to_manim.cli generate --help
./.venv/bin/python -m math_to_manim.cli generate "Explain why derivatives are slopes" --deterministic --no-render --runs-dir /tmp/m2m2-smoke
```

如果 venv 尚未安装：

```bash
python3 -m venv .venv
./.venv/bin/python -m pip install -U pip
./.venv/bin/python -m pip install -e ".[dev]"
```

对 Codex-backed code generation，先验证本地 Codex CLI：

```bash
codex --version
codex exec "Say ready from inside this repo"
```

## Starter Workflow

1. 检查 `pyproject.toml`、`math_to_manim/cli.py` 和 `math_to_manim/pipeline/runner.py`。
2. 修改行为前运行 CLI help 或 deterministic smoke command。
3. 打开生成的 `runs/<run_id>/` bundle，先检查 JSON artifacts，再修改 downstream code。
4. 对 media work，只有 static validation 通过后才 render，并 visually inspect output。
5. 报告 exact commands、run bundle paths、skipped checks 和 changed files。

## Hermes Registration

注册 parent skills directory，而不是这个 skill directory 本身：

```bash
hermes config set skills.external_dirs "$(pwd)/hermes/skills"
hermes skills list --source local
hermes --skills hermes-learns-manim,agents-md,codebase-inspection,manim-video,systematic-debugging
```

Hermes 会递归扫描 configured external skill directories 中的 `SKILL.md` files。把 `skills.external_dirs` 指向 `hermes/skills` 后，`hermes-learns-manim` 就能按名称发现。

完整 repo instructions 见 `README.md` 和 `AGENTS.md`。
