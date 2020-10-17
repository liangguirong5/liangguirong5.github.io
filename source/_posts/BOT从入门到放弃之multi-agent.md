---
title: BOT从入门到放弃之multi-agent
tags:
  - bot
mathjax: true
categories:
  - bot
date: 2017-08-02 15:19:58
---

通过多个agent之间的交互来模拟自然语言生成等语言任务。

<!-- more -->

# 1. Deal or No Deal? End-to-End Learning for Negotiation Dialogue

 Lewis, M., Yarats, D., Dauphin, Y. N., Parikh, D., & Batra, D. (2017). (pp. 1–11). Presented at the EMNLP.

本文提出了一个全新的任务：两个agent之间进行协商，协商目标是最大化自身的收益，协商的内容是指定好的一系列物品和对应的价值，具体如下图

¨![](https://ws2.sinaimg.cn/large/006tKfTcly1fi6wij47etj31kw0sods5.jpg)

对于同样的物品不同的agent有不同的价值，文章设置了三种物品，每个agent的goal用六个数字表示，六个数字就是三个物品的个数和对应的价值，然后用一个GRU对这个向量进行编码，从而作为一个特征向量参与到之后的序列生成中，如下图

![](https://ws1.sinaimg.cn/large/006tKfTcly1fi6x3aznhvj30ro0aotaj.jpg)

其中input encoder就是生成句子的部分，而output decoder则是在对话结束后生成最终的决策。决策就是最后每个agent都得到几个物品，采用枚举法对所有的可能求概率，概率最大的方案就是最终的决策。

此外，本文由提出了加入强化学习的goal-based training，用最终的协商结果最为reward来优化agent网络的参数，

# 2. Multi-agent cooperation and the emergence of (natural) language

Lazaridou, A., Peysakhovich, A., & Baroni, M. (2017).  (pp. 1–11). Presented at the ICLR.

本文是facebook和deepmind的工作。通过训练两个agent，一个作为sender，对两个图像中被标记的一个生成单个symbol的描述，然后另一个作为receiver的输入是两个图像以及sender的描述，输出sender描述的是哪一张图像，结果为真即认为两个agent学到了他们能交流的语言。图像不是直接输入，而是先送到预训练好的VGG网络，用VGG的softmax层或者倒数第二个全连接层的输出作为图像的表示。模型结构很简单，如下图所示。

![](https://ws3.sinaimg.cn/large/006tNc79ly1fi5fao593xj31js0psqa8.jpg)

作者构造了两种sender，第一种是全连接网络对图像进行表示，第二种额外加了卷积层。模型的训练只有最终成败作为监督信号进行参数更新。最终的结果如下图所示。

![](https://ws1.sinaimg.cn/large/006tNc79ly1fi5ffsghvaj30tw0ne0wr.jpg)

![](https://ws4.sinaimg.cn/large/006tNc79ly1fi5fl6dpdfj316u0f4n15.jpg)

上表的纯度，是根据生成symbol对图片聚类，而每个图片有真实的标签，表示该图片的大类，因此能计算类内纯度作为评价指标。为了更好的观察sender给出的描述在语义上的效果，作者把相同的object的fc向量表示进行average，得到每个object的向量表示，然后用t-SNE可视化，同样的symbol用相同的颜色表示，结果如下图。

![](https://ws2.sinaimg.cn/large/006tNc79ly1fi5fxi1ddtj30uq0uq0wt.jpg)

可以看出，距离相近的大多颜色相同，但是效果仍不明显。

为了验证我们学到的是两张图是狗与猫的区别，而不是一些其他因素，我们对sender输入吉娃娃和猫，对receiver输入泰迪和猫，希望sender描述吉娃娃后，receiver能成功选中泰迪。基于这样的思想，构建大量此类的image pair，再次训练，结果如下。

![](https://ws1.sinaimg.cn/large/006tNc79ly1fi5g281p81j31g40cg0w5.jpg)

![](https://ws3.sinaimg.cn/large/006tNc79ly1fi5g18qnafj30vs0uqn1j.jpg)

可以看到纯度略有提升，而且可视化结果更加明显了，这表明两个agent的确学到了对于图片的语义描述。

最后，这种agent之间的沟通还不能被人类理解，因此要对描述的生成加入自然语言的引导，也就是一边训练agent之间的交流，一边对sender的embedding层进行外加softmax训练，使其能针对图片选择正确的单词，而不是agent才能理解的symbols。这两种训练方式等概率进行切换，同时两种训练方式共享embedding层，最终能达到一定程度的人类可理解的、agent之间可交流的效果。

# 3. Curiosity-driven Exploration by Self-supervised Prediction 

 Pathak, D., Agrawal, P., Efros, A. A., & Darrell, T. (2017). (pp. 1–12). Presented at the ICML.

在强化学习中，尤其是agent完成游戏任务，reward不是实时获得的，一般最终游戏胜利或者失败才会产生一个reward，本文提出了基于好奇心的reward机制，当模型去探索一些新鲜的状态，这就可以产生一个reward去进行鼓励，判断状态是否‘新鲜’可以通过估计当前状态和动作会产生什么状态，和实际产生了什么状态，估计与实际的差越大说明模型对于这个新状态掌握的越少，需要进行激励，鼓励模型对该状态进行探索，也就是好奇心。同时这个新的状态应该是对action相关的，例如模型不需要对树叶不同时刻随风摆动的新状态进行探索，因为这个action无关，实现方案如下图。

![](https://ws2.sinaimg.cn/large/006tKfTcly1fi6xmq752fj31fu0kutby.jpg)

通过学习一个估计的a去逼近真实的a，这样能学到状态的高层表达，这个高层表达理想上是和a直接相关的，解决了树叶问题。然后再通过估计的s与真实的s的高层表达做差，得到了内部reward。内部reward和外部reward相结合就是模型的reward，从而指导参数的更新。

最终的结果显示，单单是用内部reward就能让模型在游戏中取得很好的表现，充分表明了这种自监督的学习方式的有效性。

# 4. A Roadmap towards Machine Intelligence

Mikolov, T., Joulin, A., & Baroni, M. (2016). 