# 华为面试准备 - 核心技术问答

> 本文档涵盖 Transformer 架构、Attention 机制、矩阵运算、LoRA 微调等核心技术，以及其他高频面试问题。

---

## 一、Transformer 架构

### 1.1 基础原理

**Q1: Transformer 的整体架构是怎样的？**

A: Transformer 采用 Encoder-Decoder 结构：

```
输入 → Embedding + Positional Encoding → Encoder (N层)
                                              ↓
                                   Decoder (N层) ← 输出 Embedding
                                              ↓
                                        Linear + Softmax
                                              ↓
                                          输出
```

**Encoder** 每层包含：

- Multi-Head Self-Attention
- Feed Forward Network
- 两个子层都有残差连接 + Layer Normalization

**Decoder** 每层包含：

- Masked Multi-Head Self-Attention（防止看到未来信息）
- Cross-Attention（ encoder 的输出作为 query）
- Feed Forward Network

**Q2: Transformer 为什么用 Layer Normalization 而不是 Batch Normalization？**

A: 主要原因：

1. **序列长度可变**：不同样本的序列长度可能不同，BatchNorm 依赖批次统计量，在变长序列场景下不适用
2. **特征维度独立**：LayerNorm 对每个样本的每个位置独立归一化，不依赖批次信息
3. **训练稳定性**：LayerNorm 的均值和方差在推理和训练时一致，模型更稳定
4. **Transformer 内部维度固定**：d_model=512/768/1024，LayerNorm 沿着特征维度计算，适配 Transformer 的固定维度设计

```python
# LayerNorm
x = (x - mean) / sqrt(variance + eps) * gamma + beta
# 沿着最后一个维度（特征维度）计算均值和方差
```

**Q3: Transformer 中的残差连接有什么作用？**

A: 三个主要作用：

1. **梯度流动**：让梯度直接传递到浅层，防止梯度消失
2. **特征融合**：将上一层的特征与当前层的结果融合，保留更多信息
3. **简化学习**：让网络学习残差而不是完整映射，降低学习难度

**Q4: 为什么 Transformer 需要 Positional Encoding？**

A: Transformer 的 Self-Attention 机制是位置无关的（permutation equivariant），无法区分词语的顺序位置。引入 Positional Encoding 可以：

1. **注入位置信息**：让模型知道词语在序列中的位置
2. **支持任意长度**：通过 sin/cos 函数可以编码任意长度的位置
3. **相对位置建模**：某些编码方式（如相对位置编码）可以学习相对位置关系

```python
# 原始 Transformer 的绝对位置编码
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
```

---

### 1.2 复杂度分析

**Q5: Transformer 的时间和空间复杂度是多少？**

A: 设序列长度为 n，隐藏层维度为 d，attention head 数为 h。

| 操作                 | 时间复杂度       | 空间复杂度                   |
| -------------------- | ---------------- | ---------------------------- |
| Self-Attention       | O(n² · d)      | O(n²) 存储 attention scores |
| Multi-Head Attention | O(n² · d · h) | O(n² · h)                  |
| Feed Forward         | O(n · d²)      | O(d²)                       |

**Q6: 如何解决 Transformer 在长序列上的计算瓶颈？**

A: 主要方法：

1. **稀疏注意力**：Swin Transformer 的滑动窗口attention
2. **线性注意力**：Performer、Linear Transformers，用线性复杂度近似
3. **分块计算**：Longformer 的局部+全局注意力组合
4. **Flash Attention**：IO-aware 的精确注意力计算，通过tiling和累加避免完整attention矩阵存储
5. **层级化**：Hierarchical Transformer，先处理局部再融合全局

**Q7: Flash Attention 的原理是什么？**

A: Flash Attention 通过两大技术实现：

1. **Tiling（分块）**：将 N×N 的 attention 矩阵分成小块逐步计算，GPU 以 SRAM 为单位处理
2. **Recomputation（重计算）**：不存储完整的 attention 矩阵，而是在反向传播时重新计算

```python
# 核心思想：分块计算 attention，然后累加
for block_i in blocks:
    for block_j in blocks:
        # 在 SRAM 中计算小块
        block_attn = softmax(Q[block_i] @ K[block_j].T / sqrt(d))
        # 更新输出和分母
```

