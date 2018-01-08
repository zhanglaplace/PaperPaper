[arXiv 1710.03144]Island Loss for Learning Discriminative Features in Facial Expression
==============================================================

### ABSTRACT
  作者在CenterLoss的基础上，提出了一个新的Loss，在关注类别的类内距离的同时，优化类间距离，使得每个类别拥有更大的margin,从而迫使网络能够学习到更具判别性的特征。
#### 当前问题
  在环境不可控（光照，姿态，遮挡，人物状态）等条件下，不同表情间的类间距离往往会大于类内距离。同时因为高的类内距离，同一类别的样本分布趋势呈现出了径向(NormFace中提到的softmax的Scale不变性)

  ![1.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_1.png)

  $$ 不同loss情况下类别特征空间分布情况 $$

#### 本文贡献
  本文主要有一下贡献:</br>
- 提出了一个新的loss函数，目的是能够得到更具判别性的特征，从而使得类内距离小，类间距离大。
- 设计了一个机遇IslandLoss的卷积神经网络框架，完成人脸面部表情的识别。

  作者在三个数据库上进行了实验，(CK+,Oulu-CASIA 和MMI),并且在SFEW的数据库上进行了验证。对比与传统的SoftmaxLoss和CenterLoss，新提出的Loss效果更好。

### 理论方法
- $softmax loss$: </br>
  $$ L = -\frac{1}{N}\sum_{j=1}^{N}{y_ilog{\frac{exp(z_{y_i})}{\sum_{k=1}^{c}{exp(z_k)}}}} $$

- $centerLoss$，在softmax的基础上，新添加了一种loss：
  $$ L_c = \frac{1}{2}\sum_{i=1}^{n}||x_i-c_{yi}|| $$
  其中，$yi$为样本i的类心特征向量。因此整体的Loss为
  $$ L = L_{softmax} + \lambda L_c $$
  反向更新的时候，每个batch内，每个样本仅仅负责更新该类的类心.

-  $IslandLoss$ ,在softmax的基础上，优化类间距离:
  $$ L_{island}=L_c+ \lambda_{1} \sum_{c_{j} \in N}\sum_{c_k \in N ,c_k \neq c_j}{\frac{c_k \cdot c_j}{||c_k||_2||c_j||_2}+1} $$
  公式看着比较复杂，实际上即每个类心求余弦距离，+1 使得范围为0-2，越接近0表示类别差异越大，从而优化Loss即使得类间距离变大。当类别过多时，loss容易溢出.

  ![3.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_3.png)
  $$ islandloss 前向公式 $$
  $$ L = L_{softmax}+ \lambda L_{island} $$
  反向更新过程即求导过程,实际简化为x/||x||的求导,需要注意i==j和i!=j.
  ![4.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_4.png)
  $$ islandloss 反向更新公式 $$

  ![5.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_5.png)
  $$ islandLoss层次类心的迭代与更新过程 $$

### 实验过程
   实验包含预处理，卷积神经网络超参数。
#### 数据预处理
  根据人脸特征点的位置，以双眼为中心，进行人脸对齐。人脸图像缩放到60*60，同时，图像cropSize为48\*48,图像随机旋转-10°~10°，并进行水平flip，从而增加训练样本的数量.

#### 卷积神经网络结构
  3个卷积层后接Prelu和BN，每一个bn的后面接一个MaxPooling，最终接入两个全连接层，softmax和Islandloss共同驱动最终的loss,采用sgd+momentum batch_size为300，weight_decay设置为0.05,每个全连接后接dropout层。
  ![2.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_2.png)
  $$ 卷积神经网络结构图$$

### 实验结果
   作者做了一个简单的可视化，并统计了不同loss中特征类心的距离.
  ![10.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_10.png)
  $$ 特征可视化 $$

  ![16.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_16.png)
  $$ 不同loss的类别距离 $$

- CK+: 7表情94.35%
  ![6.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_6.png)
  $$ ck+数据库上islandloss混淆矩阵 $$
  ![7.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_7.png)
  $$ ck+数据库上不同算法准确率对比 $$

- MMI：6表情70.67%
  ![8.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_8.png)
$$ MMI数据库上islandloss混淆矩阵 $$
  ![9.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_9.png)
$$ MMI数据库上不同算法准确率对比 $$

- Ouli-CASIA: 6表情77.29%
  ![11.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_11.png)
$$ Oulu-CASIA数据库上islandloss混淆矩阵 $$
  ![12.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_12.png)
$$ Oulu-CASIA数据库上不同算法准确率对比 $$

- SFEW: 验证集52.52% 测试集 59.41%
  ![13.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_13.png)
$$ SFEW数据库验证集上islandloss混淆矩阵 $$
  ![14.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_14.png)
$$ SFEW数据库测试集上islandloss混淆矩阵  $$
  ![15.png](https://github.com/zhanglaplace/PaperPaper/blob/master/Expression/imgs_2/Island%20Loss%20for%20Learning%20Discriminative%20Features%20in%20Facial%20Expression%20Recognition_15.png)
$$ SFEW数据库不同算法准确率对比 $$


### 总结
  emmmmm,作者在centerloss的启发下，提出了softmax，并且以余弦距离衡量类间分布，相比于centerloss和传统的softmaxloss确实有提高，在网络上，bn或许对网络的性能有损失，而且具体的参数 $\lambda$也不好把握，小类别可以尝试。

  >本文作者： 张峰 </br>
  >本文链接： [http://www.enjoyai.site/2018/01/08/](http://www.enjoyai.site/2018/01/08/Image%20based%20Static%20Facial%20Expression%20Recognition%20with%20Multiple%20Deep%20Network%20Learning/) </br>
  >版权声明： 本博客所有文章，均采用 CC BY-NC-SA 3.0 许可协议。转载请注明出处！
