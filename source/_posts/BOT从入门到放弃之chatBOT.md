---
title: BOT从入门到放弃之chatBOT
tags:
  - bot
mathjax: true
categories:
  - bot
date: 2017-07-18 09:47:39
---

chatbot一般都是非任务型、数据驱动、闲聊式，第一篇文章用强化学习解决无限环，2、3两文在encoder和decoder之间维护了另一个RNN，3相较于2还引入了一个隐变量z，思想是来自VAE。4则是使用了他们组之前的一个工作：前后向seq2seq模型。

<!-- more -->

# 1. Deep reinforcement learning for dialogue generation

Li, J., & Monroe, W. (2016). Deep Reinforcement Learning for Dialogue Generation, 1–11.

传统的seq2seq倾向于回复generic response，也就是无意义的安全回答，li 2015提出可以最大化互信息(MMI)来解决这一问题，但是仍不能解决多轮对话中的infinite loop问题，如下图

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhmv0sauq5j31ds0nik0g.jpg)

为此，作者提出了加入RL的seq2seq模型，状态是前两轮的对话pi和qi，动作是本轮产生的回答，policy就是seq2seq网络，reward有三部分：

- 易于回答：不想让bot成为话题终结者，因此人工构造了枯燥回复集S，定义

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhmv6tfy4kj30i004gglu.jpg)

- 信息流：不想产生无限环，因此要让同一个agent接连两次的回复尽可能不相似， 不能总是同一句话反复说。

  ![](https://ws3.sinaimg.cn/large/006tKfTcly1fhmv8y83sej30is03st8z.jpg)

- 语义连贯：当前agent的回复应该和另一agent的提问紧密相关，这样不仅能做到所答即所问，还能避免产生I don't know等安全回复。做法是计算互信息，作为reward

  ![](https://ws1.sinaimg.cn/large/006tKfTcly1fhmvc49kwij30ne02qaah.jpg)

最终的reward是

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhmvdfipevj30ig02cweo.jpg)

## simulation

本文作者训练了两个agent，进行你问我答的对话生成，

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhn19kmxigj31j80nyaea.jpg)

首先用有监督的方法预训练模型，借鉴[sequence level training with recurrent neural network](https://liangguirong5.github.io/2017/03/16/ImageCaption从入门到放弃之三-强化学习/)那篇文章的思想，把reward设置为互信息的大小，然后退火的方式训练模型，最终梯度的更新公式如下

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhmzkck9crj30lg02ymxi.jpg)

PS: 我觉得这里等号右边缺少了求期望操作，或者等号变成约等号。

前面是模型预训练，接下来是用之前构造的reward进行强化学习训练，梯度更新公式如下

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhmzru8mbej30pi04udgc.jpg)

# 2. Building End-To-End Dialogue Systems Using Generative Hierarchical Neural Network Models

Iulian V Serban, A. S. Y. B. A. C. J. P. (2016). Building End-To-End Dialogue Systems Using Generative Hierarchical Neural Network Models (pp. 1–8).

层叠式的seq2seq模型结构如下图，主要是在encoder和decoder之间维护了一个独立的RNN，这个RNN用于连接输入输出两个端，且在一次对话的多个轮次中不断更新状态，从而一定程度的解决了多轮对话中不连贯的问题。

![](https://ws4.sinaimg.cn/large/006tKfTcly1fhnzcd6p83j30u10fi0un.jpg)

# 3. A Hierarchical Latent Variable Encoder-Decoder Model for Generating Dialogues

Serban, I. V., Sordoni, A., Lowe, R., Charlin, L., Pineau, J., Courville, A., & Bengio, Y. (2016, May 20). A Hierarchical Latent Variable Encoder-Decoder Model for Generating Dialogues. *arXiv.org*.

在上图的基础上引入了隐变量z，模型结构如下。

![](https://ws4.sinaimg.cn/large/006tKfTcly1fho0oianddj30sx0fjmyt.jpg)

引入隐变量的动机在于，之前的模型虽然兼顾了多轮对话中的信息，但是很难产生有意义的回复，本文作者分析可能的原因是模型只有底层的随机变量来决定最终的输出，这样没有办法兼顾全局，而在更高层次引入隐变量z，则可以一定程度的改善这一问题。此外，从计算的角度看，如果用隐状态来包含所有的历史信息，那么由于梯度消失，这种信息必然会丢失一部分，可以加入隐变量z来加强对历史信息的建模，从而达到更好的回复效果。

#3.x TOWARDS AN AUTOMATIC TURING TEST: LEARNING TO EVALUATE DIALOGUE RESPONSES

![](https://ws1.sinaimg.cn/large/006tNc79ly1fjel9x4nf8j30t208sta1.jpg)



​	

# 4.Sequence to Backward and Forward Sequences: A Content-Introducing Approach to Generative Short-Text Conversation 

Mou, L., Song, Y., Yan, R., Jin, Z., Zhang, L., & Li, G. (2016). Sequence to Backward and Forward Sequences: A Content-Introducing Approach to Generative Short-Text Conversation (pp. 1–10). Presented at the COLING.

本文提出，先用PMI方法从query中抽取一个关键词，本文特指名词，然后利用这个关键词和seq2BF模型，去生成回复。seq2BF是指从句子中间的某个位置向两侧生成，最终得到完整的句子，而不是seq2seq那种从初始到末尾的生成。如下图所示

![](https://ws2.sinaimg.cn/large/006tNc79ly1fhoyq6bnjsj31000o6gow.jpg)

这样的好处在于，一方面能针对query中的重点信息生成回复，避免了无意义回答，同时seq式的语言生成模块能保证语句的流畅可理解。

#4. DocChat

是基于检索式的闲聊型对话，文章中主要是通过构造一系列的人工特征，然后结合输入语句，从候选答案中进行匹配，从而实现对闲聊的合理回复。

#5. Conversational Contextual Cues: The Case of Personalization and History for Response Ranking

Al-Rfou, R. (2016). Conversational Contextual Cues: The Case of Personalization and History for Response Ranking, 1–10.

网络结构如下图所示，并不是生成式的，而是answer selection的方法，对Reddit 上1.33亿个conversation进行模型训练，输入的几项分别是：

input: dialog[i], context: dialog[0:i-1], author: 该评论的作者

正样本：response=dialog[i+1]

负样本：response=随机其他comment

![](https://ws2.sinaimg.cn/large/006tKfTcly1fhnxgdo4vuj31he0lygpf.jpg)

另外，文章还提出了multi-loss的方法，旨在度量以上网络中每个feature的有效性。分别对组合{response, input}, {response, context}, {response, author}求一个hidden layer, 将这3个hiddenlayer{h1, h2, h3} concatenate成一个总hidden layer h4, 将{h1, h2, h3, h4}分别得到的分类loss加和作为网络总loss进行优化，但最后prediction的时候只采用h4的输出结果。

# 

# 6. Neural Response Generation via GAN with an Approximate Embedding Layer

本文提出了用GAN来进行对话生成，其主要创新点在于提出了AEL层，为的是解决蒙特卡洛采样操作导致的梯度无法从Discriminator传到Generator中的问题。整个模型结构如下图所示，右侧是AEL层的结构。

![](https://ws2.sinaimg.cn/large/006tNc79ly1fjel1aw53fj31d60pm7ag.jpg)

