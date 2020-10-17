---
title: BOT从入门到放弃之阶段性综述
tags:
  - bot
mathjax: true
categories:
  - bot
date: 2017-09-11 10:19:58
---

对话系统一般分为任务型和闲聊式两种，其中任务型对话，它主要针对某些指定的场景，例如餐馆推荐、机票预订等服务场景，通过和用户的交互，完成指定的任务，因此称作任务型对话，也叫封闭域对话。闲聊型的对话主要是指BOT能够针对用户的输入，给出一个合理的回复，从而实现基础的聊天功能，又称作开放域对话，因为用户此时的输入是无特定场景的。实际的系统则常常是二者兼具的，因为商用对话系统既要能作为个人助理完成用户的指定需求，又要能对用户的闲聊做出合理的回应。这样就有一个问题，如何判断此时用户的意图是闲聊还是任务呢？文章[1]提出把用户的意图（任务或闲聊）的划分看做一个二分类问题，然后可以用SVM或者CNN作为分类器，通过词向量和一些人工特征作为模型的输入，最终可以得到用户的意图的概率输出，从而判断此时对话系统的工作状态应该是闲聊型还是任务型。

<!-- more -->

本文会按照任务型和闲聊型两种对话类型进行分别介绍，还有一些既不是任务型也不是闲聊型的研究工作，这些工作主要在通过对话任务来研究语言的产生和语义的解释性问题，这些工作会作为本文对对话系统研究综述的一部分在最后进行介绍。

## 1. 任务型对话

常见的任务型对话系统一般包括语音识别、自然语言理解、状态跟踪与决策、自然语言生成、语音合成这五大部分，本文主要针对对话系统在自然语言层面上的研究进展进行介绍，而不涉及语音识别与语音合成。

