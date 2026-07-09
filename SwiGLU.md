SwiGLU FFN 完整公式
单层 SwiGLU（Qwen3.5 每个 expert 内部）：
$$
\text{Output} = \mathbf{W}_d \cdot \Big( \underbrace{\text{SiLU}(\mathbf{W}_g \cdot \mathbf{x})}_{\text{门控}} \;\odot\; \underbrace{(\mathbf{W}_u \cdot \mathbf{x})}_{\text{升维}} \Big)
$$
- $\mathbf{W}_g \in \mathbb{R}^{d_{\text{ff}} \times d_{\text{model}}}$ — gate_proj
- $\mathbf{W}_u \in \mathbb{R}^{d_{\text{ff}} \times d_{\text{model}}}$ — up_proj
- $\mathbf{W}_d \in \mathbb{R}^{d_{\text{model}} \times d_{\text{ff}}}$ — down_proj
- $\odot$ — 逐元素乘（Hadamard product）
- $\text{SiLU}(z) = z \cdot \sigma(z) = \dfrac{z}{1 + e^{-z}}$（sigmoid 线性单元）
合并后的 gate_up_proj（transformers v5）：
$$
\mathbf{W}_{\text{gu}} = \begin{bmatrix} \mathbf{W}_g \\ \mathbf{W}_u \end{bmatrix} \in \mathbb{R}^{2d_{\text{ff}} \times d_{\text{model}}}, \quad \mathbf{z} = \mathbf{W}_{\text{gu}} \cdot \mathbf{x} \in \mathbb{R}^{2d_{\text{ff}}}
$$
$$
\text{Output} = \mathbf{W}_d \cdot \Big( \underbrace{\text{SiLU}(\mathbf{z}_{[:d_{\text{ff}}]})}_{\text{前半是 gate}} \;\odot\; \underbrace{\mathbf{z}_{[d_{\text{ff}}:]}}_{\text{后半是 up}} \Big)
$$
MoE 中（Qwen3.5 全部 128 个 expert）：
$$
\mathbf{W}_{\text{gu}}^{\text{all}} \in \mathbb{R}^{128 \times 2d_{\text{ff}} \times d_{\text{model}}}, \quad \mathbf{W}_d^{\text{all}} \in \mathbb{R}^{128 \times d_{\text{model}} \times d_{\text{ff}}}
$$
$$
\text{AllExpertOutputs} = \text{grouped\_mm}(\mathbf{W}_d^{\text{all}},\; \text{SiLU}(\text{grouped\_mm}(\mathbf{W}_{\text{gu}}^{\text{all}},\; \mathbf{x})) \odot \dots)
$$
然后 router 从 128 个 expert 输出中选 top-8，加权求和：
$$
\text{Final} = \sum_{i \in \text{TopK}(r(\mathbf{x}),\; k=8)} g_i \cdot \text{Expert}_i(\mathbf{x})
$$
其中 $r(\mathbf{x})$ 是 router 网络，$g_i$ 是 softmax 归一化后的 router 权重。
一句话：gate 决定「哪些维度重要」，up 负责「往上投影」，门控后的结果通过 down 回到原始维度。MoE 把这个过程复制了 128 份，每次只激活最相关的 8 份。