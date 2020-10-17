---
title: ImageCaption从入门到放弃之三_ 强化学习
tags:
  - caption
  - RL
categories:
  - Caption
date: 2017-03-16 00:22:51
mathjax: true
---

因为传统的seq2seq模型存在exposure bias（训练的时候，不论前一时刻的生成词是什么，总把期望的标签词作为当前时刻的输入；而测试时，使用前一时刻的生成词作为当前时刻的输入，这种现象就是exposure bias），而且优化的也不是整个序列的出现概率，而是每个词出现的条件概率，参见：[ImageCaption从入门到放弃之二_attention机制](https://liangguirong5.github.io/2017/03/12/ImageCaption从入门到放弃之二_attention机制/)

因此第一篇文章提出用强化学习直接优化最终的生成序列的metric分数，避免了上面的两个问题；第二篇文章是对第一篇的改进，目前在MSCOCO上的caption评测分数最高。

<!-- more -->

# Sequence Level Training with Recurrent Neural Networks

XENT：最小化交叉熵的序列生成，上面两个问题都存在，尤其是exposure bias。模型结构见下图，就是最原始的带attetion的seq2seq模型。

![](https://ww2.sinaimg.cn/large/006tNc79ly1fdp1tcrau7j30vc0lcq5o.jpg)

DAD：随机选择下一时刻的输入是标签or上一时刻的输出，是一种退火的思路，刚开始用样本的标签来训练这个模型，随着训练的深入，逐渐减小这个用标签训练的概率，直至最后模型完全不需要标签样本。存在的问题是，反传的时候不知道当前的输出是怎么产生的，这样有很多不能解释的地方。

![](https://ww3.sinaimg.cn/large/006tNc79ly1fdp1totgqoj30wg09ujsm.jpg)

E2E：对beam search的改进，直接取输出层前k个最大的词，重新归一化他们的权重，然后进行加权求和，这个得到的融合词作为下一时刻的输入。实际操作时，还是退火方法训练。

![](https://ww2.sinaimg.cn/large/006tNc79ly1fdp1usfepuj30y20ao3zt.jpg)

MIXER：这是本文提出的方法，mix XENT and RL的简写，强化学习的模型agent是decoder，状态s是当前的输入$x_t$和$h_{t-1}$，policy则是RNN内部的各个参数，action是每次预测出来的词，reward是计算出来的metric分值。因为最后用一整串词来计算metric分数，例如BLEU值，因此需要用policy gradient来进行模型的学习，损失函数如下：
$$
L_\theta=-\sum_{w_1^g,...,w_T^g}p_\theta(w_1^g,...,w_T^g)r(w_1^g,...,w_T^g)=-E_{[w_1^g,...,w_T^g]\sim p(\theta)}r(w_1^g,...,w_T^g)
$$
也就是生成序列长度为T，他的概率乘上对应的回报需要最大化，对应于上面的损失函数最小化。然后涉及到对这个函数进行求导，参考前人的工作得到
$$
\frac{\partial L_\theta}{\partial \theta}=\sum_t\frac{\partial L_\theta}{\partial o_t}\frac{\partial o_t}{\partial \theta}
$$
其中，$o_t$是softmax层的输入，然后有
$$
\frac{\partial L_\theta}{\partial o_t}=(r(w_1^g,...,w_T^g)-\widetilde{r}_{t+1})(p_{\theta}(w_{t+1}|w_t^g,h(t+1),c_t)-1(w_{t+1}^g))
$$

其中，$\widetilde{r}_{t+1}$是一个baseline，表示一个标准的回报值。如果我们生成的序列reward高，那第一个括号项就是正数，否则为负数，从而在这个式子中决定梯度的方向。第二个括号项就是梯度方向，是输出与标签的差（我们把t+1时刻的输出作为伪标签），之所以这么写，可以回想逻辑回归里面的梯度计算，误差函数求导之后就是这两项的差。我们这种计算梯度的方式，就是强化学习中的 policy gradient，如果不理解这个公式请看下面这一段，懂得可以跳过：

我们不知道生成序列的每一步的reward，但是我们知道最终这个序列的reward（BLEU、ROUGE等分数），这样如果最终reward高（对应上式第一个括号项大于零），那么我们的伪标签就是positive的，我们希望产生这样的结果，伪标签也就一定程度的变“真”了，这时直接沿着梯度下降去更新参数即可，且更新的幅度和reward大小成正比；如果最终reward很不理想，导致第一个括号项为负，那么再进行梯度下降去优化，就会得到相反的效果（max损失函数），使得输出离伪标签更远（这时伪标签是negative的），达到我们的本意。

在实际中，我们的字典很大，意味着action space很大，直接进行RL训练很难进行，因此作者提出预训练+退火：

前$N^{XENT}​$个epochs用ground truth进行训练，损失函数采用交叉熵；然后$N^{XE+R}​$个epochs中，前$T-\Delta​$步进行XENT训练，后$\Delta​$步进行Reinforce训练。算法和结构框图如下：

```
Data: a set of sequences with their corresponding context.
Result: RNN optimized for generation.
Initialize RNN at random and set N_XENT,N_XE+R and Delta;
for s=T,1,-Delta do 
	if s==T then
		train RNN for N_XENT epochs using XENT only;
	else
		train RNN for N_XE+R epochs. Use XENT loss in the first s 			steps, and RL in the remaining T-s steps.
	end
end
```

![](https://ww1.sinaimg.cn/large/006tNc79ly1fdp1v5gpb4j30z009egmz.jpg)

# Self-critical Sequence Training for Image Captioning

细心阅读会发现，上文提到的$\widetilde{r}_{t+1}$并没有说明是怎么确定的，在该论文中只是说他们用RNN的隐状态作为一个线性回归模型输入，去预测未来的reward，理论依据不足。因此，本篇论文就这一点完善了上文的工作。

本文提出了SCST(Self-critical Sequence Training )训练方式，利用当前模型去跑一遍解码过程，得到的评测分数作为baseline，参与强化学习训练中，下面这个图更有助于理解：

![](https://ww4.sinaimg.cn/large/006tNc79ly1fdtd1qyovcj312y0jcadn.jpg)

效果图

![](https://ww4.sinaimg.cn/large/006tNc79ly1fdtepb6turj30qa0agabz.jpg)

# 总结

SCST是目前表现最好的模型，也是ImageCaption从入门到放弃的最后一期。思考了几个问题，SCST每一次的标准reward是由相似的模型解码得到的，是不是可以用GAN来进行改进呢？此外，能不能设计一个模型，生成每一步的reward呢？这样不仅能加快训练，效果肯定也有提升。最后，关于DQN和PG的优劣，可能会单独发一篇博文来好好整理一下。