**优势**：将空间复杂度从 O(n²) 降低到 O(n)，同时通过更好的内存访问模式提升速度。

---

## 二、Attention 机制

### 2.1 基础概念

**Q8: 什么是 Self-Attention？它的计算过程是怎样的？**

A: Self-Attention 允许序列中的每个位置关注序列中的所有其他位置。

**计算步骤**：

1. **线性投影**：输入 X 通过三个权重矩阵投影得到 Q、K、V

   ```
   Q = X · W_Q
   K = X · W_K
   V = X · W_V
   ```
2. **计算注意力分数**：

   ```
   Attention(Q, K, V) = softmax(Q · K^T / √d_k) · V
   ```
3. **Scale**：除以 √d_k 防止点积值过大导致 softmax 梯度消失
4. **加权求和**：V 根据注意力权重加权求和

**Q9: 为什么要用 Multi-Head Attention？**

A: 多个 attention head 的作用：

1. **多子空间学习**：不同的 head 可以学习关注不同的特征子空间
2. **增强表达能力**：拼接多个 head 的输出，维度扩大 h 倍
3. **稳定训练**：多个 head 提供冗余，降低单一 attention 失效的风险

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # 线性投影
        self.W_Q = nn.Linear(d_model, d_model)
        self.W_K = nn.Linear(d_model, d_model)
        self.W_V = nn.Linear(d_model, d_model)

        # 多头拼接后再次线性投影
        self.fc = nn.Linear(d_model, d_model)

    def forward(self, Q, K, V, mask=None):
        # ... 分头计算再拼接 ...
```

**Q10: Attention 的 Key-Value 记忆机制是怎样的？**

A: 这是一个重要的高级概念：

1. **Query**：当前需要生成的内容（decoder端）
2. **Key-Value**：外部知识库的表示（encoder端输出）

```python
# Cross-Attention: Query 来自 Decoder，K-V 来自 Encoder
attn_output = softmax(Q_enc @ K_dec.T / √d_k) @ V_dec
```

**应用场景**：

- RAG（检索增强生成）：用外部文档的 K-V 记忆回答问题
- 知识图谱：多跳推理时多步 Cross-Attention
- 多模态：图像/音频作为 K-V 记忆，文本作为 Query

### 2.2 注意力变体

**Q11: 介绍几种常见的 Attention 变体？**

A: 主要变体：

| 类型                              | 特点              | 时间复杂度              |
| --------------------------------- | ----------------- | ----------------------- |
| **Scaled Dot-Product**      | 原始 Transformer  | O(n²)                  |
| **Linear Attention**        | 核函数近似        | O(n)                    |
| **Sparse Attention**        | 只计算部分位置    | O(n√n) ~ O(n·logn)    |
| **Flash Attention**         | IO-aware 高效实现 | O(n²) 但实际 O(n) 内存 |
| **Grouped Query Attention** | K/V 头数少于 Q    | GQA，用于 Llama2        |

**Q12: 什么是 Relative Position Embedding？**

A: 相对位置编码关注 token 之间的相对距离而不是绝对位置：

```python
# 相对位置注意力偏置
attention_score = Q @ K.T + Q @ relative_position_bias + bias

# bias 是一个可学习的矩阵，大小为 (n, n) 或 (2k+1, 2k+1)
```

**优势**：

- 可以泛化到训练时未见过的序列长度
- 更直接地建模相对关系

**Q13: Grouped Query Attention (GQA) 和 Multi-Query Attention (MQA) 有什么区别？**

A:

| 类型 | Q 头数 | K/V 头数  | 特点                    |
| ---- | ------ | --------- | ----------------------- |
| MHA  | h      | h         | 原始多头注意力          |
| MQA  | h      | 1         | 所有 Q 共享一组 K/V     |
| GQA  | h      | g (g < h) | Q 分 g 组，每组共享 K/V |

- **MQA**：推理速度快（KV-cache小），但效果可能下降
- **GQA**：MHA 和 MQA 的折中，Llama2 采用

---

## 三、矩阵运算

### 3.1 矩阵运算基础

**Q14: 描述 Self-Attention 的矩阵运算过程？**

A: Self-Attention 的矩阵运算：

```python
import torch
import torch.nn.functional as F

