---
title: ImageCaption从入门到放弃之二_attention机制
tags:
  - caption
  - seq2seq
categories:
  - Caption
mathjax: true
date: 2017-03-12 20:55:43
---

前一篇文章[ImageCaption从入门到放弃之一_任务综述](https://liangguirong5.github.io/2017/03/01/image_caption/)中引向paperweekly的链接，介绍了传统的基于seq2seq的caption模型，这个模型来自机器翻译，因此机器翻译中的attetion机制在我们的caption任务中也可以借鉴，很多论文就是围绕这个attention该怎么加而展开的，我们主要介绍下面两篇比较有代表性的论文：

<!-- more -->

Xu, K., & Bengio, Y. (2016, April 20). Show, Attend and Tell: Neural Image Caption Generation with Visual Attention. *arXiv.org*.

Lu, J., & Socher, R. (2016, December 7). Knowing When to Look: Adaptive Attention via A Visual Sentinel for Image Captioning. *arXiv.org*.

注意：这个两篇文章的符号不统一，请注意区分。

# Neural Image Caption Generation with Visual Attention

第一篇论文是Bengio课题组的一份工作，主要是在seq2seq进行image caption的基础上，引入attention机制，强调生成caption时，不同时刻的关注点应该在不同的图像区域。 下图是模型的整体框架。

![](https://ww2.sinaimg.cn/large/006tNc79ly1fdkdc2lxybj31gq0o6afg.jpg)

其中，encoder部分是一个卷积网络，目的是获取原图像的L个特征，每个特征是一个D维的向量，用以表示原图的某一部分。
$$
a=\{a_1,…,a_L\}, a_i\in R^D
$$
原文采用VGG网络的第四层卷积结果作为这些特征向量，共有512个14*14的feature map，flatten成512个196维的特征。

decoder部分是一个LSTM，叠加使用了attention机制。本文的分别使用了hard和soft两种attention。

## Stochastic “Hard” attention

用t表示生成第t个词的时刻，$s_{t,i}$是一个布尔量，表示第t时刻第i个图像特征$a_i$是否为注意力特征，服从多元伯努利分布，如下式所示
$$
p(s_{t,i}=1|s_{j<t},a)=\alpha_{t,i}
$$

$$
z_t = \sum_is_{t,i}a_i
$$

其中，$z_t$表示经过attention机制后融合图像特征，作为LSTM的输入。

我们要优化的是$log\ p(y|a)$，想让它尽可能的大，那可以通过构造一个和$s$有关的下界$L_s$，最大化$L_s$即可最大化$log\ p(y|a)$，
$$
L_s = \Sigma_sp(s|a)log\ p(y|s,a)<=log\Sigma_sp(s|a)p(y|s,a)=log\ p(y|a)
$$
这样反传的时候，求导数就有

![](https://ww3.sinaimg.cn/large/006tNc79ly1fdl6m491kuj30j006iq3i.jpg)

这里面s服从多元伯努利分布，
$$
s_t \sim Multinoulli_L(\{\alpha_i\})
$$
因此前面的导数公式可以用N次蒙特卡洛采样近似，即

![](https://ww2.sinaimg.cn/large/006tNc79ly1fdl6qs71mwj30ik060dgf.jpg)

最后，引入了一个平均阈值项b和多元伯努利分布的熵项H，来减小上面梯度计算的方差，

![](https://ww2.sinaimg.cn/large/006tNbRwly1fearlia3acj30pq040wet.jpg)

最终的公式为

![](https://ww2.sinaimg.cn/large/006tNc79ly1fdl6xtphfcj30km05qq3q.jpg)

$\lambda_r$和$\lambda_e$分别是需要交叉验证来确定的参数。（此处黑人问号，待参考：Multiple object recognition with visual attention）

## Deterministic “Soft” Attention

所谓soft，就是不引入上面的s来决定哪个图像特征需要被关注，而是每个图像特征都按照原本的被关注的概率进行期望求和，得到我们需要的输入Z。
$$
z_t = \sum_i^L\alpha_{t,i}a_i
$$
值得注意的是，$\sum_i\alpha_{ti}=1$是不会变的，因为我们是通过softmax得到的$\alpha$。我们再令$\sum_t\alpha_{ti}\approx1$，这是为了鼓励模型在整个句子生成的期间内，虽然每个特征被注意的时刻不同，但是每个特征总有一些时刻被着重注意了，也就是每个特征都有用，不能从头到尾都和生成的词无关。

又引入了一个scalar $\beta$，使$z_t =\beta \sum_i^L\alpha_{t,i}a_i$，$\beta_t=\sigma(f_\beta(h_{t-1}))$，没什么理由，只是单纯实验发现这样做能让注意力集中在图像中的目标上。

最后，最小化下面的带惩罚项的损失函数。

![](https://ww2.sinaimg.cn/large/006tNc79gy1fdl870l391j30h602wq35.jpg)

## Results

![](https://ww4.sinaimg.cn/large/006tNbRwly1fear7o1a7uj31c60lkn4k.jpg)

# Knowing When to Look: Adaptive Attention via A Visual Sentinel for Image Captioning

这篇文章是上一篇的改进，出发点在于不需要每个词的生成都对应一个attetion，一些介词和连词并不需要图像中的信息，因此，本文提出attention的时间和空间的概念。

![](https://ww2.sinaimg.cn/large/006tNc79ly1fdlpl501wbj30so0cuab1.jpg)

原始的模型如上图（a），这个模型就是刚刚那篇论文的模型。本文对其首先进行如图（b）的改进，使得每次的attention根据当前的h而不是上一时刻的h来生成。其中，$x_t$是$y_{t-1}$的词向量。

![](https://ww3.sinaimg.cn/large/006tNc79ly1fdlpncja9kj30om0e4abg.jpg)

其次，引入了visual sentinel视觉哨兵，什么意思呢？

原始的经过CNN提取到的特征是$v_1....v_L$，每次计算attention权重$\alpha$的时候，我们用下面的公式
$$
z_t = w_h^Ttanh(W_vV+W_gh_tI^T)\\\alpha_t = softmax(z_t)
$$

其中，V是d\*k维，表示k个图像特征；$h_t$是第t时刻的decoder的隐状态输出，是d\*1维，$w_h$是k\*1维的参数，两个W是k\*d维的参数。

当引入视觉哨兵时，看上面的图，除了L个v向量，多了一个s向量，这个s的定义是

![](https://ww3.sinaimg.cn/large/006tNc79ly1fdm4wfurm8j30ga03q74l.jpg)

可以看出，g是一个门系数，他由上一时刻的隐状态和当前的输入（前一个单词）决定，这个g控制当前的记忆m，记忆就是LSTM的记忆门输出，他包含了截止到目前的语义信息和图像信息。 上面的两个公式可以理解为：根据前一时刻的状态和输入，对当前的记忆进行加工（点乘），加工后的向量称作哨兵信息。

这个哨兵信息作为和CNN的图像特征v一起，进行attention计算，如果attention的重点在s哨兵，说明当前不需要关注图像信息，当前的输出单词可能是介词、连词之类的词。所以，新的attention系数公式如下。

![](https://ww4.sinaimg.cn/large/006tNc79ly1fdm59gq97lj30nm02uq3b.jpg)

![](https://ww4.sinaimg.cn/large/006tNc79ly1fdmgtyl5b4j30bc02mwek.jpg)

![](https://ww3.sinaimg.cn/large/006tNc79ly1fdm5f6tytxj30d802cmx7.jpg)

从而得到新的context信息。

## Results

![](https://ww2.sinaimg.cn/large/006tNbRwly1feasdshgrrj31gk0ec79s.jpg)

# 总结

这两篇论文都是对attention进行研究和改进，本质上还是没有解决exposure bias问题，而且原始的seq2seq用每一步生成单词的概率进行连乘，作为整个序列的概率，这一点也可以改进，下一文会介绍如何用强化学习来解决这两个问题。