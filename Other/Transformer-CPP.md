这是经典 Transformer 架构示意图，主要包括 Encoder 和 Decoder 两部分，其中涉及多个核心模块（Multi-Head Attention、Add & Norm、Feed Forward 等）。下面是用 C++ 编写的一个简化实现，涵盖 Transformer 的基本结构和功能模块。

我会尽量详细地添加中文注释来帮助你理解每个模块的实现。

---

### **C++ Transformer 主要代码**

```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <random>
#include <algorithm>

using namespace std;

// 定义矩阵类型
using Matrix = vector<vector<float>>;

// 随机数生成器用于初始化权重
float randomFloat() {
    static random_device rd;
    static mt19937 gen(rd());
    static uniform_real_distribution<> dis(-0.1, 0.1);
    return dis(gen);
}

// ---- 核心模块1：多头注意力机制 ---- //
class MultiHeadAttention {
private:
    int num_heads;
    int head_dim;
    int model_dim;

    Matrix Wq, Wk, Wv, Wo;

public:
    MultiHeadAttention(int model_dim, int num_heads) : model_dim(model_dim), num_heads(num_heads) {
        head_dim = model_dim / num_heads;
        
        // 初始化权重矩阵 Wq、Wk、Wv 和 Wo
        Wq = initializeWeight(model_dim, model_dim);
        Wk = initializeWeight(model_dim, model_dim);
        Wv = initializeWeight(model_dim, model_dim);
        Wo = initializeWeight(model_dim, model_dim);
    }

    Matrix initializeWeight(int rows, int cols) {
        Matrix weight(rows, vector<float>(cols));
        for (auto& row : weight)
            for (auto& w : row) 
                w = randomFloat();
        return weight;
    }

    // 简化的注意力计算
    Matrix scaledDotProductAttention(const Matrix& Q, const Matrix& K, const Matrix& V) {
        int seq_len = Q.size();
        Matrix attention_weights(seq_len, vector<float>(seq_len));

        // 计算 QK^T / sqrt(d_k)
        float scale = sqrt(K[0].size());
        for (int i = 0; i < seq_len; ++i) {
            for (int j = 0; j < seq_len; ++j) {
                float dot_product = inner_product(Q[i].begin(), Q[i].end(), K[j].begin(), 0.0);
                attention_weights[i][j] = dot_product / scale;
            }
        }

        // 选择最大值归一化 Softmax
        for (auto& row : attention_weights) {
            float sum_exp = 0.0;
            for (auto& v : row) v = exp(v), sum_exp += v;
            for (auto& v : row) v /= sum_exp;
        }

        // 计算加权结果
        Matrix output(seq_len, vector<float>(V[0].size(), 0.0));
        for (int i = 0; i < seq_len; ++i) {
            for (int j = 0; j < V[0].size(); ++j) {
                for (int k = 0; k < seq_len; ++k) {
                    output[i][j] += attention_weights[i][k] * V[k][j];
                }
            }
        }
        return output;
    }

    // 执行多头注意力
    Matrix forward(const Matrix& input) {
        // 生成查询、键和值
        Matrix Q = multiply(input, Wq);
        Matrix K = multiply(input, Wk);
        Matrix V = multiply(input, Wv);
        return scaledDotProductAttention(Q, K, V);
    }

    Matrix multiply(const Matrix& A, const Matrix& B) {
        int rows = A.size(), cols = B[0].size(), common = B.size();
        Matrix result(rows, vector<float>(cols, 0.0));
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                for (int k = 0; k < common; ++k) {
                    result[i][j] += A[i][k] * B[k][j];
                }
            }
        }
        return result;
    }
};

// ---- 核心模块2：前馈网络 ---- //
class FeedForwardNetwork {
private:
    int model_dim;
    int hidden_dim;
    Matrix W1, W2;

public:
    FeedForwardNetwork(int model_dim, int hidden_dim) : model_dim(model_dim), hidden_dim(hidden_dim) {
        W1 = initializeWeight(model_dim, hidden_dim);
        W2 = initializeWeight(hidden_dim, model_dim);
    }

    Matrix initializeWeight(int rows, int cols) {
        Matrix weight(rows, vector<float>(cols));
        for (auto& row : weight)
            for (auto& w : row)
                w = randomFloat();
        return weight;
    }

    Matrix relu(const Matrix& input) {
        Matrix output = input;
        for (auto& row : output) {
            for (auto& val : row) {
                val = max(0.0f, val);
            }
        }
        return output;
    }

    Matrix forward(const Matrix& input) {
        Matrix hidden = multiply(input, W1);
        hidden = relu(hidden);
        return multiply(hidden, W2);
    }

    Matrix multiply(const Matrix& A, const Matrix& B) {
        int rows = A.size(), cols = B[0].size(), common = B.size();
        Matrix result(rows, vector<float>(cols, 0.0));
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                for (int k = 0; k < common; ++k) {
                    result[i][j] += A[i][k] * B[k][j];
                }
            }
        }
        return result;
    }
};

// ---- 核心模块3：Layer Normalization ---- //
class LayerNormalization {
public:
    void normalize(Matrix& input) {
        for (auto& row : input) {
            float mean = accumulate(row.begin(), row.end(), 0.0) / row.size();
            float variance = 0.0;
            for (const auto& val : row) {
                variance += (val - mean) * (val - mean);
            }
            variance /= row.size();
            float stddev = sqrt(variance + 1e-6);

            for (auto& val : row) {
                val = (val - mean) / stddev;
            }
        }
    }
};

int main() {
    const int model_dim = 512;
    const int num_heads = 8;
    const int hidden_dim = 2048;

    // 输入假数据
    Matrix input = {{1.0, 2.0, 3.0}, {4.0, 5.0, 6.0}};

    MultiHeadAttention attention(model_dim, num_heads);
    FeedForwardNetwork ffn(model_dim, hidden_dim);
    LayerNormalization layerNorm;

    // 测试 Attention 模块
    Matrix attention_output = attention.forward(input);
    cout << "Multi-Head Attention 结果:" << endl;
    for (const auto& row : attention_output) {
        for (const auto& val : row) {
            cout << val << " ";
        }
        cout << endl;
    }

    // 测试前馈网络
    Matrix ffn_output = ffn.forward(input);
    cout << "前馈网络结果:" << endl;
    for (const auto& row : ffn_output) {
        for (const auto& val : row) {
            cout << val << " ";
        }
        cout << endl;
    }

    // 测试 LayerNorm
    layerNorm.normalize(input);
    cout << "Layer Normalization 结果:" << endl;
    for (const auto& row : input) {
        for (const auto& val : row) {
            cout << val << " ";
        }
        cout << endl;
    }

    return 0;
}
```

---

### **代码说明**
1. **多头注意力机制 (MultiHeadAttention)**：
   - 实现了 Q、K、V 的计算和缩放点积注意力，使用 softmax 做权重归一化。

2. **前馈网络 (FeedForwardNetwork)**：
   - 两层全连接网络，带有 ReLU 激活函数。

3. **层归一化 (LayerNormalization)**：
   - 对输入矩阵进行标准化处理。

4. **中文备注与结构**：
   - 每个核心模块都有清晰的中文备注，方便理解算法原理与实现细节。

---

如果需要进一步优化或补充其他功能，请告诉我！
