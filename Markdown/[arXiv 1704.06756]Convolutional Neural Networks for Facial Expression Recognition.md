[arXiv 1704.06756]Convolutional Neural Networks for Facial Expression Recognition
====================================================================

  这篇论文整体来说，比较水,但还是说说吧.
### ABSTRACT
  本文提出了一种卷积神经网络的人脸表情识别(7类别),同时，讲CNN的特征与HOG特征进行融合，训练分类器，最终发现HOG特征对于整体的效果没有大的影响。

### Methods
  模式采用的卷积，BN，RELU + maxpooling的结构，数据集为FER2013,作者在cnn的fc(512)后加入了HOG特征，做拼接，完成两个特征的融合。