![](https://ws4.sinaimg.cn/large/006tNc79ly1fjeth24o1kj30kw09u3ze.jpg)

###1.1 自然语言理解

自然语言理解主要是指对用户的输入进行建模，挖掘用户意图，一般是事先设定好几种用户的意图，然后转换为一个对用户输入进行多分类的问题[1]，或者是对用户的输入进行词性标注或命名体识别等序列标注方法得到结构化数据，从而得到输入语句的结构表示[2]。随着深度神经网络研究的不断深入，用神经网络对输入源语句进行向量表示[3-6]作为语言理解模块的方法已经逐渐成为当下的研究热点，一般是用循环神经网络RNN对输入语句进行迭代[3,4,5]，从而得到向量表示，而[6]是采用普通的DNN和CNN，如下图所示。

![](https://ws2.sinaimg.cn/large/006tNc79ly1fjelanivc6j31040qq0yj.jpg)

###1.2 对话状态跟踪与决策

对话状态跟踪是指对于当前的任务，已经确定的信息和未确定的信息都有哪些，从而让系统得出当前还需要向用户确定的信息，以及根据当前信息能提供给用户的候选结果。这个对话状态跟踪模块是任务型对话系统的核心，常见的是slot-value填槽方法，也就是预先确定本任务会有哪些信息slot需要确定，以及每个信息的取值范围value，在每一个自然语言理解模块后进行slot-value检测，确定当前这句话包含的信息，从而更新slot-value。传统的slot-value更新就是从用户句子中进行命名体识别，然后再通过匹配来更新状态[3]。最新的工作一般是利用输入语句的向量化表示，或是同所有slot-value对进行匹配，得到最可能的slot-value对[6]，或是同前一时刻的状态向量一同作为一个RNN网络的输入，更新状态向量[4,5]。下图是先对源语句进行CNN卷积特征提取，第一层卷积、第二层卷积和第三层之后的池化结果共同组成了Jordan-RNN的输入，进行状态更新。

![](https://ws3.sinaimg.cn/large/006tNc79ly1fi5ezq9i6zj315g0pe0ya.jpg)

除了slot-value方法，还有基于检索式的对话模型，避开了状态更新这个步骤，直接利用源语言和候选回复进行匹配，匹配最好的一句就直接作为回复进行输出[3]，如下图所示。

![](https://ws3.sinaimg.cn/large/006tNc79ly1fi5e7h3etsj319e0jowkc.jpg)

对话决策是指根据当前的对话状态，确定任务候选的答案，例如餐馆推荐任务就是根据当前的对话进度确定用户可能感兴趣的餐馆，即为候选答案。一般的做法都是根据当前确定的信息，直接在数据库或者知识库中进行查询，这样就能得到候选答案。

###1.3 自然语言生成

如果是上文的检索式，直接把检索结果作为输出即可。另外一种方法设定好系统的所有可能回复句式，然后根据前两个模块的输出来确定当前应该采取的回复句式，再根据当前状态和决策结果对句式进行填充，然后得到完整的生成句[2]。此外，还可以通过一个RNN网络进行自然语言生成，RNN的输入一般是之前模块提取到的输入表示、跟踪状态、决策结果三者的融合[4]，[5]在其基础上又引入了一个隐变量z，作为用户意图的表示，主要思想来自于变分自编码器VAE，模型结构如下图所示。模型结构图中没有体现的是强化学习部分，强化学习一般是在损失函数中引入回报值reward，这个回报一般是根据最后系统是否能完成任务来决定。

![](https://ws2.sinaimg.cn/large/006tNc79ly1fi5f0vtrjrj31i80ua46f.jpg)

## 2. 闲聊型对话

闲聊型对话一般是基于端到端的模型，用大量双句语料进行训练，同时一般还加入强化学习来辅助。[7]用强化学习解决无限环，[8]在encoder和decoder之间维护了另一个RNN，[9]则在[8]的基础上还引入了一个隐变量z，[10]则是使用了前向后向seq2seq模型，[11]把生成对抗网络GAN引入到模型中进行语言生成，[12]则是基于检索式的闲聊系统。

### 2.1 基于强化学习的无限环解决方案

传统的seq2seq倾向于回复generic response，也就是无意义的安全回答，[13]提出可以最大化互信息(MMI)来解决这一问题，但是仍不能解决多轮对话中的infinite loop问题，如下图

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

本文作者训练了两个agent，进行你问我答的对话生成，

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhn19kmxigj31j80nyaea.jpg)

首先用有监督的方法预训练模型，借鉴[14]中的训练思想，把reward设置为互信息的大小，然后退火的方式训练模型，最终梯度的更新公式如下

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhmzkck9crj30lg02ymxi.jpg)

前面是模型预训练，接下来是用之前构造的reward进行强化学习训练，梯度更新公式如下

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhmzru8mbej30pi04udgc.jpg)



### 2.2 基于层叠式网络结构的端到端模型 

层叠式[8]的seq2seq模型结构如下图，主要是在encoder和decoder之间维护了一个独立的RNN，这个RNN用于连接输入输出两个端，且在一次对话的多个轮次中不断更新状态，从而一定程度的解决了多轮对话中不连贯的问题。![](https://ws4.sinaimg.cn/large/006tKfTcly1fhnzcd6p83j30u10fi0un.jpg)

[9]则在上图的基础上引入了隐变量z，模型结构如下。![](https://ws4.sinaimg.cn/large/006tKfTcly1fho0oianddj30sx0fjmyt.jpg)

引入隐变量的动机在于，之前的模型虽然兼顾了多轮对话中的信息，但是很难产生有意义的回复，本文作者分析可能的原因是模型只有底层的随机变量来决定最终的输出，这样没有办法兼顾全局，而在更高层次引入隐变量z，则可以一定程度的改善这一问题。此外，从计算的角度看，如果用隐状态来包含所有的历史信息，那么由于梯度消失，这种信息必然会丢失一部分，可以加入隐变量z来加强对历史信息的建模，从而达到更好的回复效果。

[10]则是先用PMI方法从query中抽取一个关键词，本文特指名词，然后利用这个关键词和seq2BF模型，去生成回复。seq2BF是指从句子中间的某个位置向两侧生成，最终得到完整的句子，而不是seq2seq那种从初始到末尾的生成。如下图所示

![](https://ws2.sinaimg.cn/large/006tNc79ly1fhoyq6bnjsj31000o6gow.jpg)

这样的好处在于，一方面能针对query中的重点信息生成回复，避免了无意义回答，同时seq式的语言生成模块能保证语句的流畅可理解。

### 2.3 非Seq2Seq的闲聊式对话

[12]是基于检索式的闲聊型对话，文章中主要是通过构造一系列的人工特征，然后结合输入语句，从候选答案中进行匹配，从而实现对闲聊的合理回复。

[14]则是提出了用GAN来进行对话生成，其主要创新点在于提出了AEL层，为的是解决蒙特卡洛采样操作导致的梯度无法从Discriminator传到Generator中的问题。整个模型结构如下图所示，右侧是AEL层的结构。![](https://ws2.sinaimg.cn/large/006tNc79ly1fjel1aw53fj31d60pm7ag.jpg)

## 3. Multi-agent对话系统

前文介绍了最近任务型和闲聊型对话的研究进展和方向，当下还有一些研究工作是想通过对话来探索语言中的逻辑或者语言本身的产生机制。

[15]提出了一个全新的任务：两个agent之间进行协商，协商目标是最大化自身的收益，协商的内容是指定好的一系列物品和对应的价值，具体如下图

¨![](https://ws2.sinaimg.cn/large/006tKfTcly1fi6wij47etj31kw0sods5.jpg)

对于同样的物品不同的agent有不同的价值，文章设置了三种物品，每个agent的goal用六个数字表示，六个数字就是三个物品的个数和对应的价值，然后用一个GRU对这个向量进行编码，从而作为一个特征向量参与到之后的序列生成中，如下图

![](https://ws1.sinaimg.cn/large/006tKfTcly1fi6x3aznhvj30ro0aotaj.jpg)

其中input encoder就是生成句子的部分，而output decoder则是在对话结束后生成最终的决策。决策就是最后每个agent都得到几个物品，采用枚举法对所有的可能求概率，概率最大的方案就是最终的决策。此外，本文还提出了加入强化学习的goal-based training，用最终的协商结果最为reward来优化agent网络的参数。文章的结果证明了这种最优化目标的训练要比最大似然的训练方法更好，能让模型学习到推理的能力和语言表达能力。

[16]是facebook和deepmind的工作。通过训练两个agent，一个作为sender，对两个图像中被标记的一个生成单个symbol的描述，然后另一个作为receiver的输入是两个图像以及sender的描述，输出sender描述的是哪一张图像，结果为真即认为两个agent学到了他们能交流的语言。图像不是直接输入，而是先送到预训练好的VGG网络，用VGG的softmax层或者倒数第二个全连接层的输出作为图像的表示。模型结构很简单，如下图所示。

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

##4. 评价方法

常见的对话评价方法一般使用BLEU值来比对生成结果和标准答案之间的差距，但是这从本质上不能揭示对话的质量好坏。最近的研究[17]在尝试用神经网络模型对生成结果进行评价，但这也受限于训练评价网络的语料，不能真正的适应开放域的对话评价。其他的评价指标诸如对话轮数、信息熵[10]、对话成功率[2]、基于检索的评价指标[12]、多样性和相关性[11]等指标，虽然能对对话的某一方面进行评价，但都不能兼顾语言和语义两个层面，甚至有论文指出，对话是无法评价的。无论如何，一个合适的评价方法对于对话系统的研究是至关重要的，相信评价方法的研究仍会是对话系统中一项重要的工作，更好的评价方法会不断出现。

## 5. 总结

近些年对话系统的研究呈现出不断增长的态势，一方面是由于人工智能领域取得的长足发展让工业界和学术界都想挑战对话这个相对复杂的人工智能任务，另一方面也是由于对话系统的广大应用前景和应用需求的推动。无论是任务型还是闲聊型，当前的研究都没有深入到对话的本质，从对话本身的机制入手研究，对话的目的是交流，因此我们人类的每一句话都有自身的目的——信息的外传或内收。且人类的对话是可控的，我们能决定对话的进行方向，而只用双句语料进行seq2seq的训练，没有办法做到对未来对话的掌控。总之，对话系统是一种相对复杂的人工智能任务，当下的基础理论还不足以支撑让对话系统像人一样思考，但是相信随着技术的发展和进步，对话系统的研究也会不断取得长足的进步。

[1] Akasaki, S., & Kaji, N. (2017). Chat Detection in an Intelligent Assistant: Combining Task-oriented and Non-task-oriented Spoken Dialogue Systems (pp. 1–11). Presented at the ACL.

[2] Williams, J. D., Asadi, K., & Zweig, G. (2017). Hybrid Code Networks: practical and efficient end-to-end dialog control with supervised and reinforcement learning (pp. 1–13). Presented at the ACL.

[3] Wu, Y., Wu, W., Xing, C., Li, Z., & Zhou, M. (2017). Sequential Matching Network: A New Architecture for Multi-turn Response Selection in Retrieval-Based Chatbots (pp. 1–10). Presented at the ACL.

[4] Wen, T.-H., Vandyke, D., Mrksic, N., Gasic, M., Rojas-Barahona, L. M., Su, P.-H., et al. (2017). A Network-based End-to-End Trainable Task-oriented Dialogue System (pp. 1–12). Presented at the EACL.

[5] Wen, T.-H., Miao, Y., Blunsom, P., & Young, S. (2017). Latent Intention Dialogue Models  (pp. 1–10). Presented at the ICML.

[6] NikolaMrksic, aghdha, D. S., Wen, T.-H., Thomson, B., & Young, S. (2017). Neural Belief Tracker: Data-Driven Dialogue State Tracking  (pp. 1–12). Presented at the ACL.

[7] Li, J., Monroe, W., Ritter, A., Galley, M., Gao, J., & Jurafsky, D. (2016). Deep Reinforcement Learning for Dialogue Generation  (pp. 1–11). Presented at the EMNLP.

[8]Serban, I. V., Sordoni, A., Bengio, Y., Courville, A., & Pineau, J. (2015). Building End-To-End Dialogue Systems Using Generative Hierarchical Neural Network Models (Vol. cs.CL). Presented at the AAAI.

[9] Serban, I. V., Alessandro Sordoni, Lowe, R., Charlin, L., Pineau, J., Courville, A., & Bengio, Y. (2016, June 15). A Hierarchical Latent Variable Encoder-Decoder Model for Generating Dialogues .

[10] Mou, L., Song, Y., Yan, R., Li, G., Zhang, L., & Jin, Z. (2016). Sequence to Backward and Forward Sequences: A Content-Introducing Approach to Generative Short-Text Conversation (pp. 1–10). Presented at the COLING.

[11] Sun, C., & Wang, Z. (2017). Neural Response Generation via GAN with an Approximate Embedding Layer (pp. 1–11). Presented at the EMNLP.

[12] Yan, Z., Duan, N., Bao, J. W., Chen, P., Zhou, M., Li, Z., & Zhou, J. (2016). DocChat: An Information Retrieval Approach for Chatbot Engines Using Unstructured Documents. Presented at the ACL.

[13] Jiwei Li, Michel Galley, Chris Brockett, Jianfeng Gao, and Bill Dolan. 2016a. A diversity-promoting objective function for neural conversation models. In Proc. of NAACL-HLT.

[14] Ranzato, M., Chopra, S., Auli, M., & Zaremba, W. (2016). Sequence Level Training with Recurrent Neural Networks (pp. 1–16). Presented at the ICLR.

[15]Lewis, M., Yarats, D., Dauphin, Y. N., Parikh, D., & Batra, D. (2017). Deal or No Deal? End-to-End Learning for Negotiation Dialogues (pp. 1–11). Presented at the EMNLP.

[16]Lazaridou, A., Peysakhovich, A., & Baroni, M. (2017). Multi-Agent Cooperation and the Emergence of (Natural) Language  (pp. 1–11). Presented at the ICLR.

[17]Lowe, R., Noseworthy, M., Serban, I. V., Angelard-Gontier, N., Bengio, Y., & Pineau, J. (2017). Towards an Automatic Turing Test: Learning to Evaluate Dialogue Responses (pp. 1–20). Presented at the ACL.