def self_attention(Q, K, V, scale=True):
    """
    Q: (batch, seq_len, d_k)
    K: (batch, seq_len, d_k)
    V: (batch, seq_len, d_v)
    """
    d_k = Q.size(-1)

    # Step 1: 计算注意力分数
    scores = torch.matmul(Q, K.transpose(-2, -1))  # (batch, seq_len, seq_len)

    # Step 2: Scale
    if scale:
        scores = scores / math.sqrt(d_k)

    # Step 3: Softmax
    attn_weights = F.softmax(scores, dim=-1)  # 最后一维做 softmax

    # Step 4: 加权求和
    output = torch.matmul(attn_weights, V)  # (batch, seq_len, d_v)

    return output, attn_weights
```

**Q15: Transformer 中矩阵乘法的计算复杂度分析？**

A:

```
输入: X ∈ R^(batch_size × seq_len × d_model)

1. Q/K/V 投影:
   X @ W_Q ∈ R^(batch × seq × d_model) × (d_model × d_model)
   复杂度: O(batch × seq × d_model²)

2. Attention Score:
   Q @ K^T ∈ R^(batch × seq × d_k) × (batch × d_k × seq)
   复杂度: O(batch × seq² × d_k)

3. Softmax + Weighted Sum:
   attn_weights @ V ∈ R^(batch × seq × seq) × (batch × seq × d_v)
   复杂度: O(batch × seq² × d_v)
```

**Q16: 如何优化大矩阵乘法的性能？**

A:

1. **算子融合**：将多个操作融合为一个 kernel，减少内存访问

   ```python
   # 融合前：多次内存读写
   scores = Q @ K^T
   scores = scores / sqrt(d_k)
   attn_weights = softmax(scores)

   # 融合后：Flash Attention，内存访问大幅减少
   ```
2. **混合精度**：使用 FP16/BF16 加速矩阵运算

   ```python
   with torch.cuda.amp.autocast():
       output = F.scaled_dot_product_attention(Q, K, V)
   ```
3. **算子分块**：将大矩阵分块计算，提高 cache 利用率

   ```python
   # Block-wise matrix multiplication
   for i in range(0, M, block_size):
       for j in range(0, N, block_size):
           C[i:i+block, j:j+block] += A[i:i+block, :] @ B[:, j:j+block]
   ```
4. **Tensor Parallel**：将矩阵按维度切分到多个 GPU
5. **知识蒸馏**：用小模型近似大矩阵运算结果

**Q17: 解释 Attention Score 为什么要除以 √d_k？**

A: 两个原因：

1. **控制方差**：假设 Q 和 K 的每个维度是独立随机变量，均值为 0，方差为 1。则 Q·K^T 的每个元素是 d_k 个独立变量之和，方差为 d_k，标准差为 √d_k
2. **防止 softmax 饱和**：如果方差很大，softmax 的输入值会很大，导致梯度消失

   ```
   当 x >> 0 时，softmax(x) ≈ one_hot(max_idx)
   梯度 ≈ 0，无法有效学习
   ```

除以 √d_k 后，方差恢复到 1，softmax 的输入分布合理，梯度稳定。

**Q18: 矩阵乘法中 CPU 和 GPU 的优化策略有什么区别？**

A:

| 方面               | CPU                | GPU                           |
| ------------------ | ------------------ | ----------------------------- |
| **缓存层级** | L1/L2/L3 多级缓存  | 全局显存 + Shared Memory      |
| **并行方式** | 多线程（OpenMP）   | SIMT（单指令多线程）          |
| **优化重点** | 缓存命中，数据预取 | 线程块划分，显存访问合并      |
| **常用库**   | MKL, OpenBLAS      | cuBLAS, cuDNN, FlashAttention |
| **分块大小** | 64-256             | 16-32（线程块内共享内存）     |

**GPU 特殊优化**：

- **Bank Conflict 避免**：Shared Memory 分区访问优化
- **Warp 级操作**：同一 Warp 内线程协作
- **Tensor Core**：专用矩阵运算单元（FP16/BF16 Matrix Multiply-Accumulate）

---

## 四、LoRA 微调

### 4.1 基础原理

**Q19: 什么是 LoRA？它的原理是什么？**

A: LoRA（Low-Rank Adaptation）是一种参数高效微调方法。

**核心思想**：预训练模型的权重矩阵是过参数化的，存在低秩结构。LoRA 假设对权重 W 的更新 ΔW 也是低秩的。

```python
# 原始全量微调：直接更新 W
W_new = W_0 + ΔW  # 需要微调所有参数

