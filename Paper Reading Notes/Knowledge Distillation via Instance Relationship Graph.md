# **Knowledge distillation via instance relationship graph**

<center>2020年07月13日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp CVPR &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 关系提取 &nbsp</span> 
</center> 



## **摘要**

> 提出用于知识蒸馏的实例关系图（IRG），IRG通过对包括实例特征，实例关系和特征空间转换三种知识进行建模，首先将实例特征和实例关系分别视为图顶点和边构造IRG来建模一个网络层的提取知识。其次，提出IRG变换来建模跨层的特征空间变换。最后，通过损失函数使学生的IRG模仿教师的IRG的结构。该方法通过IRG有效地捕获了整个网络的知识，从而对不同的网络架构显示出稳定的收敛性和强大的鲁棒性。

## **论文内容**

### 问题

将模型的输出或是中间层的特征表示为实例特征，当前方法主要将实例特征作为知识由教师模型转移到学生模型，并未考虑实例关系，但它们有助于减少类内差异，并扩大特征空间中的类间差异。此外实例特征的知识蒸馏方式在教师模型和学生模型之间存在结构差异时，会有较大的性能损失。再者，这些方法只提取教师模型的某些特定层，而不考虑推理过程。

实例关系提供了特征分布的充分的信息，并使提炼的知识能够指导与教师使用不同架构的学生网络。为了避免过紧的约束，引入了跨层的特征空间转换作为知识的第三种类型，并通过IRG来对该知识进行建模。与对中间层教师实例特征的密集拟合相比，特征空间变换的描述更为宽松。

### 方法

**实例关系图**

<img src="figure\image-20200713212750437.png" alt="image-20200713212750437" style="zoom:40%;" />

实例关系图（IRG）是建立在网络的层上，用 $f_l(x_i)$ 表示第 $l$ 的输出特征，则该层的IRG表示为：
$$
I R G_{l}=\left(\mathcal{V}_{l}, \mathcal{E}_{l}\right)=\left(\left\{f_{l}\left(x_{i}\right)\right\}_{i=1}^{I}, \mathbf{A}_{l}\right)
$$

$$
\mathbf{A}_{l}(i, j)=\left\|f_{l}\left(x_{i}\right)-f_{l}\left(x_{j}\right)\right\|_{2}^{2}, i, j=1, \ldots, I
$$

由此可以定义，不同层之间IRG的转化：
$$
I R G-t_{l_{1} l_{2}} =\left(\operatorname{Trans}\left(\mathcal{V}_{l_{1}}, \mathcal{V}_{l_{2}}\right), \operatorname{Trans}\left(\mathcal{E}_{l_{1}}, \mathcal{E}_{l_{2}}\right)\right) =\left(\boldsymbol{\Lambda}_{l_{1}, l_{2}}, \boldsymbol{\Theta}_{l_{1}, l_{2}}\right)
$$

$$
\boldsymbol{\Lambda}_{l_{1}, l_{2}}(i, i) =\left\|f_{l_{1}}\left(x_{i}\right)-f_{l_{2}}\left(x_{i}\right)\right\|_{2}^{2}, \quad i=1, \ldots, I
$$

$$
\boldsymbol{\Theta}_{l_{1}, l_{2}}=\|  \mathbf{A}_{l_{1}}-\mathbf{A}_{l_{2}} \|_{2}^{2}
$$

**损失函数**

​	**IRG匹配：**
$$
L_{I R G}(\boldsymbol{x})
=\lambda_{1} \cdot \sum_{i=1}^{I}\left\|f_{L}^{T}\left(x_{i}\right)-f_{l_{M}}^{S}\left(x_{i}\right)\right\|_{2}^{2} 
+\lambda_{2} \cdot\left\|\mathbf{A}_{L}^{T}-\mathbf{A}_{l_{M}}^{S}\right\|_{2}^{2}
$$
教师模型与学生模型的层匹配，包含两种模式：一对一模式和一对多模式：

<img src="figure\image-20200713210730808.png" alt="image-20200713210730808" style="zoom:60%;" />

对于学生模型和教师模型层数相同的情况下，采用一对一模型，而不相等情况下，考虑到教师模型最后一层蕴含更多信息，可以一对多指导学生模型的多个层，因此改写损失函数：
$$
L_{I R G}(\boldsymbol{x})=\lambda_{1} \cdot L_{l o g i t s}(\boldsymbol{x})+\lambda_{2} \cdot \sum_{l_{M} \in \mathbf{L}_{M}}\left\|\mathbf{A}_{L}^{T}-\mathbf{A}_{l_{M}}^{S}\right\|_{2}^{2}
$$
​	**IRG转化匹配：**
$$
L_{I R G-t}(\boldsymbol{x}) 
=\left\|\boldsymbol{\Lambda}_{l_{1}, l_{2}}^{T}-\boldsymbol{\Lambda}_{l_{3}, l_{4}}^{S}\right\|_{2}^{2}+\left\|\boldsymbol{\Theta}_{l_{1}, l_{2}}^{T}-\boldsymbol{\Theta}_{l_{3}, l_{4}}^{S}\right\|_{2}^{2}
$$
出于计算资源的考虑，移除关于边的匹配：
$$
L_{I R G-t}(\boldsymbol{x}) 
=\left\|\boldsymbol{\Lambda}_{l_{1}, l_{2}}^{T}-\boldsymbol{\Lambda}_{l_{3}, l_{4}}^{S}\right\|_{2}^{2}
$$
综上所述，综合损失表示为：
$$
L_{MTK}(\boldsymbol{x}) = L_{GT}(\boldsymbol{x}) + \lambda_1 L_{logits}(\boldsymbol{x}) + \lambda_{2} \cdot \sum_{l_{M} \in \mathbf{L}_{M}}\left\|\mathbf{A}_{L}^{T}-\mathbf{A}_{l_{M}}^{S}\right\|_{2}^{2} + \lambda_3 \sum_{l_{1},l_2,l_3,l_4 \in \mathbf{L}_{Tran}}\left\|\boldsymbol{\Lambda}_{l_{1}, l_{2}}^{T}-\boldsymbol{\Lambda}_{l_{3}, l_{4}}^{S}\right\|_{2}^{2}
$$
$L_{GT}(\boldsymbol{x})$ 表示分类损失，$L_{logits}(\boldsymbol{x})$ 表示最后一层的IRG损失，整体结构如图：

<img src="figure\image-20200713212702850.png" alt="image-20200713212702850" style="zoom:50%;" />

### 实验结果

首先进行超参数和batch size调优：

<img src="figure\image-20200713221038529.png" alt="image-20200713221038529" style="zoom:50%;" />

实现中，两个重要的细节：

- IRG匹配的是教师的最后一层与学生的后三层；
- IRG-t匹配的是网络的前三层

实验结果基本达到SOTA（毕竟复杂）

<img src="figure\image-20200713221310950.png" alt="image-20200713221310950" style="zoom:60%;" />

## **结论**

1. 运用三种知识进行知识蒸馏，包括实例特征，实例关系以及跨层的特征空间转换。

2. 引入了不同的损失来监督学生网络的训练。

> **观点** 看起来不同层的匹配是一个问题，文中匹配的选择还是较为随意，如果可以更好的匹配需要迁移知识的层，或许会有所改进？

**Ref:**  [Liu, Yufan, et al. "Knowledge distillation via instance relationship graph." Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2019.](https://openaccess.thecvf.com/content_CVPR_2019/papers/Liu_Knowledge_Distillation_via_Instance_Relationship_Graph_CVPR_2019_paper.pdf)