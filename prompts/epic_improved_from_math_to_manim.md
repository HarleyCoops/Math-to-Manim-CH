创建一段 75 秒的电影感 Manim 动画，标题为“线性与非线性系统：小原因何时保持小，何时爆炸”。

受众：高中物理学生。

从两个并排系统开始。左侧展示一个线性弹簧-质量振子，包含胡克定律 \(F=-kx\)、平滑正弦运动和成比例响应：初始位移加倍，振幅也加倍。右侧展示一个非线性受迫摆或 logistic map，其中初始条件的微小差异会增长成明显不同的未来。

使用清晰、完整格式化的 LaTeX 方程：
\[
F=-kx
\]
\[
x(t)=A\cos(\omega t+\phi)
\]
\[
x_{n+1}=rx_n(1-x_n)
\]
\[
|\Delta x(t)|\approx |\Delta x(0)|e^{\lambda t}
\]

在每个系统中动画展示两个几乎相同的初始条件。在线性情况中，让 trajectories 保持接近且可预测。在非线性情况中，展示它们分离、分叉并形成 chaotic pattern。使用颜色编码：蓝色表示 linear stability，橙色/红色表示 nonlinear sensitivity。结尾给出 takeaway：“线性系统会按比例缩放；非线性系统可能带来惊讶。”
