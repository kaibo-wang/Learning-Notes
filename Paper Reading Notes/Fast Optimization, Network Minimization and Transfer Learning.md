# **A Gift from Knowledge Distillation: Fast Optimization, Network Minimization and Transfer Learning**

<center>2020年07月14日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp CVPR &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 关系提取 &nbsp</span> 
</center> 



## **摘要**

> 根据层之间的流动可以定义要传输的提炼知识，通过计算两层特征之间的内积来计算的。两层之间的流动所传递的知识表现出三个现象：（1）学生DNN学习提炼知识的过程比原始模型快得多；（2）蒸馏后学生DNN优于原始DNN；（3）学生DNN可以从受过不同任务训练的老师DNN中学习提炼的知识，而学生DNN的表现要优于从头开始训练的原始DNN。

## **论文内容**

### 问题

传统知识蒸馏方法具有局限性，例如难以优化非常深的网络，而且知识转移的表现对如何定义提炼的知识非常敏感。

考虑到老师除了传授知识，还会教给学生解决问题的流程，因此知识也可以定义为解决问题的流程，如网络两层特征之间的关系。从两层提取的特征可以用于生成求解过程（FSP）矩阵流，训练学生DNN使其FSP矩阵类似于教师网络DNN的FSP矩阵。

<img src="figure\image-20200714120528965.png" alt="image-20200714120528965" style="zoom:67%;" />

### 方法

对于第 $i$ 层特征 $F_i(x) \in \mathbb{R}^{m \times H \times W}$ 和第 $j$ 层特征 $F_j(x) \in \mathbb{R}^{n \times H \times W}$，计算两层间的FSP矩阵 $G_{i,j} \in \mathbb{R}^{m \times n}$，矩阵中元素 $G_{ab}$ 等于对应两通道特征图的内积：
$$
G_{ab} =  \frac{1}{HW} F_i^{a}(x) \cdot F_j^{b}(x)
$$
损失函数只考虑教师模型与学生模型大小一致的FSP对，计算二者之差的二范数：
$$
L_{F S P}\left(W_{t}, W_{s}\right) 
=\frac{1}{N} \sum_{x} \sum_{i=1}^{n} \lambda_{i} \left\|\left(G_{i}^{T}\left(x ; W_{t}\right)-G_{i}^{S}\left(x ; W_{s}\right) \|_{2}^{2}\right.\right.
$$
匹配方式如下，值得注意的是，FSP矩阵计算两特征图大小一致，教师模型与学生模型FSP匹配的大小也要一致：

<img src="figure\image-20200714143935367.png" alt="image-20200714143935367" style="zoom:67%;" />

这里的实验更接近迁移学习的场景：

1. 教师模型预训练数据集
2. 利用FSP矩阵迁移知识
3. 用具体任务的损失训练学生模型

<img src="figure\image-20200714144315242.png" alt="image-20200714144315242" style="zoom:50%;" />

### 实验

方法提高优化速度（一点点）：

<img src="figure\image-20200714144827176.png" alt="image-20200714144827176" style="zoom:70%;" />

在Cifar10（左），Cifar100（中），以及迁移学习任务（小样本）中有不错的效果：

<img src="figure\image-20200714145133588.png" alt="image-20200714145133588" style="zoom:60%;" />


## **结论**

1. 提出一种建模处理问题流程的FSP矩阵来提取知识。 
2. 该方法有利于快速优化，提高知识蒸馏效果，在迁移学习中也有不错表现。

> **观点** 之前所说的，这个方法对维度匹配要求太高了，随意设计的学生模型可能和教师模型没有一个匹配的FSP矩阵。

**Ref:**  [Yim, Junho, et al. "A gift from knowledge distillation: Fast optimization, network minimization and transfer learning." *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*. 2017.](https://openaccess.thecvf.com/content_cvpr_2017/papers/Yim_A_Gift_From_CVPR_2017_paper.pdf)
