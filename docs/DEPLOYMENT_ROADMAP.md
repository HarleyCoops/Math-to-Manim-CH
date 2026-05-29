# Manim 动画引擎部署路线图

这份路线图面向想部署 Math-To-Manim 类服务的团队：用户提交教育 prompt，系统规划解释、生成 Manim 代码、在隔离 worker 中渲染视频，并返回可检查的 run bundle。这是可复用的实现指南，不是托管支持服务。

## 目标形态

从朴素、可检查的架构开始：

```text
browser or API client
  -> API service
  -> database row for the job
  -> queue
  -> render worker sandbox
  -> object storage
  -> status/result API
```

即便一开始所有阶段都在一个 worker process 内运行，也要把 planning、code generation、validation、rendering 和 publishing 保持为分离阶段。核心产品 contract 应该是 artifacts，而不是 side effects：prompt、plan、generated scene、validation report、render result、review notes、final video 和 manifest。

## 架构选择

- API service：FastAPI、Django、Rails 或 Node 都可以。选择团队已经能稳定运维的技术栈。
- Queue：优先使用托管队列，如 SQS、Cloud Tasks、Pub/Sub 或 hosted Redis queue。渲染不能运行在 request/response path 中。
- Workers：把 Manim、Python dependencies、FFmpeg、LaTeX、fonts 和 engine code 打进固定版本 container image。
- Database：存储 job state、ownership、prompt metadata、artifact keys、retry counts、timestamps，以及需要时的 billing 或 quota metadata。
- Object storage：把 run bundles 和 videos 存在 S3、GCS、Azure Blob、R2 或类似存储中。不要把大视频存在数据库。
- UI：轮询或订阅 job state，显示 stage progress，安全暴露 logs，并在 publishing 完成后链接可下载输出。

第一个生产版本只需要一个 API service、一个 queue、一个 worker image、一个 database 和一个 storage bucket。

## Job 生命周期

使用明确状态，让失败可支持、可排查：

1. `queued`：请求已接受，基础限制已检查，job 已持久化。
2. `planning`：prompt 变成 intent、graph、curriculum、storyboard 和 scene spec artifacts。
3. `codegen`：scene spec 变成 `generated_scene.py`。
4. `validating`：Manim 前运行 AST/import/scene discovery checks。
5. `rendering`：sandboxed worker 调用 Manim，并捕获 stdout/stderr。
6. `reviewing`：可选的视频 probes、frame checks 或 model review。
7. `published`：manifest、video、thumbnails 和 reports 已存储。
8. `failed`：error summary、stage、command 和相关 artifact paths 已存储，用于调试。

Retries 应该感知 stage。Render retry 应复用冻结的上游 scene spec 和捕获到的 render error，而不是重新跑完整 planning。

## 沙箱和安全

生成的 Manim 代码是不受信任代码。把 render worker 当成 containment boundary：

- 每个 job 在 fresh container、Firecracker microVM、gVisor sandbox 或类似隔离环境中运行。
- 除非经过审查的功能需要，否则禁用 render jobs 的 outbound network access。
- 挂载 job-specific working directory，并只允许在该目录内写 output。
- 使用 non-root user、read-only base image layers、CPU 和 memory limits、process limits、timeout limits 和 disk quotas。
- Secrets 只传给需要它们的 API 或 model-call 阶段。Render sandboxes 默认不应接收 provider keys。
- 渲染前验证生成代码。拦截明显 unsafe imports、job directory 外的 filesystem writes、subprocess calls、network calls 和 dynamic execution patterns。
- 存储 logs 时进行 secret redaction。绝不要在 UI logs 中暴露原始 environment variables 或 provider credentials。

对于高风险 public upload 或 arbitrary-code 场景，优先使用 VM-level isolation，而不是 plain Docker。

## 渲染依赖

Manim 渲染不只是一个 Python package。Worker image 通常需要：

