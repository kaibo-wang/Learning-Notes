# **Zero-shot Knowledge Transfer via Adversarial Belief Matching**

<center>2020年07月15日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp Nips &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 生成 &nbsp</span>
</center> 



## **摘要**

> 提出了一种可以在不使用任何数据的情况下进行知识蒸馏的方法。通过训练对抗生成器来搜索学生与老师的匹配不佳的图像，使用它们来训练学生模型以实现这一目标。此外，还提出了一种度量标准，用于量化决策边界附近的师生之间的信息匹配程度，并且观察到我们的少样本学生和老师之间的匹配程度要远高于从真实数据中精炼出来的学生和老师之间的匹配程度。

## **论文内容**

### 问题

在知识蒸馏的过程中，往往需要训练数据，但出于信息储存、传输、安全性等考虑，希望能够不利用训练数据，只利用训练好的教师模型进行知识蒸馏。

相关工作：归纳点和数据集提取（Inducing point methods and dataset distillation）。

### 方法

方法的思路比较简单，就是通过生成器最大化教师模型和学生模型输出概率分布的KL散度，但实验中有些细节：

1. 迭代中 $n_S > n_G$ 给学生更多的时间来匹配，并鼓励生成器在下一次迭代时探索输入空间的其他区域。

2. 生成样本效果是使学生模型输出高熵。

3. 考虑到教师模型和学生模型通常有类似的结构，在损失中加入AT：
   $$
   \mathcal{L}_{S}=D_{K L}\left(T\left(\boldsymbol{x}_{p}\right) \| S\left(\boldsymbol{x}_{p}\right)\right)+\beta \sum_{l}^{N_{L}}\left\|\frac{\boldsymbol{f}\left(A_{l}^{(t)}\right)}{\left\|\boldsymbol{f}\left(A_{l}^{(t)}\right)\right\|_{2}}-\frac{\boldsymbol{f}\left(A_{l}^{(s)}\right)}{\left\|\boldsymbol{f}\left(A_{l}^{(s)}\right)\right\|_{2}}\right\|
   $$

4. 研究了许多其他损失函数，但对表现没有帮助，包括样本多样性，样本一致性以及教师或学生的熵。

简单的实验如下所示，随机生成的样本点，通过最大化模型间KL散度，可以很快找到样本间的决策边界：

<img src="figure\image-20200715121755855.png" alt="image-20200715121755855" style="zoom:50%;" />

<img src="figure\image-20200715115917043.png" alt="image-20200715115917043" style="zoom:50%;" />

### 实验

在Cifar10和SVHN上均有不错的效果：

<img src="figure\image-20200715152657220.png" alt="image-20200715152657220" style="zoom:50%;" />

实验的样本据文中说不会出现由于远离分布而导致无法学习，样本使学生模型输出平均概率在0.3左右，类似对抗性样本，可视化后，确实接近难以分辨的纹理：

<img src="figure\image-20200715152849219.png" alt="image-20200715152849219" style="zoom:50%;" />

### 决策边界附近匹配

提出一种计算决策边界附近两模型输出匹配程度的曲线 $p_j \ curve$ 和指标 $\operatorname{MTE}\left(n e t_{A}, n e t_{B}\right)$

主要方法是在学生模型上生成对抗样本，计算教师模型同样被攻击的概率（迁移率），算法如上图右边所示，对各步骤取平均，以及除了正确类别外各个类别平均，得到指标 $\operatorname{MTE}\left(n e t_{A}, n e t_{B}\right)$：
$$
\operatorname{MTE}\left(n e t_{A}, n e t_{B}\right)=\frac{1}{N_{t e s t}} \sum_{n}^{N_{t e s t}} \frac{1}{C-1} \sum_{c}^{C-1} \frac{1}{K} \sum_{k}^{K}\left|p_{j}^{A}-p_{j}^{B}\right|
$$
<img src="figure\image-20200715154336876.png" alt="image-20200715154336876" style="zoom:50%;" />

## **结论**

1. 提出了一种新颖的对抗算法，在无数据的情况下进行知识蒸馏
2. 定义了决策边界附近两个网络之间的匹配度

> **观点** 用噪声学习得到的网络，往往难以提取主要的特征，在常规的分类任务上都没有很好的效果，这里可能经过学习的噪声足够的多，以此学到一定的特征，但个人认为，模型的数据分布，或者问题域，本身就是最丰富的信息，主动学习中有采样偏差一说，就是说采样不符合数据的分布，会使模型训练不佳。

**Ref:**  [Micaelli, Paul, and Amos J. Storkey. "Zero-shot knowledge transfer via adversarial belief matching." Advances in Neural Information Processing Systems. 2019.](http://papers.nips.cc/paper/9151-zero-shot-knowledge-transfer-via-adversarial-belief-matching.pdf)