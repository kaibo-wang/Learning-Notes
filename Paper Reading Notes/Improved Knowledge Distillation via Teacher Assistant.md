# **Improved Knowledge Distillation via Teacher Assistant**

<center>2020年07月10日</center>

<center>
    <span style="background:pink;border-radius:8px;color:white">&nbsp 知识蒸馏 &nbsp</span> &nbsp &nbsp
    <span style="background:SkyBlue;border-radius:8px;color:white">&nbsp 理论 &nbsp</span> &nbsp &nbsp
    <span style="background:LightGreen;border-radius:8px;color:white">&nbsp 主题 &nbsp</span> 
</center> 



## **摘要**

> 知识蒸馏中，当学生与教师之间的结构差距较大时，学生网络的学习性能会下降。因此采用多阶段知识精馏，使用一个中等规模的网络（助教模型），以补充学生和教师之间的差距。

## **论文内容**

### 问题

知识蒸馏并不总是有效的，尤其是当老师和学生之间的结构差距很大的时候。一个由大型教师网络蒸馏得到的小型网络性能甚至不如由较小的教师网络蒸馏得到。

**解决思路：**在教师模型和学生模型之间引入助教模型，由教师模型蒸馏得到助教模型，由助教模型蒸馏学生模型。

<img src="figure\image-20200710105355732.png" alt="image-20200710105355732" style="zoom:40%;" />



### 实验

学生模型与教师模型差距较大，可能学习效果不好（学生模型大小为2）：

<img src="figure\image-20200710164722299.png" alt="image-20200710164722299" style="zoom:67%;" />

**原因：**

1. 随着教师模型复杂度提高，其表现也会提高，因此，可以为学生模型提供更好的监督。
2. 教师模型变得复杂，以至于即使得到暗示，学生也没有足够的能力或机制来模仿其行为。
3. 教师模型对数据的确定性增加，从而使其标签熵更低，削弱了通过匹配软标签实现的知识转移。

在早期，原因1使得学生模型效果提高，而后原因2，3使其效果下降。

加入助教模型后可以改善学生模型学习效果：

<img src="figure\image-20200710165106521.png" alt="image-20200710165106521" style="zoom:50%;" />

### 分析

**最佳的助教模型大小：**

助教模型准确率位于教师模型与学生模型准确率平均值。

<img src="figure\image-20200710165221105.png" alt="image-20200710165221105" style="zoom:40%;" />

**多助教模型：**

多个助教可以进一步提高知识蒸馏效果，但要考虑成本设计最佳的蒸馏路径。

<img src="figure\image-20200710165441307.png" alt="image-20200710165441307" style="zoom:50%;" />

**其他：**

1. 可以加入其他知识蒸馏方法中作为改进
2. 理论依据表示助教模型的效果（其中推导似乎存在错误（式5，6，7））
3. 对损失函数的梯度有平滑效果

**Ref:**  [Mirzadeh S I, Farajtabar M, Li A, et al. Improved Knowledge Distillation via Teacher Assistant[J]. arXiv preprint arXiv:1902.03393, 2019.](https://arxiv.org/pdf/1811.00995)