---
title: "机器学习中常用的一些公式推导"
date: 2023-12-18T21:54:24+08:00
categories:
    - machine learning
tags:
    - machine learning
---

## 线性层的反向传播

对于函数 $ Y = XW $ （ 注：$ X $ 是一个 $ m \times n $ 的矩阵，$ W $ 是一个 $ n \times k $ 的矩阵，$ Y $ 是一个 $ m \times k $ 的矩阵。这里的 $ W $ 通常代表模型的权重，而 $ X $ 代表输入数据。）

如何求 $ \frac{\partial Y}{\partial W} $呢，通常我们只关心其一个特定的切片，即 $ \frac{\partial Y_{ij}}{\partial W_{rs}} $。

对于单个元素 $ Y_{ij} $ 而言，其是矩阵 $ X $ 的第 $ i $ 行和矩阵 $ W $ 的第 $ j $ 列的点积。因此：

$$ Y_{ij} = \sum_{p=1}^{n} X_{ip} \cdot W_{pj} $$

当我们对 $ Y_{ij} $ 关于 $ W_{rs} $ 求导时，我们只关心 $ W_{rs} $ 这一项，其他项的导数为0。因此：

$$ \frac{\partial Y_{ij}}{\partial W_{rs}} = \frac{\partial}{\partial W_{rs}} \left( \sum_{p=1}^{n} X_{ip} \cdot W_{pj} \right) $$

$$ \frac{\partial Y_{ij}}{\partial W_{rs}} = X_{ir} \cdot \frac{\partial W_{sj}}{\partial W_{rs}} $$

由于 $ \frac{\partial W_{sj}}{\partial W_{rs}} $ 只有当 $ s = r $ 且 $ j = s $ 时才为1，否则为0，因此：

$$ \frac{\partial Y_{ij}}{\partial W_{rs}} = X_{ir} \cdot \delta_{sj} $$

其中 $ \delta_{sj} $ 是Kronecker delta 函数，当 $ s = j $ 时为1，否则为0

因此，$ \frac{\partial Y}{\partial W} $ 的每个元素 $ \frac{\partial Y_{ij}}{\partial W_{rs}} $ 都是一个 $ m \times n \times k $ 的张量，其中 $ i $ 和 $ j $ 分别索引 $ Y $ 的行和列，而 $ r $ 和 $ s $ 分别索引 $ W $ 的行和列。

然而，通常我们不会将 $ \frac{\partial Y}{\partial W} $ 视为一个张量，而是将其简化为一个矩阵。因为在实际应用中，我们通常对整个矩阵 $ W $ 进行更新，而不是单独更新它的每个元素。因此，我们可以将 $ \frac{\partial Y}{\partial W} $ 的所有元素 $ \frac{\partial Y_{ij}}{\partial W_{rs}} $ 合并成一个 $ m \times k \times n $ 的张量，然后将其简化为一个 $ m \times n $ 的矩阵，即 $ X $。这是因为在矩阵乘法中，$ Y $ 的每个元素 $ Y_{ij} $ 只依赖于 $ X $ 的第 $ i $ 行和 $ W $ 的第 $ j $ 列，所以 $ \frac{\partial Y}{\partial W} $ 实际上是一个 $ m \times n $ 的矩阵，而不是一个 $ m \times n \times k $ 的张量，最终有：

$$ \frac{\partial Y}{\partial W} = X $$

可通过此网站 [https://www.matrixcalculus.org/](https://www.matrixcalculus.org/) 验证矩阵求导的正确性。

在神经网络的链式求导过程，我们求 $ Y $ 对 $ W $ 的梯度，通常还会乘以 $ Y $ 本身对应的梯度 $grad_y$，$grad_y$ 是 $ m \times k $ 的矩阵，这时候我们需要对 $ \frac{\partial Y}{\partial W} $ 转置。



## Softmax函数的反向传播
Softmax函数的定义是：

$$ p_i = \frac{e^{z_i}}{\sum_j e^{z_j}} $$

对于Softmax函数的导数 $ \frac{\partial p_i}{\partial z_j} $， 可以通过以下方式计算：

$$ \frac{\partial p_i}{\partial z_j} = \frac{\partial}{\partial z_j} \left( \frac{e^{z_i}}{\sum_k e^{z_k}} \right) $$

使用商的求导法则，我们得到：

$$ \frac{\partial p_i}{\partial z_j} = \frac{e^{z_i} \cdot \frac{\partial}{\partial z_j}(\sum_k e^{z_k}) - \frac{\partial}{\partial z_j}(e^{z_i}) \cdot \sum_k e^{z_k}}{(\sum_k e^{z_k})^2} $$

简化后，我们得到：

$$ \frac{\partial p_i}{\partial z_j} = \frac{e^{z_i} \cdot \delta_{ij} - e^{z_i} \cdot e^{z_j}}{(\sum_k e^{z_k})^2} $$

其中 $ \delta_{ij} $ 是Kronecker delta函数，当 $ i = j $ 时为1，否则为0。

进一步简化，我们得到：

$$ \frac{\partial p_i}{\partial z_j} = p_i \cdot (1 - \delta_{ij}) - p_i \cdot p_j $$

最终，我们得到Softmax函数的导数为：

$$ \frac{\partial p_i}{\partial z_j} = \begin{cases} p_i (1 - p_j)   \text{  , if } i = j;
\newline -p_i p_j  \text{ , if } i \neq j \end{cases} $$


Softmax函数的导数是一个矩阵，其中每个元素都是输出概率的函数。

## Relu函数的反向传播

ReLU（Rectified Linear Unit）函数是深度学习中常用的一种激活函数，其数学表达式为：

$$ f(x) = \max(0, x) $$

其反向传播公式为：

$$  \frac{\partial f(x)}{\partial x} = \begin{cases} 0   \text{  , if } x \leq 0;
\newline 1  \text{ , if } x > 0 \end{cases}$$

## MSE函数的反向传播

MSE（Mean Squared Error）函数是深度学习中常用的一种损失函数，其公式如下：

$$ MSE = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2 $$

其中，$ n $ 是数据点的数量，$ y_i $ 是第 $ i $ 个数据点的真实值，$ \hat{y}_i $ 是第 $ i $ 个数据点的预测值。

在反向传播过程中，MSE 损失函数单个预测值 $ \hat{y}_i $ 的梯度可以表示为：

$$ \frac{\partial MSE}{\partial \hat{y}_i} =  -\frac{2}{n} (y_i - \hat{y}_i) $$
