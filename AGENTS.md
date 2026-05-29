# AGENTS.md

本文件是给在本仓库工作的 AI coding agents 使用的最佳实践说明。把它视为 `README.md` 的仓库专用伴随文档：`README.md` 面向人类讲产品故事；这里面向 agents 规定操作 contract。

## 项目概览

M2M2 是 Math-To-Manim 的重写：简短教育 prompt 会变成类型化 planning artifacts、生成的 Manim code、可选渲染、review outputs，以及可复现 run bundle。

核心承诺：先讲故事，再写符号；先做几何，再做代数；先落产物，再产生副作用。

Primary package: `math_to_manim`.
Primary CLI entry points: `m2m2` and `math-to-manim`.
Primary runtime path: `math_to_manim/pipeline/runner.py`.
Architecture reference: `docs/ARCHITECTURE.md`.
Human-facing landing page: `README.md`.

## Agent 操作原则

每次修改都遵循这些 Karpathy-inspired rules：

1. 编码前先思考。
   - 不要静默假设 requirements、architecture、file ownership 或 command behavior。
   - 当歧义会改变实现选择时，明确指出。
   - 只有真正被阻塞时才问澄清；否则选择最小安全解释并说明假设。
   - 当请求有明显复杂度、安全或产品含义时，说明 tradeoffs。

2. 简单优先。
   - 优先选择能满足请求的最小可维护改动。
   - 除非被要求，不要添加 speculative abstractions、宽泛 configurability、background services、新 frameworks 或“future-proofing”。
   - 如果方案变大，先停下寻找更小切口。

3. 外科式修改。
   - 只触碰任务必需的 files 和 lines。
   - 不要顺手重写 comments、formatting、docs 或相邻 code。
   - 匹配正在编辑文件的现有风格。
   - 只有当你的改动让 imports/functions/files 变成 unused，或用户明确要求 cleanup 时，才删除它们。
   - 无关 dead code 在 final notes 中提到，不要擅自删除。

4. 目标驱动执行。
   - 编辑前定义 success criteria。
   - 对 bugs，实践可行时先复现失败或加 failing test。
   - 对 features，实践可行时围绕行为变更加或更新 tests。
   - Final response 前用精确 commands 验证。

## 仓库布局

- `math_to_manim/agents/` — intent、graph、curriculum、math、storyboard、scene spec、codegen、static review、render、video review 和 publishing 的 stage adapters。
- `math_to_manim/schemas/` — Pydantic artifact contracts。把它们视为 public pipeline interfaces。
- `math_to_manim/pipeline/` — orchestration、tracing、state 和 repair loop 行为。
- `math_to_manim/tools/` — graph work、AST/static validation、scene discovery 和 artifact storage 的 deterministic helpers。
- `math_to_manim/rendering/` — Manim、FFmpeg 和 render command wrappers。
- `math_to_manim/providers/` — provider-specific integrations，例如 Codex CLI bridge。
- `math_to_manim/app/` — 可选 API/UI surfaces。
- `tests/unit/` — 当前自动化 test suite。
- `docs/` — architecture、docs index、showcase 和 visual documentation assets。
- `docs/showcase/assets/` — 有意跟踪的 legacy showcase GIFs，用作 art-direction targets。
- `scripts/` — render dependency bootstrap 等 operational helper scripts。
- `runs/` — generated run bundles；已 ignored，通常不 commit。

## Setup commands

存在本地 virtual environment 时优先使用：

