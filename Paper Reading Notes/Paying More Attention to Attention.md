# **Paying More Attention to Attention: Improving the Performance of Convolutional Neural Networks via Attention Transfer**

<center>2020年07月11日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp ICLR &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 特征 &nbsp</span> &nbsp &nbsp
</center> 



## **摘要**

> 通过卷积神经网络的注意力，可以使用这类信息，通过使学生CNN网络模仿一个强大的教师网络的注意力地图，从而显著提高其性能。为此提出了几种转移注意力的新方法，在各种数据集和卷积神经网络架构上显示出显著的改进。

## **论文内容**

### 问题

一个教师网络是否可以通过提供其注意力信息来改善学生网络的性能。

**贡献：**

1. 我们提出将注意力作为一种将知识从一个网络转移到另一个网络的机制。
2. 提出基于激活和基于梯度的空间注意图。
3. 通过实验表明，在各种数据集和网络体系结构上均提供了改进，并且可以与其他知识蒸馏方法结合使用

### 激活注意力

注意力映射：$F: \mathbb{R}^{C \times H \times W} \to \mathbb{R}^{H \times W}$

用 $A$ 表示某个Res Block的激活：

- **绝对值之和：**
  $$
  F_{sum}(A) = \sum_{i=1}^C |A_i|
  $$
-  **$p$ 次幂绝对值之和：**
  $$
  F_{sum}^p(A) = \sum_{i=1}^C |A_i|^p
  $$
- **$p$ 次幂最大绝对值：**
  $$
  F_{max}^p(A) = \max_{i=1,C} |A_i|^p
  $$

**区别：**

- 在不同空间位置，$F_{sum}^p(A)$ 更偏重于最高激活神经元对应的空间位置。

- 在同一空间位置，$F_{max}^p(A)$ 只考虑一个最高激活的神经元，而 $F_{sum}^p(A)$ 会考虑多个的综合。

不同层提取得到的注意力有所区别：

<img src="figure\image-20200711143223207.png" alt="image-20200711143223207" style="zoom:50%;" />

注意力迁移方式：

- **深度相同：**逐层对齐

- **深度不同：**各个Res Block对齐

  <img src="figure\image-20200711143427765.png" alt="image-20200711143427765" style="zoom:50%;" />

**损失函数：**
$$
\mathcal{L}_{A T}=\mathcal{L}\left(\mathbf{W}_{S}, x\right)+\frac{\beta}{2} \sum_{j \in \mathcal{I}}\left\|\frac{Q_{S}^{j}}{\left\|Q_{S}^{j}\right\|_{2}}-\frac{Q_{T}^{j}}{\left\|Q_{T}^{j}\right\|_{2}}\right\|_{p}
$$

### 梯度注意力

对齐教师模型和学生模型的梯度：
$$
\mathcal{L}_{A T}\left(\mathbf{W}_{\mathbf{S}}, \mathbf{W}_{\mathbf{T}}, x\right)=\mathcal{L}\left(\mathbf{W}_{\mathbf{S}}, x\right)+\frac{\beta}{2}\left\|J_{S}-J_{T}\right\|_{2}
$$

$$
J_{S}=\frac{\partial}{\partial x} \mathcal{L}\left(\mathbf{W}_{\mathbf{S}}, x\right), J_{T}=\frac{\partial}{\partial x} \mathcal{L}\left(\mathbf{W}_{\mathbf{T}}, x\right)
$$

另一种方法，**翻转对齐**：
$$
\mathcal{L}_{s y m}(\mathbf{W}, x)=\mathcal{L}(\mathbf{W}, x)+\frac{\beta}{2}\left\|\frac{\partial}{\partial x} \mathcal{L}(\mathbf{W}, x)-\operatorname{flip}\left(\frac{\partial}{\partial x} \mathcal{L}(\mathbf{W}, \operatorname{flip}(x))\right)\right\|_{2}
$$

### 结果

在大小数据集上都取得不错成果，以Cifar为例：

<img src="figure\image-20200711143843774.png" alt="image-20200711143843774" style="zoom:67%;" />

<img src="figure\image-20200711143900983.png" alt="image-20200711143900983" style="zoom:67%;" />

实验设置参考原文


## **结论**

1. 提出了几种可以用于知识蒸馏，将注意力从一个网络转移到另一个网络的方法，并在几个图像识别数据集上进行了实验。


**Ref:**  [Zagoruyko S, Komodakis N. Paying more attention to attention: Improving the performance of convolutional neural networks via attention transfer[J]. arXiv preprint arXiv:1612.03928, 2016.](https://arxiv.org/pdf/1612.03928.pdf)