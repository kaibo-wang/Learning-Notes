# **Dreaming to Distill: Data-free Knowledge Transfer via DeepInversion**

<center>2020年07月09日</center>

<center>
    <span style="background:orange;border-radius:8px;color:white">&nbsp CVPR &nbsp</span> &nbsp &nbsp
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 生成模型 &nbsp</span> &nbsp &nbsp
    <span style="background:LightGreen;border-radius:8px;color:white">&nbsp 无数据 &nbsp</span> 
</center> 



## **摘要**

> 提出了一种通过逆向教师模型，利用BN层的信息，由随机噪声开始，反向优化得到条件样本，还通过最大化教师模型和学生模型logits之间的JS散度提高生成样本的多样性。在CIFAR10和ImageNet上都产生不错的生成效果，并在模型剪枝、知识迁移、持续学习三个任务上进行评估。

## **论文内容**

### 问题

在知识蒸馏的过程中，往往需要训练数据，但出于信息储存、传输、安全性等考虑，希望能够不利用训练数据，只利用训练好的教师模型进行知识蒸馏。

DeepDream对输入图像进行合成或变换，以生成对给定类别产生高输出响应的样本，实现方式时优化输入(随机噪声或自然图像)，其中使用了一些正则化，但该方法不限制中间表示，因此产生的梦境图像缺乏自然的图像形式。

<img src="figure\image-20200709115655057.png" alt="image-20200709115655057" style="zoom:33%;" />

因此，在DeepDream工作上进行改进生成高保真图像，并在模型剪枝、知识迁移、持续学习三个任务上取得不错效果。

### 方法

**DeepDream：**

通过优化下式生成样本 $\hat{x}$ ：
$$
\min_{\hat{x}} \mathcal{L}(\hat{x},y) + \mathcal{R}(\hat{x})
$$
其中，$\mathcal{L}(\hat{x},y)$ 是模型的分类损失（如交叉熵损失），而 $\mathcal{R}(\hat{x})$ 表示正则化，其中采用了总变分损失和 $\ell_2$ 范数损失：
$$
\mathcal{R}_{\text {prior }}(\hat{x})=\alpha_{\text {tv }} \mathcal{R}_{\text {TV }}(\hat{x})+\alpha_{\ell_{2}} \mathcal{R}_{\ell_{2}}(\hat{x})
$$

$$
\mathcal{R}_{\text {TV }}(x)=\sum_{i, j}\left(\left(x_{i, j-1}-x_{i, j}\right)^{2}+\left(x_{i+1, j}-x_{i, j}\right)^{2}\right)^{\frac{1}{2}}
$$

这样生成的图像依然与实际的分布有些距离。

**模型总体结构如图所示：**

<img src="figure\image-20200710003440224.png" alt="image-20200710003440224" style="zoom:35%;" />

**DeepInversion：**

通过扩展正则项，加入特征分布正则项，为了有效地加强各级特征的相似性，最小化特征图统计指标之间的距离，假设每批次第 $l$ 层的输入是正态分布：
$$
\mathcal{R}_{\text {feature }}(\hat{x})= \sum_{l}\left\|\mu_{l}(\hat{x})-\mathbb{E}\left(\mu_{l}(x) \mid \mathcal{X}\right)\right\|_{2} + \sum_{l}\left\|\sigma_{l}^{2}(\hat{x})-\mathbb{E}\left(\sigma_{l}^{2}(x) \mid \mathcal{X}\right)\right\|_{2}
$$
然而，某一批次原分布样本的均值和方差不能得到（训练集不可得到），但训练集总体的均值和方差被储存在网络的BN层中，用BN层中的均值方差代替上式，可以得到新的正则化：
$$
\mathcal{R}(\hat{x}) = \mathcal{R}_{\text {prior }} + \alpha_f \mathcal{R}_{\text {feature }}(\hat{x})
$$

