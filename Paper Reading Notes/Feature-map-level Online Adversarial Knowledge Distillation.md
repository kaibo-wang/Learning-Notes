# **Feature-map-level Online Adversarial Knowledge Distillation**

<center>2020年07月16日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp ICML &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 生成 &nbsp</span> &nbsp &nbsp
    <span style="background:LightGreen;border-radius:8px;color:white">&nbsp 在线 &nbsp</span> 
</center> 

## **摘要**

> 提出了一种在线知识提取方法，使用对抗训练框架来传递概率和特征图的知识，通过使用鉴别器来区分不同网络的特征图分布，同时训练多个网络。每个网络都有其对应的鉴别器，该鉴别器将自己模型的特征映射鉴别为假，而将其他网络的鉴别为真。通过训练一个网络来欺骗相应的鉴别器，就可以学习另一个网络的特征图分布。此外提出了一种新颖的循环学习方案，用于训练两个以上的网络。

## **论文内容**

### 问题

传统的蒸馏方法需要预先训练功能强大的教师网络，并执行单向转移到相对较小且未经训练的学生网络。在线蒸馏中，所有网络都同时互相学习，不仅会经交叉熵损失，也通过模仿损失训练，以向同行学习。

但是，上述在线蒸馏方法仅利用logit信息。其中的挑战在于，logit信息反应最终的分类结果，多个模型还是在追求最终的分类结果，而这个结果是不会变的。但是训练过程中，模型的特征图会很快的变化，直接通过距离度量来模仿特征图，并不能很好的得到有效的学习（你仍然在学习别人旧的解法，别人又去学习你旧的解法），因此通过对抗生成式学习的方法来匹配模型特征图的分布：

<img src="figure\image-20200716120517546.png" alt="image-20200716120517546" style="zoom:50%;" />

由于在每次训练迭代中对鉴别器进行了训练以区分网络的特征图分布之间的差异（包含针对不同输入图像的特征图的历史记录），因此通过欺骗鉴别器，网络将学习到共同训练网络的变化特征图分布。

交换特征图分布的知识有助于网络收敛到更好的特征图流形，从而更好地泛化。

**损失函数**

**分类损失**

CE损失训练网络预测正确的真相标签，而相互损失则尝试匹配网络的输出，从而使网络能够在logit层面共享知识。
$$
\mathcal{L}_{\text {logit}}^{1}=\mathcal{L}_{c e}\left(y, \sigma\left(z_{1}\right)\right)+T^{2} \times \mathcal{L}_{k l}\left(\sigma\left(z_{2} / T\right), \sigma\left(z_{1} / T\right)\right)
$$

$$
\mathcal{L}_{\text {logit}}^{2}=\mathcal{L}_{c e}\left(y, \sigma\left(z_{2}\right)\right)+T^{2} \times \mathcal{L}_{k l}\left(\sigma\left(z_{1} / T\right), \sigma\left(z_{2} / T\right)\right)
$$

**特征损失**
$$
\mathcal{L}_{D_{1}}=\left[1-D_{1}\left(T_{2}\left(G_{2}(x)\right)\right)\right]^{2}+\left[D_{1}\left(T_{1}\left(G_{1}(x)\right)\right)\right]^{2}
$$

$$
\mathcal{L}_{G_{1}}=\left[1-D_{1}\left(T_{1}\left(G_{1}(x)\right)\right)\right]^{2}
$$

其中 $T$ 表示可能的对特征图进行维度统一的模块。

通过基于logit的损失来更新参数一次，然后通过基于特征图的损失来再次更新参数。我们为每个损失项分别优化的原因是，需要使用不同的学习率，对抗性损失需要较慢的学习速度。

<img src="figure\image-20200716142812835.png" alt="image-20200716142812835" style="zoom:50%;" />

**循环学习框架**

多个网络，为了避免使用过多的判别器，可以环形学习，每个网络学习它的上一个网络，以此循环。

<img src="figure\image-20200716152624785.png" alt="image-20200716152624785" style="zoom:40%;" />

### 实验

无论与离线或在线算法比较都有显著提高：

**离线**（用离线的损失函数代替在线的）：

<img src="figure\image-20200716180646318.png" alt="image-20200716180646318" style="zoom:50%;" />

**在线：**

<img src="figure\image-20200716180736151.png" alt="image-20200716180736151" style="zoom:50%;" />

### 分析

值得注意的是，该方法可以在特征图级别上传递知识，同时保留每个单独网络已学习的特征，而不会破坏它自己获得的知识。

在比较两模型特征图的距离时，该算法是相似度最低的，但取得不错的性能。

<img src="figure\image-20200716182109798.png" alt="image-20200716182109798" style="zoom:67%;" />


## **结论**

1. 提出一种在线知识提炼方法，该方法不仅利用logit，还利用卷积层的特征图。 
2. 该方法不是通过使用距离损失直接对齐特征图，而是通过使用判别器进行对抗训练来学习它们的分布，来传递特征图的知识。
3. 提出了一种循环学习方案，用于同时训练两个以上的网络。

> **观点** 用判别器接近分布的方法比较有意思，但训练这么多判别器是否有必要？二分类可以用一个分类器，多分类也可以。

**Ref:**  [Chung I, Park S U, Kim J, et al. Feature-map-level online adversarial knowledge distillation[J]. arXiv preprint arXiv:2002.01775, 2020.](https://arxiv.org/pdf/2002.01775.pdf)