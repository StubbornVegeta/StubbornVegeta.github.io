---
title: "tensorflow新建op (cpp) "
date: 2022-08-06T22:12:00+08:00
image: "/categories/tensorflow/tensorflow.png"
categories:
    - tensorflow
tags:
    - tensorflow1.x
    - op
---

## 为什么使用cpp新建op

1. 一些操作表示成现有操作的组合不好实现或者无法实现。
2. 已有操作的组合效率不高。
3. 想要自定义一些基本操作的组合，因为未来编译器做这种融合可能会比较困难。

## 如何使用cpp新建op

1. 注册op，注册op会定义一个接口（规范），比如定义op的名称和它的输入输出、shape函数（用于获取张量的形状）
2. 实现op，对于CPU和GPU可以有不同的实现
3. 为op编写一个函数来计算梯度（可选）

其实通过前两个步骤，我们就可以编写出一个可用的op，只是神经网络的反向传播需要计算梯度，因此涉及到反向传播求梯度操作时，我们还需要为op编写梯度计算函数，如果自定义的op不涉及求梯度，则无需编写梯度计算函数。

## 新建矩阵乘法op

### 注册矩阵乘法op
您可以在 [**这里**](https://github.com/StubbornVegeta/tensorflow-custom-op) 看到完整的代码
```cpp
#include "tensorflow/core/framework/op_kernel.h"
#include "tensorflow/core/framework/op.h"
#include "tensorflow/core/framework/shape_inference.h"

using namespace tensorflow;

REGISTER_OP("Mymatmul")
  .Attr("T: {float, int32, int64, double}")
  .Input("matrix1: T")
  .Input("matrix2: T")
  .Output("matmuled: T")
  .SetShapeFn([](::tensorflow::shape_inference::InferenceContext* c){
                auto N = c->Dim(c->input(0), 0);
                auto M = c->Dim(c->input(1), 1);
                c->set_output(0, c->MakeShape({N,M}));
                return Status::OK();
              });
```
* 这里通过`.Attr`为输入输出添加多种类型，从而达到多态的目的
* 我们可以从`InferenceContext*`类型的上下文参数中获取输入以及它们的形状
* `SetShapeFn`用来确定输出的形状
    * `c->input(0)`： 获取第一个输入参数
    * `c->Dim(X, 0)`：获取`X`的第一个维度
    * `c->set_output(0, ...)`：设置第一个输出的形状


### 实现op

实现op的大体框架如下：
```cpp
template<typename T>
class MymatmulOp : public OpKernel {
public:
  explicit MymatmulOp(OpKernelConstruction* context) : OpKernel(context) {}
  void Compute(OpKernelContext* context) override {
    // ...
  }
```
我们要创建一个继承自`OpKernel`的类，并重载`Compute`方法

`Compute`方法有一个类型为`OpKernelContext*`的参数`context`，从中可以访问输入输出张量等有用的信息

接下来我们要在这个框架中完成具体的op实现

#### 创建输入输出张量

我们可以从`context`中直接读取输入张量以及它们的形状，根据它们的形状来计算输出张量的形状，进而为输出张量分配内存:

$$ Output_{N\times M} = Input1_{N\times K} \times Input2_{K\times M}  $$

$$ \downdownarrows $$

$$[N, K] \times [K, M] \to [N, M]$$

具体实现如下：
```cpp
    // create input tensor
    const Tensor& input_tensor1 = context->input(0);
    const Tensor& input_tensor2 = context->input(1);
    const TensorShape& input1_shape = input_tensor1.shape();
    const TensorShape& input2_shape = input_tensor2.shape();

    // create output tensor
    TensorShape output_shape;
    const int N = input1_shape.dim_size(0);
    const int M = input2_shape.dim_size(1);
    output_shape.AddDim(N);
    output_shape.AddDim(M);
    Tensor* output_tensor = NULL;
    OP_REQUIRES_OK(context, context->allocate_output(0, output_shape, &output_tensor));
```

#### 实现矩阵乘法
根据下面公式实现矩阵乘法

$$ { output_{ij} = input1_{ik} \times input2_{kj} }$$
```cpp
    auto input1 = input_tensor1.matrix<T>();
    auto input2 = input_tensor2.matrix<T>();
    auto output = output_tensor->template matrix<T>();

    // matmul
    for(int i = 0; i < N; i++) {
      for(int j = 0; j < M; j++) {
        output(i,j) = 0;
        for(int k = 0; k < input1_shape.dim_size(1); k++) {
          output(i,j) += input1(i, k) * input2(k, j);
        }
      }
    }
```

#### 添加约束条件
添加约束条件主要考虑到两点：
1. 自定义的op可能有多种实现，比如针对CPU和GPU有不同的实现
2. 定义了多态，需要向TensorFlow系统指明本次注册的op实现是针对哪一种类型的

```cpp
#define REGISTER_KERNEL(type)                                     \
  REGISTER_KERNEL_BUILDER(                                        \
    Name("Mymatmul").Device(DEVICE_CPU).TypeConstraint<type>("T"),\
    MymatmulOp<type>)

  REGISTER_KERNEL(int32)
  REGISTER_KERNEL(int64)
  REGISTER_KERNEL(float)
  REGISTER_KERNEL(double)
```
这里定义了`REGISTER_KERNEL`宏，方便我们注册多种类型的op

### 构建库文件

本文提供了g++的构建方式，可使用下面的命令构建op的库文件
```bash
TF_CFLAGS=( $(python -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_compile_flags()))') )
TF_LFLAGS=( $(python -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_link_flags()))') )

g++ -std=c++11 -shared op_mymatmul.cc -o op_mymatmul.so -fPIC ${TF_CFLAGS[@]} ${TF_LFLAGS[@]} -O2
```
`TF_CFLAGS`和`TF_LFLAGS`分别为构建op所需要的头文件路径和库文件路径


### 验证op可行性
```python
import tensorflow as tf
m = tf.load_op_library('./op_mymatmul.so')

a = tf.constant(
        [[1., 2],
        [3, 4],
        [1, 1]])

b = tf.constant(
        [[1., 2, 1],
        [3, 4, 1]])

with tf.Session('') as s:
    print(s.run(m.mymatmul(a, b)))
    print(s.run(tf.matmul(a, b)))
```
`mymatmul`和TensorFlow自带的`matmul`输出结果一致。


## 为op添加梯度计算

如果将构建好的op应用到tensorflow搭建好的神经网络中，比如`mnist手写数字识别`，我们将得到梯度未定义的错误

> LookupError: No gradient define for operation 'Mymatmul' (op type Mymatmul)

因此我们还需要为op添加梯度计算


### 注册op
与前面流程类似，我们仍然需要先注册梯度op
```cpp
REGISTER_OP("MymatmulGrad")
  .Attr("T: {float, int32, int64, double}")
  .Input("grad: T")
  .Input("input1: T")
  .Input("input2: T")
  .Output("grad_input1: T")
  .Output("grad_input2: T");
```
这里会接受三个输入，`grad`为矩阵乘法op输出的梯度，`input1`和`input2`为参与矩阵乘法的两个矩阵，输出为两个矩阵的梯度

### 实现op

已知输出梯度，求输入梯度，经典的反向传播求梯度

假设输出误差为$L$，输出为$y$， 两个输入矩阵分别$W$和$x$

$$  y = Wx    $$

$$  \dfrac{\partial L}{\partial x} = \dfrac{\partial y}{\partial x} \dfrac{\partial L}{\partial y}  = W\dfrac{\partial L}{\partial y} $$

根据公式编写代码如下
```cpp
template<typename T>
class MymatmulGradOp : public OpKernel {
public:
  explicit MymatmulGradOp(OpKernelConstruction* context) : OpKernel(context) {}
  void Compute(OpKernelContext* context) override {
    // create input tensor ...

    // create output tensor ...

    // init
    for(int j = 0; j < K; j++) {
      for(int i = 0; i < N; i++) {
        grad_input1(i, j) = 0.0;
      }
    }

    for(int j = 0; j < M; j++) {
      for(int i = 0; i < K; i++) {
        grad_input2(i, j) = 0.0;
      }
    }

    // matmul
    for(int i = 0; i < N; i++) {
      for(int j = 0; j < M; j++) {
        for(int k = 0; k < K; k++) {
          grad_input1(i, k) += input2(k, j) * grad(i, j);
          grad_input2(k, j) += input1(i, k) * grad(i, j);
        }
      }
    }
  }
};

```
给op添加约束条件以及构建库文件同前面一样

### 注册梯度计算
```python
# FILE: op_mymatmul_grad.py
import tensorflow as tf
from tensorflow.python.framework import ops
m = tf.load_op_library('./op_mymatmul_grad.so')

@ops.RegisterGradient("Mymatmul")
def mymatmul_grad_cc(op, grad):
    return m.mymatmul_grad(grad, op.inputs[0], op.inputs[1])
```
这里需要注意，`@ops.RegisterGradient(...)`里面传的是自定义op的名字，而不是梯度op的名字，因为我们要将梯度和自定义op绑定在一起

### 验证可用性
```python
# ...

import op_mymatmul_grad

# ...

# In addition to replacing matrix multiplication with mymatmul, just write your neural network model normally

m = tf.load_op_library('./op_mymatmul.so')

m.mymatmul(...)

```
最终结果正常。

## 项目地址
[StubbornVegeta/tensorflow-custom-op](https://github.com/StubbornVegeta/tensorflow-custom-op)
