# **Invertible Residual Networks**

<center>2020年6月9日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp ICML &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 图像 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 生成 &nbsp</span> &nbsp &nbsp
    <span style="background:LightGreen;border-radius:8px;color:white">&nbsp 模型结构 &nbsp</span> 
</center> 


## **摘要**

> 利用巴拿赫不动点理论，证明在约束网络李普希茨常数的情况下，可以构建 $\mathbb{R}^{d} \to \mathbb{R}^{d}$ 的可逆的残差网络，因此该模型也可以作为生成模型，通过极大似然法生成样本，为了提高效率，提出了雅可比矩阵对数行列式的近似算法，并给出截断误差上界。

## **论文内容**

### 问题

传统残差网络只能由输入到输出，由于结果中存在重叠交叉，不构成输入到输出的双射，因此不能由输出逆推得到输入，以开展生成等任务，由此设计了满足双射要求的可逆残差网络

<img src="figure\image-20200609010427346.png" alt="image-20200609010427346" style="zoom:67%;" />

### 可逆残差网络

**充分条件** 卷积神经网络表示为$F_\theta = (F_\theta^1 * F_\theta^2 ... *F_\theta^T)$，其中$F_\theta^t=I+g_{\theta_t}$，可逆的充分条件是：
$$
\text{Lip}(g_{\theta_t}) < 1 \ \  for \ all \ t=1,...,T
$$
根据巴拿赫不动点理论有迭代算法$x^{n+1}:=y-g(x^n)$，收敛速度：
$$
\left\|x-x^{n}\right\|_{2} \leq \frac{\operatorname{Lip}(g)^{n}}{1-\operatorname{Lip}(g)}\left\|x^{1}-x^{0}\right\|_{2}
$$
满足条件的残差网络前向后向都是李普希茨约束的。

**满足李普希茨约束** 通过约束卷积层权重的谱范数可以约束李普希茨常数：
$$
\text{Lip}(g) < 1, \ if \ ||W_i||_2 < 1
$$
使用幂迭代法可以得到$||W_i||_2$的近似$\tilde{\sigma}$，因此直接对权重正则化：
$$
\tilde{W}_{i}=\left\{\begin{array}{ll}
c W_{i} / \tilde{\sigma}_{i}, & \text { if } c / \tilde{\sigma}_{i}<1 \\
W_{i}, & \text { else }
\end{array}\right.
$$

**注意：** 这不保证李普希茨约束，但选取合适的$c$通常是满足的

### 作为生成模型

采样$z \sim p_z(z)$，而$x=\Phi(z)$，则可以通过$\Phi$函数的逆函数$F$得到样本$x$的似然：
$$
\ln p_{x}(x)=\ln p_{z}(z)+\ln \left|\operatorname{det} J_{F}(x)\right|
$$
然而右式第二项在高维情况下难以计算，因此化简得：
$$
\ln p_{x}(x)=\ln p_{z}(z)+\operatorname{tr}\left(\ln \left(I+J_{g}(x)\right)\right)
$$

$$
\operatorname{tr}\left(\ln \left(I+J_{g}(x)\right)\right)=\sum_{k=1}^{\infty}(-1)^{k+1} \frac{\operatorname{tr}\left(J_{g}^{k}\right)}{k}
$$

取级数的有限项就可以作为近似，其中$\operatorname{tr}\left(J_{g}^{k}\right)$可以通过随机近似得到：

对于方阵$A$，方阵的迹可以如下估计：
$$
\operatorname{tr}(A)=\mathbb{E}_{p(v)}\left[v^{T} A v\right]
$$
其中随机变量$v$满足期望为0，方差为$I$

用$P S\left(J_{g}, n\right)$表示$n$项的截断，则估计误差上界：
$$
\left|P S\left(J_{g}, n\right)-\ln \operatorname{det}\left(I+J_{g}\right)\right| 
\leq -d\left(\ln (1-\operatorname{Lip}(g))+\sum_{k=1}^{n} \frac{\operatorname{Lip}(g)^{k}}{k}\right)
$$

***相关证明的推导思路：** 李普希茨常数 -> 谱半径 -> 特征值 （详见原文）

## **结论**

1. 作为识别模型，准确率仅比不可逆的残差网络略低，好于其他可逆结构

2. 生成效果不错，略差于其他可逆结构，原因可能来自于截断误差

部分生成样本如图：

<center><img src="figure\image-20200609111720451.png" alt="image-20200609111720451" style="zoom:67%;" /></center>

   

> **观点** 个人认为，可逆残差网络最麻烦的地方不在于李普希茨的约束，而在于要在整个结构中保持$\mathbb{R}^{d} \to \mathbb{R}^{d}$的映射，而卷积网络中有许多的降维操作——降采样、较大卷积核，都会受到限制或是需要额外的处理，完全保持可逆的结构是一件很麻烦的事，或许可以用一些其他方式，牺牲一部分可逆性，提高模型结构的灵活度。

**Ref:**  [Behrmann J, Grathwohl W, Chen R T Q, et al. Invertible residual networks[J]. arXiv preprint arXiv:1811.00995, 2018.](https://arxiv.org/pdf/1811.00995)