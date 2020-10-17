---
title: BOT从入门到放弃之task-oriented
tags:
  - bot
mathjax: true
categories:
  - bot
date: 2017-07-15 20:21:42
---

task-oriented类型的对话系统，是有明确的任务导向的，一般用于订酒店之类的任务。

本文首先在第一部分介绍了end2end memory网络结构，文章2是在MemN2N网络的基础上，引入了三种学习方式：模仿学习、强化学习、提前预测式学习；文章3提出了一种交互式的对话方法，让bot进行反问，学习什么时候该反问，论证了反问会带来效果提升；文章4是我目前没发觉到值得学习之处的一篇ICLR2017；文章5是google提出的加入用户模型（画像）的一种bot；文章6提出了一个比较严谨的任务型bot框架，采用深度学习和强化学习结合的方法训练。7则是基于检索式的对话，8、9是深度强化学习任务驱动的对话系统。

<!-- more -->

# 1. MemN2N

Sukhbaatar, S. (2015). End-To-End Memory Networks, 1–11.

这是一个[memory network](http://blog.csdn.net/u011274209/article/details/53384232?ref=myread)的改进版本，用于端到端的生成任务，前面的链接有讲memory network的工作原理，本文主要解释MemN2N的结构和一些细节。

end-to-end memory network，简称MemN2N，下面左右两图分别是单层和多层的MemN2N。



![](https://ws1.sinaimg.cn/large/006tNc79ly1fhku2l9qchj319w0jen0o.jpg)

## 单层

先拿单层进行说明，我们有一系列的$x_i$作为输出，构建记忆模块$m_i$，构建方法是用embedding矩阵$A$对$x_i$进行向量化表示；然后再用矩阵$B$对输入$q$向量化，表示为$u$，然后通过内积求出输入与记忆的关联：
$$
p_i = Softmax(u^Tm_i)
$$
$x_i$还要通过矩阵C向量化得到输出模块$c_i$，输出$o$可以表示为
$$
o=\sum_ip_ic_i
$$
最终的输出是
$$
\hat{a}=Softmax(W(o+u))
$$

## 多层

如果扩展到多层，则参考图(b)，下一层的输入是
$$
u^{k+1}=u^k+o^k
$$
每一层都有自己的$A^k$和$C^k$，用于对x向量化。文章提出了两种权重设置：

- 邻接：$A^{k+1}=C^k$, $W^{T}=C^k$, $B=A^1$
- 层级：$A^1=...=A^k$, $C^1=…=C^k$, $u^{k+1}=Hu^k+o^k$

## 细节

- 不想忽略句子中单词顺序，因而加入位置信息

$$
m_i = \sum_jl_j•Ax_{ij}
$$

其中j表示第i句第j个词，$l_{kj} = (1-j/J)-(k/d)(1-2j/J)$, 其中$J$是句子长度，$d$是$l$的维度

- 同时，不同句子之间的先后顺序也不能忽略，那么加入时序信息

$$
m_i = \sum_jAx_{ij}+T_A(i)
$$

$T_A(i)$是$T_A$的第i行，$T_c$同理

- 加入一些噪声（10%的空白）记忆



# 2. Dialog-based Language Learning

Weston, J. (2016, April 21). Dialog-based Language Learning. *arXiv.org*.

本文在[MemN2N](https://liangguirong5.github.io/2017/07/15/end-to-end-memory-network/)的基础上，提出了10种对话任务(见下图)，和三种训练方式：imitation learning，reward-based imitation，forward prediction。

![](https://ws4.sinaimg.cn/large/006tKfTcly1fhls2xviifj30h10ie43a.jpg)

## 数据集构建

文中的试验数据集是bAbI和MovieQA, 下载地址http://fb.ai/babi。

针对上面的十个任务需要分别构建不同的数据集合，每个任务在构建数据集合的时候都分别设置三个$\pi_{acc}$(0.5,0.1,0.01)，这个参数表示回答正确的概率。

## 模型学习

基模型还是MemN2N，只是输出层最后的softmax变了，

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhlt2xme6rj30ce01vt8r.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhlt36kkqtj30cb022dfw.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhlt3cexhvj30bj01uq2y.jpg)

新加的$y_1...y_C$表示的是候选集合C中的每一项。文中的C是bAbI任务训练集合中所有出现过的actions，对于MovieQA就是他的KB的全部单词。

- imitation learning：有监督学习，默认训练集中的answer就是标准回答，进行训练。
- reward-based imitation：好的回复产生+1的reward，不好的回复reward为0，简单的策略就是对好的回复采用imitation learning，不好的直接弃之不理。

![](https://ws2.sinaimg.cn/large/006tKfTcly1fhlstlrgecj30fx0bvjsf.jpg)

- forward prediction：把输出层更改成下图的样子。

![](https://ws3.sinaimg.cn/large/006tKfTcly1fhltkizzxyj30dq0cqt9x.jpg)

具体来讲，是给定question和ans，想要预测ans对应teacher的response，我理解的response就是reward为正还是0，

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhm069qsp2j30j2024q33.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1fhm09i7qlyj30b0019mx6.jpg)

其中，yi是student的候选回复，$\beta$是d维向量表示student实际选择的action

- reward-based imitation + forward prediction: 两种方法相结合

## Results

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhm1m2cqpfj30tn0c643u.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1fhm1mm5dw0j30t90bjdkm.jpg)



# 3. LEARNING THROUGH DIALOGUE INTERACTIONS BY ASKING QUESTIONS

Li, J., & Weston, J. (2017). LEARNING THROUGH DIALOGUE INTERACTIONS BY ASKING QUESTIONS, 1–16.

代码 https://github.com/facebook/MemNN/tree/master/AskingQuestions

本文研究了基于交互的生成式对话方法，证明了bot在反问后，聊天质量明显提升。

主要借鉴的是前面的	Jason Weston. Dialog-based language learning. arXiv preprint arXiv:1604.06045, 2016.

数据集采用WikiMovies dataset，是一个roughly 100k questions over 75k entities based on questions with answers in the open movie dataset (OMDb)

文章希望通过交互式的反问解决一下三类常见错误：bot不理解问题；不能把现有的知识和问题建立关联；缺乏回答问题必备的知识

## The Tasks

### 1 QUESTION CLARIFICATION

文章具体指问题中夹杂拼写错误的词，解决方法是bot或者请求重新提问，或者反问Do you mean [correct] ?

1.Question Paraphrase & 2.Question Verification

### 2 KNOWLEDGE OPERATION

bot尝试去寻找相关的知识回答，可以反问问题是否与***相关？也可以直接让提问者指出相关的KB知识。

3.ask for revelvant knowloge & 4.knowledge verification

### 3 KNOWLEDGE ACQUISITION

KB不完整，只能请求提问者给出答案，训练bot使得下次碰到相同的问题能回答正确答案。

5.missing question entity 6. missing answer entity 7. missing relation entity 8. missing triple 9.missing everything

## Train/Test Regima

模型训练的目标有两个：

​	检验这种交互式的能否带来效果提升；

​	学习到什么时候应该反问

文章采用线下有监督，线上强化学习的方法分别实现这两个目标。

### 1. offline

训练集TrainQA, TrainAQ, and TrainMix，测试集 TestQA, TestAQ, and TestModelAQ，这样组合可以有9种，然后对于前面提到的9个任务分别训练对应的模型，每种模型又分别在前面9种组合的train、test数据上进行训练。举例说明，在QA上训练，AQ上测试，就是观察一个训练中从来不发问的student在测试时每次必问的表现；在AQ上训练、QA上测试亦是如此。需要注意的是，TrainMix是一半QA一半AQ，TestModelAQ是要保证反问的问题必须要和问题相关。

### 2. online RL

每次提问都会有cost，cost的大小反映了teacher的耐心程度，取值范围是[0,2]

![](https://ws3.sinaimg.cn/large/006tNc79ly1fhkhr74qc5j30qa04gt9e.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1fhmmk0skvjj30eb028mxd.jpg)

其中，$a_1$表示是否提问，$a_2$表示最终的回答。

# 4. Learning End-to-End Goal-Oriented Dialog 

Bordes, A., & Weston, J. (2017). Learning End-to-End Goal-Oriented Dialog (pp. 1–15). Presented at the ICLR.

借用[另一篇博文](http://blog.csdn.net/abcjennifer/article/details/53428053#end-to-end-generative-dialogue)的话来评论本文：

> 评论：数据较弱，方法不实用。
>
> 本文就是End-to-End Memory Network的一个应用，基于End-to-End Memory Network的实验，最终也是answer selection(而不是answer generation). 数据为以餐馆预订为目的地bAbI数据集，模型定义得非常个性化，难以迁移。

# 6. End-to-end LSTM-based dialog control optimized with supervised and reinforcement learning

Williams, J. D., & Zweig, G. (2016, June 4). End-to-end LSTM-based dialog control optimized with supervised and reinforcement learning. *arXiv.org*.

本文用dl和rl训练下图模型，模型其实非常简单，先对query进行NER，然后存储下识别好的entity，同时送给lstm以entity的type和values，lstm的输出是所有回复的模板的概率分布，用RL就从中采样，用DL就取最大，得到的answer模板是没有填充实体value的，这时候需要用写好的规则去填充，然后判断是回复给对话者还是进行api call。

![](https://ws1.sinaimg.cn/large/006tKfTcly1fhq9t47j9lj31go0istc3.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1fhqegimc0mj30pq05iq3o.jpg)

# 7. Sequential Matching Network: A New Architecture for Multi-turn Response Selection in Retrieval-Based Chatbots 

Wu, Y., Wu, W., Xing, C., Li, Z., & Zhou, M. (2017).  Presented at the ACL.

![](https://ws3.sinaimg.cn/large/006tNc79ly1fi5e7h3etsj319e0jowkc.jpg)



# 8. A Network-based End-to-End Trainable Task-oriented Dialogue System 

Wen, T.-H., & Young, S. (2017). Presented at the EACL.

![](https://ws2.sinaimg.cn/large/006tNc79ly1fi5ezd5ux2j31140p80xb.jpg)

![](https://ws3.sinaimg.cn/large/006tNc79ly1fi5ezq9i6zj315g0pe0ya.jpg)

# 9. Latent Intention Dialogue Models

Wen, T.-H., Miao, Y., Blunsom, P., & Young, S. (2017). Presented at the ICML.

![](https://ws2.sinaimg.cn/large/006tNc79ly1fi5f0vtrjrj31i80ua46f.jpg)

# 10. Neural Belief Tracker: Data-Driven Dialogue State Tracking  

![](https://ws2.sinaimg.cn/large/006tNc79ly1fjelanivc6j31040qq0yj.jpg)