- Python 和带 render extras 安装的项目 package。
- 固定版本的 Manim Community Edition。
- 用于视频输出和后处理的 FFmpeg。
- 用于 `MathTex` 和公式密集场景的 LaTeX 与 `dvisvgm`。
- Cairo、Pango、fontconfig 和 system fonts。
- 只有当 scenes 或 post-processing 真正使用 GPU 时，才加入可选 GPU libraries。

构建一次 image，在 image validation 中运行一个小 deterministic scene，并按 digest 发布 image。避免在 job runtime 安装系统渲染依赖。

## API 和 UI Surface

最小 API：

- `POST /jobs`：从 prompt、style、quality 和 render options 创建 job。
- `GET /jobs/{id}`：返回 state、current stage、timestamps 和 safe errors。
- `GET /jobs/{id}/artifacts`：列出用户可访问的 manifest entries。
- `GET /jobs/{id}/download`：返回 video 和选定 reports 的 signed URLs。
- `POST /jobs/{id}/cancel`：在渲染前或渲染中请求取消。

最小 UI：

- Prompt form，清楚说明 quality 和 render-time tradeoffs。
- 带 stage progress 的 job status page。
- Final video playback、download links 和 artifact/report links。
- Failure page，解释失败 stage，但不泄露 internals 或 secrets。

如果暴露 generated code，请明确标注它是 generated，并且只在 sandboxed worker path 中运行。

## 可观测性

捕获足够细节，以便不用 shell 进 worker 也能回答“发生了什么”：

- Job id、user id 或 tenant id、stage、status、timestamps、duration、attempt。
- Queue wait time、render time、total wall-clock time、CPU 和 memory usage。
- Container image digest 和 project version。
- Manim command、exit code、stderr summary 和 artifact paths。
- 适用时记录 model provider、model name、token counts 和 cost metadata。
- 每个 stage transition 的 structured events。

Dashboards 应跟踪 queue depth、worker saturation、failure rate by stage、timeout rate、median 和 p95 render time、storage growth，以及 cost per completed video。

## 成本和扩展说明

渲染是 bursty 且 CPU-heavy 的。先规划 backpressure，再扩展：

- 从 fixed-size workers 和严格 per-job timeouts 开始。
- 根据 queue depth 和 oldest-message age 增加 autoscaling。
- 限制 quality presets。低质量 preview renders 比最终高质量 renders 便宜得多。
- 缓存 base images 和 reusable assets，但不要跨用户缓存 untrusted job workspaces。
- 使用 lifecycle policies 过期 temporary artifacts、logs 和 preview media。
- 如果 final renders 会阻塞快速反馈，拆分 preview 和 final queues。
- 围绕 prompts、concurrent jobs、render minutes、storage 和 retries 设置 quotas。

大多数团队应该先横向扩展 workers，再考虑 GPUs 或自定义 render orchestration。

## 实用上线计划

1. Local engine：deterministic no-render jobs 创建 typed artifacts 和 manifest。
2. Local render：一个 trusted scene 使用生产中相同的 worker command 渲染。
3. Container image：render dependencies 在 CI 中固定并验证。
4. Private queue：API 创建 jobs，一个 worker 消费 jobs，object storage 接收 bundles。
5. Sandbox hardening：强制执行 network、filesystem、process、memory、CPU 和 timeout limits。
6. Public beta：启用 quotas、cancellation、safe error messages 和 artifact expiry。
7. Production hardening：完成 autoscaling、dashboards、alerting、abuse controls、cost reporting 和 incident runbooks。

## 生产就绪清单

- Jobs 绝不在 API requests 内同步渲染。
- Generated code 在 render 前验证，并且只在 sandbox 中执行。
- Workers 默认无法访问 model provider secrets。
- 每个 job 都写 manifest，并在需要时写 stage-specific failure record。
- Render dependencies 安装在 image 中，而不是 job 运行时。
- Videos 和 run bundles 存在 object storage 中，并通过 signed URLs 访问。
- Queue depth、failures、render duration 和 costs 可观测。
- Timeouts、quotas、cancellation、retries 和 artifact retention 都显式定义。
- 每次 worker image release 都运行一个小 deterministic render smoke test。