```bash
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

macOS/Linux/WSL fresh checkout：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

Windows PowerShell fresh checkout：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

只有任务需要真实 Manim rendering 时才安装 render extras：

```bash
python -m pip install -e ".[dev,render]"
./scripts/bootstrap-render.sh  # Debian/Ubuntu/WSL system deps: FFmpeg, LaTeX, etc.
```

## Verification commands

结束前运行最快的相关检查。优先使用 venv-qualified form，避免依赖 shell activation：

```bash
./.venv/bin/python -m pytest
./.venv/bin/python -m math_to_manim.cli --help
./.venv/bin/python -m math_to_manim.cli generate --help
./.venv/bin/python -m math_to_manim.cli generate "Explain why derivatives are slopes" --deterministic --no-render --runs-dir /tmp/m2m2-smoke
```

如果 CLI entry points 已安装在 active environment，以下等价命令也应可用：

```bash
m2m2 generate "Explain why derivatives are slopes" --deterministic --no-render
math-to-manim generate "Explain why derivatives are slopes" --deterministic --no-render
```

对 codegen-provider 工作，在责怪 M2M2 前单独验证 Codex：

```bash
codex --version
codex exec "Say ready from inside this repo"
```

对 render 工作，只有在 render dependencies 已安装后才运行小型 render-quality smoke。如果 full render 太慢或不可用，运行 deterministic no-render 和相关 unit tests，并明确报告跳过 render 的原因。

## Pipeline contracts

正常生成会在 `runs/<run_id>/` 下写 run bundle，包含：

- `request.json`
- `intent.json`
- `knowledge_graph.json`
- `curriculum.json`
- `math_packet.json`
- `storyboard.json`
- `scene_spec.json`
- `generated_code.json`
- `generated_scene.py`
- `validation_report.json`
- `render_result.json`
- `review_report.json`
- `animation_package.json`
- `manifest.json`

规则：

- 除非任务明确是 schema/pipeline migration，否则保留 artifact names。
- 如果修改 schema，更新所有依赖它的 producers、consumers、tests 和 docs。
- Deterministic mode 必须保持 offline 且 reproducible。
- Rendering 必须由 static validation gate 控制；validation failure 不应调用 Manim。
- Repair loops 应基于冻结的 upstream `scene_spec` 和记录的 stderr/stdout 运行，而不是重新跑完整 planning。

## Code style 和架构

- Python 3.10+。
- 使用 Pydantic models 表达 artifact boundaries。
- Provider-specific behavior 留在 stage runners/providers 后面；不要把 OpenAI、Anthropic、Gemini、Kimi 或 Codex 假设泄漏到 schemas 中。
- 对 validation、graph operations、filesystem packaging 和 command construction，优先使用 pure functions 和 deterministic helpers。
- Stage outputs 应保持为可检查 JSON。
- Errors 要 actionable：可用时包含 command、artifact path、stderr summary 和 stage。
- 避免 pipeline runner 中的 hidden parallelism；文档化 runtime shape 是 single-threaded and ordered。
- 不要绕过 static review 让 rendering “work”。修复 generated code 或 validator contract。

## Testing guidance

- 行为变更要在 `tests/unit/` 中添加或更新 tests。
- Schema 变更要测试 serialization/validation，并至少测试一个 pipeline consumer。
- CLI 变更要测试 argument parsing 或运行 CLI smoke command。
- Provider 变更尽量 mock subprocess/network boundaries；unit tests 不应要求真实 subscription credentials。
- Render 变更应尽量把 command construction 和 result parsing 从实际 Manim execution 中隔离。

## Generated files 和 assets policy

默认不要 commit：

- `.venv/`, `venv/`
- `.env`, `.env.*`
- `.pytest_cache/`, `.ruff_cache/`, `.mypy_cache/`
- `runs/`, `.tmp-runs/`
- `media/`, `output/`, `artifacts/`
- generated `*.mp4`、logs、temporary contact sheets，或临时生成的 GIFs/PNGs。

有意跟踪的例外：

- `docs/showcase/assets/*.gif` — 原始 Math-To-Manim repo 的 curated legacy showcase GIFs。它们是 art-direction targets，不是当前 rewrite outputs。

触碰 showcase media 时：

- 验证 asset 本地存在，且不是 blank/broken placeholder。
- 对新 images/GIFs，优先 visual inspection 或 representative frame/contact-sheet inspection。
- README/showcase links 已指向的文件名保持稳定。
- 改变 gallery membership 时，同时更新 `README.md` 和 `docs/showcase/README.md`。

## Security 和 secrets

- 永远不要 commit credentials、tokens、API keys、auth headers、`.env` contents 或 connection strings。
- 不要在 logs、docs、commits、PR bodies 或 final responses 中打印 secret values。
- 文档示例使用 `OPENAI_API_KEY="***"` 这样的 placeholder。
- 如果命令需要本地 credentials，依赖用户现有环境，并以 redacted 形式报告 values。
- Generated Manim code 不应读取任意本地文件、意外 shell out、访问网络资源，或写出 run directory 之外，除非明确设计并经过 review。

## Hermes skill workflow

本仓库适合 skill-driven Hermes/Codex work。Hermes 是 contributor tooling，不是 M2M2 runtime dependency。用它检查、计划、测试、调试、审阅和协调改动，同时保留 typed pipeline contracts。

### Hermes 应如何使用本仓库

Hermes 是 M2M2 周围的 workspace operator，不是 Python package 的一部分。用 Hermes-native tools 操作 repo-local surfaces：

- 使用 file/search tools 根据 `README.md`、`AGENTS.md`、`pyproject.toml`、`docs/`、`math_to_manim/` 和 `tests/` 证实 claims。
- 使用 patch tools 做 targeted edits；除非任务明确要求，避免 broad rewrites。
- 使用 terminal tools 做 setup、`pytest`、CLI help、deterministic smoke runs、Codex checks、render checks、FFmpeg/GIF commands 和 git verification。
- 使用 vision tools 检查 rendered frames、contact sheets、screenshots 和 GIF quality。
- 多文件工作涉及 schemas、CLI、docs、tests、render behavior 或 media assets 时，可使用 delegation/subagents 分开 review。
- 使用 todos/plans/session notes 记录 acceptance criteria、run IDs、artifact paths、skipped checks 和 rollback notes。
- Session search/memory 只用于稳定 repo decisions；不要保存 secrets、temporary run noise 或 user credentials。
- 用 skills 加载流程：`agents-md` 用于本文件，`codebase-inspection` 用于 claims，`manim-video` 用于 animation quality，`systematic-debugging` 用于 failing runs/renders，`writing-plans` 用于较大改动，`test-driven-development` 用于行为变更。

把这些 tools 映射到 M2M2 artifacts：`m2m2` / `math-to-manim` CLI、`math_to_manim/tools/` 中的 deterministic helpers、`math_to_manim/pipeline/` 中的 pipeline code、`math_to_manim/schemas/` 中的 schemas，以及包含 JSON artifacts、`generated_scene.py`、reports、contact sheets/frames 和 `manifest.json` 的 generated `runs/<run_id>/` bundles。

### 安装并验证 Hermes

Linux/macOS/WSL2：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
hermes setup
hermes doctor
hermes tools list --summary
hermes skills list
```

Native Windows 不是 Hermes repo work 的首选路径。从 Windows 上处理这个 checkout 时，请使用 WSL2。

### 为 M2M2 工作启动 Hermes

按任务预加载最小 skill set：

```bash
# General repo inspection / docs accuracy.
hermes --skills codebase-inspection

# Agent instructions and launch docs.
hermes --skills agents-md,codebase-inspection

# Animation concepting, render/GIF work, and visual quality review.
hermes --skills manim-video,systematic-debugging,codebase-inspection

# Larger pipeline or schema work.
hermes --skills writing-plans,test-driven-development,codebase-inspection

# Debugging CLI, schema, provider, render, or generated-code failures.
hermes --skills systematic-debugging,codebase-inspection

# Coordinated multi-agent implementation.
hermes --worktree --skills subagent-driven-development,writing-plans

# Pre-commit review for risky changes.
hermes --skills requesting-code-review,codebase-inspection
```

Scripted checks 的 single-shot form：

```bash
hermes -z "Inspect this M2M2 repo and verify the README, AGENTS.md, pyproject entry points, and CLI smoke command agree." \
  --skills codebase-inspection,agents-md
```

### 本仓库 Skill map

- `agents-md` — 更新本文件或其他 agent operating instructions。
- `manim-video` — 设计、批评、加固、渲染并 GIF-export Manim explanations。
- `codebase-inspection` — 根据 `pyproject.toml`、CLI help、tests、docs 和实际文件验证 claims。
- `writing-plans` — 规划 feature work、schema migrations、provider changes、render behavior changes 和 docs restructures。
- `test-driven-development` — 围绕 stage adapters、schemas、CLI flags、static validation 和 repair loops 增加行为。
- `systematic-debugging` — 诊断 deterministic runs、model-backed runs、Codex CLI provider calls、Manim renders 和 artifact handoffs 的 failures。
- `subagent-driven-development` — 把较大任务拆给 file-boundary-safe workers，例如 schemas/tests 一组、CLI 一组、docs 一组。
- `requesting-code-review` — 在提交 schema/provider/security/render changes 前请求 review。
- `github-pr-workflow` — 用户要求时 commit、push、create/update PRs，并验证 remote state。
- `codex` — 专门处理 Codex CLI-backed codegen provider 或 Codex developer workflow 时使用。

### 如果缺少 skill

安装新内容前先检查和 inspect：

```bash
hermes skills list
hermes skills search <query>
hermes skills inspect <identifier>
hermes skills install <identifier>
hermes skills audit
```

除非用户明确要求，不要把 Hermes skills vendor 到本仓库。不要 commit local skill caches 或 session-only plans。

### Hermes-specific pitfalls

- 正常 repo work 不要使用 `--ignore-rules`；它会跳过 `AGENTS.md`，可能绕过本 operating contract。
- 避免 `--yolo`，除非用户明确接受风险；它会绕过 dangerous-command approval prompts。
- 并行 agents 可能编辑重叠文件时，优先使用 `--worktree`。
- Hermes credentials 位于 repo 外，通常在 `~/.hermes/`；绝不要复制到 `.env`、docs、commits、logs 或 PR text。
- M2M2 model credentials 如 `OPENAI_API_KEY` 只能以 redacted placeholders 展示。
- Generated `runs/`、Manim `media/`、temporary renders、contact sheets 和临时 GIFs/PNGs 不应 commit，除非用户明确要求 curated docs asset。
- 不要混淆 Hermes skills 和 M2M2 runtime dependencies。Package dependencies 定义在 `pyproject.toml`；M2M2 code 不应 import Hermes。

### Hermes planning files

明确有用时，Hermes planning files 可以放在 `.hermes/plans/`。Plan 应包含 task scope、skills used、expected file changes、artifact/schema contracts affected、acceptance criteria、verification commands、known risks 和 rollback notes。

除非用户要求保存，不要 commit stale 或 session-only plans。

## Documentation rules

- 保持 `README.md` polished 且 human-facing。
- 保持 `AGENTS.md` operational 且 agent-facing。
- 保持 `docs/ARCHITECTURE.md` 与实际 runtime behavior 对齐。
- 保持 `docs/showcase/README.md` 与本地 showcase assets 对齐。
- 如果 README 和 AGENTS 中的 commands 分歧，检查 code/config 并解决 mismatch，不要猜。

## Git 和 PR guidance

- 在 focused branch 上工作。
- Commits 保持 scope 对准请求。
- 使用 conventional commit messages，例如 `docs: add agent operating guide` 或 `fix: repair deterministic pipeline smoke`。
- Commit 前运行相关检查并检查：

```bash
git status --short
git diff --check
git diff --stat
```

- 对 GitHub-visible README/docs/assets changes，用户要求时 push 并验证 remote branch/PR。
- Final status 报告 exact commands run、changed files，以及任何 skipped checks 和原因。

## Stop conditions

遇到以下情况，停下询问或升级：

- Requirements 与现有 pipeline contracts 冲突。
- 改动需要 commit secrets、大型 generated media 或 local-only artifacts。
- Tests 因与你的改动无关的原因失败，且修复需要 broad refactoring。
- 因缺少本地依赖，无法验证请求的 render 或 media change。