# LoRA：只训练低秩分解的 A 和 B
W_new = W_0 + BA  # 其中 B ∈ R^(d×r), A ∈ R^(r×k), r << min(d,k)
```

**训练时**：

- 冻结 W_0
- 只更新 A 和 B（以及可选的偏置）
- 反向传播只计算 ΔW = BA 的梯度

**推理时**：

```python
# 方式1：合并权重（推理延迟低）
W_effective = W_0 + BA  # 离线合并，单次前向

# 方式2：原位计算（节省显存）
h = W_0 @ x + BA @ x  # 实时计算，需要保持 W_0
```

**Q20: LoRA 的参数规模如何计算？**

A: 假设原始权重 W ∈ R^(d×k)，LoRA 秩为 r。

```
可训练参数：
- A: d × r（使用随机高斯初始化）
- B: r × k（初始化为零）

总参数：d×r + r×k = r(d+k)

推理时不增加额外计算：
h = W_0 x + BA x = (W_0 + BA)x
```

**示例**：d=4096, k=4096, r=8

```
LoRA 参数：8 × (4096 + 4096) = 65536 ≈ 65K 参数
原始参数：4096 × 4096 = 16M 参数
压缩比：99.6% 参数不更新
```

**Q21: LoRA 的秩 r 如何选择？**

A: 选择依据：

| r 值  | 适用场景           | 特点                             |
| ----- | ------------------ | -------------------------------- |
| 2-8   | 简单任务、小数据集 | 极低资源，微调快，但表达能力有限 |
| 8-16  | 通用对话、文本分类 | 平衡效率和效果                   |
| 32-64 | 复杂推理、专业领域 | 效果好，但参数量增加             |

**经验法则**：

1. 从 r=8 或 r=16 开始
2. 复杂任务可增大到 32-64
3. 某些任务（如代码生成）可能需要更大的 r

**Q22: LoRA 和全量微调相比有什么优缺点？**

A:

| 方面                 | LoRA              | 全量微调           |
| -------------------- | ----------------- | ------------------ |
| **参数量**     | 极小（通常 < 1%） | 全部参数           |
| **显存**       | 低（只优化 A、B） | 高（优化所有梯度） |
| **训练速度**   | 快                | 慢                 |
| **效果**       | 接近全量微调      | 最佳               |
| **灾难性遗忘** | 较轻              | 严重               |
| **可扩展性**   | 多任务切换方便    | 每个任务需完整模型 |
| **部署**       | 可合并权重        | 需完整模型         |

---

### 4.2 高级变体

**Q23: 介绍一下 LoRA 的主要变体？**

A:

| 变体              | 核心改进               | 论文                 |
| ----------------- | ---------------------- | -------------------- |
| **LoRA**    | 基础低秩适配           | Hu et al. 2021       |
| **QLoRA**   | 量化 + LoRA，4-bit NF4 | Dettmers et al. 2023 |
| **AdaLoRA** | 自适应秩分配           | Zhang et al. 2023    |
| **LoRA+**   | A/B 使用不同学习率     | Hayou et al. 2024    |
| **DoRA**    | 权重分解 + 归一化      | Liu et al. 2024      |
| **LoRAfin** | LoRA + SFT 组合        | -                    |

**Q24: QLoRA 的原理是什么？**

A: QLoRA = Quantization + LoRA

**核心创新**：

1. **NF4 量化**：4-bit NormalFloat 量化，比普通 INT4 效果好

   - 数据相关量化，自适应非均匀量化
   - 避免异常值过度量化
2. **分页优化器**：处理梯度累积时的显存峰值

   - 将 optimizer states 卸载到 CPU
   - 需要时 paging 回 GPU
3. **双重量化**：对量化常数也做量化

   - 进一步降低显存

```python
# QLoRA 典型配置
quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_quant_type="nf4",  # NormalFloat4
    bnb_4bit_use_double_quant=True,  # 双重量化
)
```

**Q25: AdaLoRA 和普通 LoRA 有什么区别？**

A: AdaLoRA 通过**自适应秩分配**动态调整不同层的 LoRA 秩：

```python
# 普通 LoRA：所有层相同秩
for layer in model.layers:
    layer.lora_A = nn.Parameter(torch.randn(d, r))
    layer.lora_B = nn.Parameter(torch.randn(r, k))

