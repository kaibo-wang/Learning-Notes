# **Optimization for deep learning: theory and algorithms**

<center>2020年6月10日</center>

<center>
    <span style="background:pink;border-radius:8px;color:white">&nbsp 综述 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 优化 &nbsp</span>
</center> 



## **摘要**

> 为什么神经网络可以被成功训练？本文概述了神经网络训练的优化算法和理论。首先，讨论了梯度爆炸/消失的问题和更一般的问题，讨论了实际的解决方案，包括初始化和归一化方法。其次，回顾了用于训练神经网络的一般优化方法，如SGD、自适应梯度方法和分布式方法，以及这些算法的现有理论结果。第三，我们回顾了现有的关于神经网络训练全局问题的研究，包括规避局部极小值、模式连通性、彩票假设和无限宽分析的结果。

## **论文内容**

### 介绍

​	现在成型的深度神经网络训练包含三个部分：

 1. **合适的网络结构：**结构；激活函数

 2. **训练算法：**SGD；动量、自适应步长

 3. **训练技巧：**Normalization层、跨接

​	以上很多是实验中总结得到的技巧，背后的理论依据可以被分为三个主要部分：控制李普希茨常数、加速收敛、更好的收敛位置，但也有一些技巧还难以理解。

<img src="figure\image-20200609142001788.png" alt="image-20200609142001788" style="zoom:67%;" />

​	为了理解神经网络结构如何影响优化算法的设计和分析，文章从理论的角度进行研究。理论性的有监督学习包括三个方面：

1. **表示：**找到一族足够复杂的函数族；
2. **优化：**最小化损失函数来确定参数；
3. **泛化：**在未知的测试样本进行预测；

​	测试误差可以被分解为上述三部分的误差，为了简化，研究优化过程中通常不考虑表示误差和泛化误差。

​	优化问题包括三个方面：

* **局部：**
  * **收敛：**梯度消失、梯度爆炸
  * **收敛速度**
* **全局：**
  * **全局优化质量：**避免局部最优点

### 梯度下降法

​	损失函数表示为：
$$
\min _{\theta} F(\theta) \triangleq \frac{1}{n} \sum_{i=1}^{n} \ell\left(y_{i}, f_{\theta}\left(x_{i}\right)\right)
$$
​	**梯度下降法（GD）：**
$$
\theta_{t+1} = \theta_t - \eta_t \nabla F(\theta_t)
$$
​	**随机梯度下降（SGD）:**
$$
\theta_{t+1} = \theta_t - \eta_t \nabla F_i(\theta_t)
$$
​	利用反向传播计算梯度：
$$
F_{0}(\theta)=\left\|y-W^{L} \phi\left(W^{L-1} \ldots W^{2} \phi\left(W^{1} x\right)\right)\right\|^{2}
$$
​	令$h$表示激活前的值，$z$表示激活后的值，$D^l=\text{diag}(\phi^\prime(h_1^l),...,\phi^\prime(h_{d_l}^l))$，误差$e=2(h^L-y)$，则对权重$W^l$的梯度表示为：
$$
\frac{\partial F_{0}}{\partial W^{l}}=\left(W^{L} D^{L-1} \ldots W^{l+2} D^{l+1} W^{l+1} D^{l}\right)^{T} e\left(z^{l-1}\right)^{T}, \quad l=1, \ldots L
$$
​	上述梯度的计算更新，可以用类似动态规划的迭代方法完成——将误差从后往前传递，成为反向传播算法。

#### 收敛性

​	优化算法的收敛定义：参数趋近全局最小值？趋近驻点？ 目标函数值收敛？

​	**常规指标：**梯度序列趋于0

​	**收敛理论：**前提是模型具有李普希茨平滑的导数（难以保证）：
$$
\|\nabla F(w)-\nabla F(v)\| \leq \beta\|w-v\|
$$
​	则：任意极限点都是驻点；如果目标函数值存在下界，则梯度收敛至0.

​	如果所有迭代都是有界的，那么具有适当恒定步长的GD会收敛。

### 神经网络的技巧

#### 由于梯度爆炸或消失导致的缓慢收敛



## **结论**

1. 结论1

**Ref:**  [Sun R. Optimization for deep learning: theory and algorithms[J]. arXiv preprint arXiv:1912.08957, 2019.](https://arxiv.org/pdf/1912.08957.pdf)

