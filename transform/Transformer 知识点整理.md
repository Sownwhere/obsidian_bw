

## 1. Embedding Matrix

**定义**：

- Embedding matrix 是一个可训练矩阵，用于将离散 token 映射为连续向量。
    
- 假设词表大小 (V)，模型维度 (d_{model})，则 embedding matrix 大小为 (E \in \mathbb{R}^{V \times d_{model}})。
    

**公式**：  
[  
x_i = E[token_i]  
]

- (x_i \in \mathbb{R}^{d_{model}}) 是 token 的向量表示。
    
- 输入 token 的索引 (token_i \in {0, 1, ..., V-1})。
    

**作用**：

1. 将输入序列 token 转换为连续向量，作为 Transformer 的输入。
    
2. 可以与输出层权重共享，将 hidden state 映射回词表概率（weight tying）：  
    [  
    \hat{y}_i = \text{softmax}(x_i \cdot E^T)  
    ]
    

**应用流程**：

- Token 序列 → Embedding Matrix → Embedding 向量序列 → 加上 Positional Encoding → Transformer 自注意力层
    

## 2. Softmax

**定义**：

- Softmax 将任意实数向量转换为概率分布。
    
- 对向量 (\mathbf{z} = [z_1, ..., z_K])，softmax 定义为：  
    [  
    \text{softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}, \quad i=1,...,K  
    ]
    
- 输出值在 (0,1)，总和为 1。
    

**作用**：

1. 将 logits 归一化为概率分布。
    
2. 放大高分值，使模型预测更自信。
    
3. 可微分，用于神经网络反向传播。
    

**示例**：

- Logits: [2.0, 1.0, 0.1]
    
- 指数化: [7.39, 2.72, 1.11]
    
- Softmax 输出: [0.659, 0.242, 0.099]
    

**Transformer 中的应用**：

1. **输出层**：
    
    - 将隐藏向量映射为每个 token 的概率：  
        [  
        P(\text{token}_i) = \text{softmax}(x \cdot E^T)  
        ]
        
2. **注意力机制**：
    
    - 将 Query 和 Key 的相似度归一化为权重：  
        [  
        \text{AttentionWeights} = \text{softmax}(Q K^T / \sqrt{d_k})  
        ]
        

## 3. Unembedding

**定义**：

- Unembedding 指的是将 Transformer 的 hidden state 映射回词表空间（logits）的过程。
    
- 通常通过与 embedding matrix 权重共享的线性变换实现（weight tying）：  
    [  
    \text{logits} = H \cdot E^T  
    ]
    
- (H) 是 Transformer 的隐藏状态矩阵。
    

**作用**：

1. 将隐藏向量转换为每个 token 的原始分数（logits）。
    
2. 结合 Softmax，可以得到每个 token 的预测概率。
    
3. 通过共享 embedding 权重，提高参数效率并增强模型泛化能力。