# AdaLoRA：重要性引导的秩分配
for layer in model.layers:
    # 重要性 = SVD(ΔW) 的奇异值
    importance = compute_importance(layer.delta_W)
    r_layer = max(1, int(importance * r_max))
    # 动态调整 A、B 的秩
```

**优势**：将参数预算分配给更重要的层，提高效率。

**Q26: LoRA 在训练中可能遇到的问题及解决方案？**

A:

| 问题                   | 原因               | 解决方案                      |
| ---------------------- | ------------------ | ----------------------------- |
| **效果不佳**     | r 太小、学习率不当 | 增大 r，使用 LoRA+ 调整学习率 |
| **训练不稳定**   | 初始化问题         | A 用高斯初始化，B 用零初始化  |
| **灾难性遗忘**   | 微调过度           | 降低学习率，增加 r            |
| **推理延迟高**   | 实时计算 BA        | 离线合并权重到 W_0            |
| **多 LoRA 切换** | 需加载多个 adapter | 使用 PEFT 库的文件存储机制    |

---

## 五、编程与数学

### 5.1 矩阵运算编程

**Q27: 写一个函数实现 Multi-Head Self-Attention？**

A:

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, num_heads, dropout=0.1):
        super().__init__()
        assert d_model % num_heads == 0

        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # 线性投影
        self.W_Q = nn.Linear(d_model, d_model)
        self.W_K = nn.Linear(d_model, d_model)
        self.W_V = nn.Linear(d_model, d_model)
        self.fc = nn.Linear(d_model, d_model)

        self.dropout = nn.Dropout(dropout)
        self.scale = math.sqrt(self.d_k)

    def forward(self, Q, K, V, mask=None):
        batch_size = Q.size(0)

        # 线性投影 + 分头
        Q = self.W_Q(Q).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_K(K).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_V(V).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)

        # 计算注意力分数
        scores = torch.matmul(Q, K.transpose(-2, -1)) / self.scale

        # 应用 mask
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)

        # Softmax + Dropout
        attn_weights = self.dropout(F.softmax(scores, dim=-1))

        # 加权求和
        context = torch.matmul(attn_weights, V)

        # 合并多头
        context = context.transpose(1, 2).contiguous().view(batch_size, -1, self.d_model)

        # 最终线性投影
        output = self.fc(context)

        return output, attn_weights
```

**Q28: 如何用 Pytorch 实现 Flash Attention？**

A:

```python
import torch
import torch.nn.functional as F

def flash_attention(Q, K, V, scale=True, mask=None, dropout_p=0.0, training=False):
    """
    Q, K, V: (batch, heads, seq_len, d_k)
    """
    d_k = Q.size(-1)
    scale_factor = 1.0 / math.sqrt(d_k) if scale else 1.0

    # 方法1: 使用 PyTorch 内置的 scaled_dot_product_attention（Flash Attention 的高效实现）
    output = F.scaled_dot_product_attention(
        Q, K, V,
        attn_mask=mask,
        dropout_p=dropout_p,
        scale_factor=scale_factor,
        is_causal=False  # 如果是 decoder 的 masked attention，设为 True
    )

    return output

# 使用示例
Q = torch.randn(batch, heads, seq_len, d_k, device='cuda')
K = torch.randn(batch, heads, seq_len, d_k, device='cuda')
V = torch.randn(batch, heads, seq_len, d_k, device='cuda')

output = flash_attention(Q, K, V)  # 输出: (batch, heads, seq_len, d_k)
```

**Q29: 矩阵乘法如何避免数值溢出？**

A:

