# **Paraphrasing Complex Network: Network Compression via Factor Transfer**

<center>2020年07月11日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp Nips &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 特征 &nbsp</span> 
</center> 


## **摘要**

> 在本文中，我们提出了一种新颖的知识转移方法，该方法使用卷积运算来解释教师的知识并将其翻译给学生。通过两个卷积模块来完成，这两个模块分别被称为释义器和翻译器。以无监督的方式训练释义器以提取表示教师网络释义信息的教师因子。位于学生网络中的翻译器提取学生因子，并通过模仿他们来帮助翻译教师因子。使用因子转移方法训练的学生网络优于使用常规知识转移方法训练的学生网络。

## **论文内容**

### 问题

虽然传统知识蒸馏方法可以提供相当不错的性能，但是直接转移教师的输出却忽略了教师网络和学生网络之间的固有差异，例如网络结构，通道数量和初始条件。因此，有必要对教师的知识进行一定解释，以使学生模型更好理解。

为了解决这个问题，提出了一种新颖的知识转移方法，使学生和教师网络都具有可迁移的知识，在本文中我们将其称为“因子”。该方法通过训练可以良好提取因子并匹配这些因子的神经网络，其中从教师网络中提取因子的神经网络称为释义器，而从学生网络中提取因子的神经网络称为翻译器。

以无监督的方式训练了释义器，期望它能提取与有监督的损失项所获得的知识不同的知识。学生方面，利用翻译器训练学生网络，以吸收从释义者中提取的因子。

<img src="figure\image-20200711151644899.png" alt="image-20200711151644899" style="zoom:40%;" />

### 释义器训练

释义器是在教师模型Res Block（输出维度为 $\mathbb{R}^{H \times W}$）之后连接的模型，模型本质上是一个自编码器，旨在将输出的特征图降维，降维之后还原**输入图像**，因此先经过多个卷积层降维，之后再通过多个转置卷积层升维，中间降维的结果就被称为“因子”。

中间“因子”的通道数由教师模型特征通道数决定，如果教师网络生成 $m$ 个特征图，将因子通道的数量调整为 $m \times k$ 。

<center><img src="figure\image-20200712101754185.png" alt="image-20200712101754185" style="zoom:40%;" /></center> 

因此损失函数为重建误差，$P$ 表示释义器：
$$
\mathcal{L}_{rec} = ||x-P(x)||^2
$$

### 翻译器训练

翻译器也是连接在学生模型后的子模型，与学生模型**同时训练**，这里翻译器扮演了一个缓冲区的角色，通过对学生网络特征图的重新描述，使学生网络从直接学习教师网络输出的负担中解脱出来。

模型的损失包括基本的分类损失 $\mathcal{L}_{cls}$ （交叉熵或其他）和因子匹配损失 $\mathcal{L}_{FT}$：
$$
\mathcal{L}_{s} = \mathcal{L}_{cls} +  \beta \mathcal{L}_{FT}
$$

$$
\mathcal{L}_{F T}=\left\|\frac{F_{T}}{\left\|F_{T}\right\|_{2}}-\frac{F_{S}}{\left\|F_{S}\right\|_{2}}\right\|_{p}
$$

### 实验结果

在Cifar10和Cifar100上均表现不错：

<img src="figure\image-20200713004228678.png" alt="image-20200713004228678" style="zoom:50%;" />

<img src="figure\image-20200713004621833.png" alt="image-20200713004621833" style="zoom:50%;" />

通过对比实验表示，释义器和翻译器的存在都是有必要的：

<img src="figure\image-20200713004742013.png" alt="image-20200713004742013" style="zoom:50%;" />

## **结论**

1. 提出一种提取有意义的特征（“因子”）的知识蒸馏方式。
2. 通过释义器模型和翻译器模型提高学生模型效率。

> **观点** 有一个值得考虑的问题在于，既然释义器将教师模型的知识降维表示，这作为一种教师向学生传递的理解，那学生直接用释义器匹配教师提出的理解不就足够了吗？难道同样的知识需要不同的解释吗？在学生端训练解释器是否有必要？有没有可能学生模型并没有模仿教师模型，而翻译器将学生模型杂乱的输出拟合到教师的理解？
>
> 事实上，这我个人找不到很有说服力的理由，但如果学生端不训练解释器，那和注意力转移的方法便没有本质区别，最后一个问题应该也不成立，解释器的能力一般不足以完成这样的任务。因此，文中说解释器起到缓冲的作用，这点是成立的，其改善效果的原因，会不会也在于用训练解释器分担了一部分难以处理的损失（dirty work）。
>
> 那这样而言，整个模型所起的效果就不是知识的释义，而解释器模型与释义器模型之间的差异，是否是教师模型和学生模型之间能力的量化呢？

**Ref:**  [Kim J, Park S U, Kwak N. Paraphrasing complex network: Network compression via factor transfer[C]//Advances in neural information processing systems. 2018: 2760-2769](http://papers.nips.cc/paper/7541-paraphrasing-complex-network-network-compression-via-factor-transfer.pdf)
