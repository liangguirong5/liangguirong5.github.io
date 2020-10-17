---
title: ImageCaption从入门到放弃之一_任务综述
tags:
  - caption
categories: 
  - Caption
mathjax: true
date: 2017-03-1 20:29:52
---

# 引言

本文转载自paperweekly：

第二十二期的PaperWeekly对Image Captioning进行了综述。今天这篇文章中，我们会介绍一些近期的工作。（如果你对Image Captioning这个任务不熟悉的话，请移步二十二期[PaperWeekly 第二十二期---Image Caption任务综述](http://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247484014&idx=1&sn=4a053986f5dc8abb45097fed169465fa&chksm=96e9ddeea19e54f83b717d63029a12715c238de8d6af261fa64af2d9b949480e685b8c283dda&scene=21#wechat_redirect)）

<!-- more -->

Image Captioning的模型一般是encoder-decoder的模型。模型对$p(S|I)$进行建模，$S$是描述，$I$是图片。模型的训练目标是最大化log似然：$\max_\theta\sum_i \log P(S_i|I_i, \theta)$。

然而使用最大似然训练有两个问题：

1、虽然训练时最大化后验概率，但是在评估时使用的测度则为BLEU，METEOR，ROUGE，CIDER等。这里有训练loss和评估方法不统一的问题。而且log似然可以认为对每个单词都给予一样的权重，然而实际上有些单词可能更重要一些（比如说一些表示内容的单词）。

2、第二个问题为Exposure bias。训练的时候，每个时刻的输入都是来自于真实的caption。而生成的时候，每个时刻的输入来自于前一时刻的输出；所以一旦有一个单词生成的不好，错误可能会接着传递，使得生成的越来越糟糕。

如何解决这两个问题呢？很显而易见的想法就是尽量使得训练和评估时的情形一样。我们可以在训练的时候不优化log似然，而是直接最大化CIDER（或者BLEU，METEOR，ROUGE等）。并且，在训练时也和测试时一样使用前一时刻的输入，而不是全使用ground truth输入。

然而这有什么难点呢？第一，CIDER或者这一些metric并不是可直接求导。（这就是为什么在分类问题中，我们把0-1 error近似成log loss，hinge loss的原因）。其次从前一时刻输出获得后一时刻的输入涉及到采样操作，这也是不可微的。为了能够解决这些不可微的问题，人们就想到了Reinforcement learning。

# **RL基本概念**

RL中有一些比较重要的基本概念：状态（state），行为（action），回报（reward）和决策（policy）。决策是一个状态到动作的函数，一般是需要学习的东西。拿打游戏的例子介绍RL最简单。如果说是玩flappy bird，RL要学习的就是在什么位置跳，能使得最后得到的分数越高。在这个例子里，最后的分数就是回报，位置就是状态，跳或者不跳就是行为，而什么时候跳就是学到的策略。

如果放在Image captioning中，状态就是你看到的图片和已生成的单词，而动作就是下一个单词生成什么，回报就是CIDER等metric。

## **相关文献**

最近已经有很多工作将RL用在NLP相关的问题上。[1]第一次将REINFORCE算法用在image caption和seq2seq问题上。[5]将使用了更先进的RL算法 — Actor-critic — 来做machine translation上。[2,4]将[1]的算法进行稍许改进（仍旧是REINFORCE算法），使用在了image captioning上。[3]将REINFORCE用在序列生成GAN中，解决了之前序列生成器输出为离散不可微的问题。[6]将RL用在自然对话系统中。这篇文章中我们主要介绍[1,2,4]。

## **RL算法背景**

这三篇文章使用的是REINFORCE算法，属于增强学习中Policy Gradient的一种。我们需要将deterministic的策略形式 $a=\pi(s,\theta)$转化为概率形式，$p(a) = \pi(a|s, \theta)$。Policy Gradient就是对参数$\theta$求梯度的方法。

直观的想，如果我们希望最后的决策能获得更高的reward，最简单的就是使得高reward的行为有高概率，低reward的行为有低概率。所以REINFORCE的更新目标为

$$\max_{\theta} \sum R(a,s)\log \pi(a|s, \theta)$$

$R(s,a)$是回报函数。有了目标，我们可以通过随机梯度下降来更新$\theta$来获得更大的回报。

然而这个方法有一个问题，训练时梯度的方差过大，导致训练不稳定。我们可以思考一下，如果reward的值为100到120之间，现在的方法虽然能更大地提高reward为120的行为的概率，但是也还是会提升低reward的行为的概率。所以为了克服这个问题，又有了REINFORCE with baseline。

$$\max_{\theta} \sum (R(a,s) - b(s))\log \pi(a|s, \theta)$$

$b(s)$在这里就是baseline，目的是通过给回报一个基准来减少方差。假设还是100到120的回报，我们将baseline设为110，那么只有100回报的行为就会被降低概率，而120回报的行为则会被提升概率。

# **三篇paper**

第一篇是FAIR在ICLR2016发表的[1]。这篇文章是第一个将RL的算法应用的离散序列生成的文章。文章中介绍了三种不同的方法，这里我们只看最后一种算法，Mixed Incremental Cross-Entropy Reinforce。

大体的想法就是用REINFORCE with baseline来希望直接优化BLEU4分数。具体训练的时候，他们先用最大似然方法做预训练，然后用REINFORCE finetune。在REINFORCE阶段，生成器不再使用任何ground truth信息，而是直接从RNN模型随机采样，最后获得采样的序列的BLEU4的分数r作为reward来更新整个序列生成器。

这里他们使用baseline在每个时刻是不同的；是每个RNN隐变量的一个线性函数。这个线性函数也会在训练中更新。他们的系统最后能比一般的的cross extropy loss，和scheduled sampling等方法获得更好的结果。

他们在github开源了基于torch的代码，https://github.com/facebookresearch/MIXER

第二篇论文是今年CVPR的投稿。这篇文章在[1]的基础上改变了baseline的选取。他们并没有使用任何函数来对baseline进行建模，而是使用了greedy decoding的结果的回报作为baseline。他们声称这个baseline减小了梯度的variance。

这个baseline理解起来也很简单：如果采样得到句子没有greedy decoding的结果好，那么降低这句话的概率，如果比greedy decoding还要好，则提高它的概率。

这个方法的好处在于避免了训练一个模型，并且这个baseline也极易获得。有一个很有意思的现象是，一旦使用了这样的训练方法，beam search和greedy decoding的结果就几乎一致了。

目前这篇文章的结果是COCO排行榜上第一名。他们使用CIDEr作为优化的reward，并且发现优化CIDEr能够使所有其他metric如BLEU，ROUGE，METEOR都能提高。

他们的附录中有一些captioning的结果。他们发现他们的模型在一些非寻常的图片上表现很好，比如说有一张手心里捧着一个长劲鹿的图。

第三篇论文[4]也是这次CVPR的投稿。这篇文章则是在$R(a,s)$这一项动了手脚。

前两篇都有一个共同特点，对所有时刻的单词，他们的$R(a,s)$都是一样的。然而这篇文章则给每个时刻的提供了不同的回报。

其实这个动机很好理解。比如说，定冠词a，无论生成的句子质量如何，都很容易在句首出现。假设说在一次采样中，a在句首，且最后的获得回报减去baseline后为负，这时候a的概率也会因此被调低，但是实际上大多数情况a对最后结果的好坏并没有影响。所以这篇文章采用了在每个时刻用$Q(w_{1:t})$来代替了原来一样的$R$。

这个$Q$的定义为，

$Q\theta(w{1:t}) = \mathbb{E}{w{t+1:T}}[R(w{1:t}, w{t+1:T})]$

也就是说，当前时刻的回报，为固定了前t个单词的期望回报。考虑a的例子，由于a作为句首生成的结果有好有坏，最后的Q值可能接近于baseline，所以a的概率也就不会被很大地更新。实际使用中，这个Q值可以通过rollout来估计：固定前t个词后，随机采样K个序列，取他们的平均回报作为Q值。文中K为3。这篇文章中的baseline则跟[1]中类似。

从实验结果上，第三篇并没有第二篇好，但是很大一部分原因是因为使用的模型和特征都比较老旧。

# **总结**

将RL用在序列生成上似乎是现在新的潮流。但是现在使用的大多数的RL方法还比较简单，比如本文中的REINFORCE算法可追溯到上个世纪。RL本身也是一个很火热的领域，所以可以预计会有更多的论文将二者有机地结合。

# **参考文献**

[1] Ranzato, Marc’Aurelio, Sumit Chopra, Michael Auli, and Wojciech Zaremba. “Sequence level training with recurrent neural networks.” arXiv preprint arXiv:1511.06732 (2015).

[2] Rennie, Steven J., Etienne Marcheret, Youssef Mroueh, Jarret Ross, and Vaibhava Goel. “Self-critical Sequence Training for Image Captioning.” arXiv preprint arXiv:1612.00563 (2016).

[3] Yu, Lantao, Weinan Zhang, Jun Wang, and Yong Yu. “Seqgan: sequence generative adversarial nets with policy gradient.” arXiv preprint arXiv:1609.05473 (2016).

[4] Liu, Siqi, Zhenhai Zhu, Ning Ye, Sergio Guadarrama, and Kevin Murphy. “Optimization of image description metrics using policy gradient methods.” arXiv preprint arXiv:1612.00370 (2016).

[5] Bahdanau, Dzmitry, Philemon Brakel, Kelvin Xu, Anirudh Goyal, Ryan Lowe, Joelle Pineau, Aaron Courville, and Yoshua Bengio. “An actor-critic algorithm for sequence prediction.” arXiv preprint arXiv:1607.07086 (2016).

[6] Li, Jiwei, Will Monroe, Alan Ritter, Michel Galley, Jianfeng Gao, and Dan Jurafsky. “Deep reinforcement learning for dialogue generation.” arXiv preprint arXiv:1606.01541 (2016).