```python
import torch
import torch.nn.functional as F

# 问题场景：两个大矩阵相乘可能导致溢出
# A: (1000, 10000), B: (10000, 1000), 元素值在 0-100 之间
# A @ B 的结果可能达到 1000 * 10000 * 100 = 1e9，超出 float16 范围

# 解决方案1: 混合精度
with torch.cuda.amp.autocast():
    C = A @ B  # 在 bfloat16 或 float32 积累

# 解决方案2: 归一化输入
A = A / (A.abs().max() + 1e-8)  # 归一化到 [-1, 1]
B = B / (B.abs().max() + 1e-8)

# 解决方案3: 分块计算 + 缩放
def safe_matmul(A, B, block_size=512):
    result = torch.zeros(A.shape[0], B.shape[1], device=A.device)
    scale = 1.0 / block_size

    for i in range(0, A.shape[0], block_size):
        for j in range(0, B.shape[1], block_size):
            block = A[i:i+block] @ B[:, j:j+block]
            # 在累加前缩放，防止溢出
            result[i:i+block, j:j+block] += block * scale

    return result * block_size
```

---

## 六、深度学习基础

**Q30: 为什么 Transformer 用 LayerNorm 而不用 BatchNorm？**

A: 原因：

1. **NLP 任务中序列长度可变**：BN 依赖 batch 维度的统计量，变长序列场景效果差
2. **特征独立性**：LN 对每个样本独立归一化，保留样本间差异
3. **训练/推理一致性**：LN 在训练和推理时计算方式一致
4. **Transformer 结构**：FFN 是逐位置操作，天然适合 LN

```python
# BatchNorm: 沿着 batch 维度归一化
mean = x.mean(dim=0)  # 所有样本的同一特征
var = x.var(dim=0)

# LayerNorm: 沿着特征维度归一化
mean = x.mean(dim=-1)  # 单个样本的所有特征
var = x.var(dim=-1)
```

**Q31: AdamW 和 Adam 的区别是什么？**

A:

| 方面               | Adam                                                 | AdamW                                            |
| ------------------ | ---------------------------------------------------- | ------------------------------------------------ |
| **权重衰减** | L2 正则化混入梯度                                    | 独立的权重衰减项                                 |
| **公式**     | θ_{t+1} = θ_t - lr * (g_t / (√v_t + ε) + λθ_t) | θ_{t+1} = θ_t - lr * (m_t/(√v_t+ε) + λθ_t) |
| **效果**     | 梯度被衰减项污染                                     | 梯度纯净，收敛更好                               |

AdamW 是修复 Adam 中 L2 正则化实现 bug 的版本，现在几乎是默认选择。

**Q32: 梯度消失和梯度爆炸的原因及解决方案？**

A:

**原因**：

- 链式法则连乘导致梯度指数级变化
- 深层网络 + 激活函数（如 sigmoid）

**解决方案**：

| 方法                          | 原理                 |
| ----------------------------- | -------------------- |
| **残差连接**            | 梯度直接传递到浅层   |
| **BatchNorm/LayerNorm** | 稳定每层输入分布     |
| **合适的激活函数**      | ReLU（梯度恒为1或0） |
| **梯度裁剪**            | 防止梯度爆炸         |
| **预训练 + 微调**       | 降低从零训练的风险   |

**Q33: 解释 Cross-Entropy Loss 和它的梯度？**

A:

```python
def cross_entropy_loss(logits, targets):
    """
    logits: (batch, num_classes) - 未归一化的分数
    targets: (batch,) - 类别索引或 (batch, num_classes) one-hot
    """
    log_probs = F.log_softmax(logits, dim=-1)
    loss = F.nll_loss(log_probs, targets)  # 负对数似然
    return loss

# 梯度推导：dL/dlogits = softmax(logits) - targets
# 即：预测概率 - 真实标签
```

---

## 七、模型优化与部署

**Q34: 如何量化一个 Transformer 模型？**

A: 主要量化方法：

| 方法           | 精度          | 效果                 | 速度 |
| -------------- | ------------- | -------------------- | ---- |
| **FP16** | 16位浮点      | 接近 FP32            | 2x   |
| **BF16** | 16位脑浮点    | 接近 FP32            | 2x   |
| **INT8** | 8位整数       | 略有下降             | 3-4x |
| **INT4** | 4位整数       | 明显下降             | 6-8x |
| **NF4**  | 4位标准化浮点 | 效果更好（数据相关） | 6-8x |

**PTQ（训练后量化）**：