> **补充：Batch Normalization**
>
> BN通过规范化与线性变换使得每一层网络的输入数据的均值与方差都在一定范围内，使得后一层网络不必不断去适应底层网络中输入的变化，从而实现了网络中层与层之间的解耦，允许每一层进行独立学习，有利于提高整个神经网络的学习速度。
>
> **训练：**
>
> 训练中，$\gamma$ , $\beta$ 为可学习的参数（线性变换），以恢复数据本身的表达能力，避免底层网络学习到的参数信息丢失。
> $$
> Z^{(l)}=W^{(l)} A^{(l-1)}+b^{(l)}
> $$
>
> $$
> \mu=\frac{1}{m} \sum_{i=1}^{m} Z^{(l)}_i
> $$
>
> $$
> \sigma^{2}=\frac{1}{m} \sum_{i=1}^{m}\left(Z^{(l)}_i-\mu\right)^{2}
> $$
>
> $$
> \tilde{Z}^{(l)}=\gamma \cdot \frac{Z^{(l)}-\mu}{\sqrt{\sigma^{2}+\epsilon}}+\beta
> $$
>
> $$
> A^{(l)}=g^{(l)}\left(\tilde{Z}^{(l)}\right)
> $$
>
> **测试：**
>
> 测试阶段使用训练期间均值与方差的无偏估计：
> $$
> \mu_{\text {test}}=\mathbb{E}\left(\mu_{\text {batch}}\right)
> $$
>
> $$
> \sigma_{\text {test}}^{2}=\frac{m}{m-1} \mathbb{E}\left(\sigma_{\text {batch}}^{2}\right)
> $$
>
> $$
> B N\left(X_{t e s t}\right)=\gamma \cdot \frac{X_{t e s t}-\mu_{t e s t}}{\sqrt{\sigma_{t e s t}^{2}+\epsilon}}+\beta
> $$

**Adaptive DeepInversion：**

为了生成多样的图片，通过加入多样性损失，鼓励学生模型和老师模型之间产生分歧：
$$
\mathcal{R}_{\text {compete }}(\hat{x})=1-\mathrm{JS}\left(p_{T}(\hat{x}), p_{S}(\hat{x})\right)
$$

$$
\mathrm{JS}\left(p_{T}(\hat{x}), p_{S}(\hat{x})\right)=\frac{1}{2}\left(\mathrm{KL}\left(p_{T}(\hat{x}), M\right)+\mathrm{KL}\left(p_{S}(\hat{x}), M\right)\right)
$$

$$
M = \frac{1}{2}\left(p_{T}(\hat{x}),p_{S}(\hat{x}\right))
$$

自适应深度反演需要学生参与，以增强图像多样性。它的竞争性和互动性有利于不断进化学生模型，逐渐迫使新的图像特征出现。

<img src="figure\image-20200709160107845.png" alt="image-20200709160107845" style="zoom:40%;" />

### 结果

在Cifar10的知识蒸馏上取得不错的成果：

<img src="figure\image-20200710003123858.png" alt="image-20200710003123858" style="zoom:60%;" />

在ImageNet上生成的样本比较真实，也能被其他模型识别：

<img src="figure\image-20200710003227389.png" alt="image-20200710003227389" style="zoom:40%;" />

关于模型剪枝、知识蒸馏、持续学习的实验参考原文。

其中，该方法实现持续学习（使学生模型学习新类别）的方法，通过优化学生模型生成样本与教师模型的距离（第一项，保持原有知识），新样本分类效果（第二项，如交叉熵损失），新样本的知识保持（第三项）。
$$
\mathcal{L}_{\mathrm{CL}}= \mathrm{KL}\left(p_{o}(\hat{x}), p_{k}(\hat{x})\right)+\mathcal{L}_{x e n t}\left(y_{k}, p_{k}\left(x_{k}\right)\right)+\mathrm{KL}\left(p_{o}\left(x_{k} \mid y \in \mathcal{C}_{o}\right), p_{k}\left(x_{k} \mid y \in \mathcal{C}_{o}\right)\right)
$$


## **结论**

1. 深度反演，可以合成高分辨率和高保真度的训练图像，不需要额外生成模型。

2. 利用学生模型，可以自适应提高生成样本多样性。

**Ref:**  [Yin, Hongxu, et al. "Dreaming to distill: Data-free knowledge transfer via DeepInversion." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.](https://openaccess.thecvf.com/content_CVPR_2020/papers/Yin_Dreaming_to_Distill_Data-Free_Knowledge_Transfer_via_DeepInversion_CVPR_2020_paper.pdf)

**Code:** [https: //github.com/NVlabs/DeepInversion](https: //github.com/NVlabs/DeepInversion)