```python
# 使用 BitsAndBytes
quantization_config = BitsAndBytesConfig(
    load_in_8bit=True,
    llm_int8_threshold=6.0,  # INT8 混合精度阈值
)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=quantization_config,
)
```

**Q35: 如何减少 Transformer 的推理延迟？**

A:

1. **KV Cache**：缓存已计算的 K、V，避免重复计算

   ```python
   # 传统：每个 token 都重新计算
   for new_token in new_tokens:
       output = attention(embeddings)  # 包含对所有历史 token 的 attention

   # KV Cache：只计算新 token 的 attention
   k_cache.append(compute_K(new_token))
   v_cache.append(compute_V(new_token))
   output = attention(q_new, k_cache, v_cache)
   ```
2. **批量推理**：多个请求合并处理
3. **投机解码**：用小模型生成 draft，大模型验证
4. **算子融合**：减少内存访问
5. **剪枝**：移除不重要的注意力头或层

---

## 八、综合问题

**Q36: 你如何理解大模型的"涌现能力"(Emergent Abilities)？**

A: 涌现能力是指当模型规模超过某个阈值后，突然出现的新能力：

1. **定义**：模型规模较小时不存在，规模增大后突然出现的能力
2. **例子**：
   - 思维链（Chain-of-Thought）：需要足够大的模型才能激活
   - 复杂推理、多步计算、上下文学习
3. **可能原因**：
   - 更大模型能学习更复杂的函数组合
   - 涌现可能只是评估指标的突跃，不是真正的"跳跃"
   - 大量无标注数据中的隐式模式学习

**Q37: 你了解哪些大模型架构？它们有什么区别？**

A:

| 架构                      | 代表模型         | 特点                       |
| ------------------------- | ---------------- | -------------------------- |
| **Encoder-only**    | BERT, RoBERTa    | 双向理解，适用于理解任务   |
| **Decoder-only**    | GPT, LLaMA, PaLM | 自回归生成，适合生成任务   |
| **Encoder-Decoder** | T5, FLAN-T5      | 适合序列到序列任务         |
| **MoE**             | Mixtral, GShard  | 稀疏激活，参数多但计算量小 |

**LLaMA vs GPT**：

- LLaMA：开源，Decoder-only，Llama2 使用 GQA
- GPT-4：闭源，多模态，RLHF 训练

**Q38: 为什么Decoder-only 架构成为主流？**

A:

1. **扩展性好**：生成任务是终极任务，理解和推理都可以通过生成表达
2. **上下文学习**：Decoder-only 的因果注意力天然适合 in-context learning
3. **工程简单**：统一的生成接口，易于扩展
4. **RLHF 友好**：直接优化生成质量，reward 信号明确
5. **涌现能力**：大规模预训练 + instruction tuning 后涌现强大能力

---

## 九、反向提问

面试最后通常会问你有什么问题。以下是一些合适的问题：

**Q39: 面试官可能会问的"你有什么问题想问我"**

A:

1. **技术相关**：

   - "华为在大模型领域的布局和重点方向是什么？"
   - "团队使用什么样的技术栈和开发流程？"
   - "对于新入职的员工，有什么培训机制？"
2. **团队相关**：

   - "团队的规模和组成是怎样的？"
   - "我入职后会负责什么样的项目？"
   - "团队的技术分享文化是怎样的？"
3. **发展相关**：

   - "华为对员工的技术成长有什么支持？"
   - "这个岗位的晋升路径是怎样的？"

---

## 十、面试技巧总结

### 回答原则

1. **结构化表达**：使用"首先、其次、最后"或"总-分-总"结构
2. **结合项目**：技术问题最好结合实际项目经验
3. **深入原理**：不仅要知其然，还要知其所以然
4. **诚实谦虚**：不会的问题坦诚承认，表达学习意愿
5. **举一反三**：展示你对技术的整体理解

### 华为面试特点

1. **技术深度**：喜欢追问原理和细节
2. **工程能力**：重视代码能力和实际问题的解决
3. **国产化**：可能涉及昇腾芯片、MindSpore 等华为生态
4. **狼性文化**：强调奋斗精神和团队协作

---

> 祝面试顺利！如果需要更详细的内容或特定方向的深入讲解，请随时告知。
