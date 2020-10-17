---
title: Tecent第一届社交广告高校算法大赛——萌新经验分享
tags:
  - Algorithm
mathjax: true
categories:
  - Competition
date: 2017-07-01 00:38:37
---



[赛题详情](http://algo.tpai.qq.com/home/information/index.html )是对腾讯的社交广告进行cvr预测，下面讲一下我们两只萌新是怎么进行赛题求解的。

<!-- more -->

## 1. 数据分析与预处理

首先我们对数据进行原始特征的分析，发现不同日期虽然点击量不同，但是转化率大致接近，除了最后几天因为回流时间的影响，导致转化率偏低，因此时间特征我们抛弃了日期，而选择了小时。同时，我们对26-30这几天的数据进行了数据清洗，清洗标准是，统计不同广告主报回流的平均时间，然后用这个时间加上当前数据的点击时间，来预计回流时间，对回流时间超过30号24点的数据进行清洗，这样可以去除肯定错误的一些数据。处理后发现，这样的数据主要集中在30号，因为大部分广告主的回流时间都在集中在24小时内，只有30号受此影响较大。

## 2. 特征工程与特征选择

### 2.1 特征工程

在自己构造的特征方面，我们主要列出以下几个有明显提升的特征：

| 特征名                 | 特征描述                            |
| ------------------- | ------------------------------- |
| tfidf               | 对用户安装列表统计user-app的tfidf值        |
| exptv_A_B           | A和B两个特征的联合转化率（tilltoday），采用均值平滑 |
| exptv_A_B_C         | ABC三个特征的联合转化率（tilltoday），采用均值平滑 |
| dup_non             | 非重复数据                           |
| dup_first(mid/last) | 重复数据的第一条（中间/最后一条）               |
| cnttv_A_B           | A和B两个特征的联合点击数（tilltoday）        |
| user_clickCreat_cnt | 截止到当前用户对某个creativeID的点击数目       |
| user_conver_history | 用户截止到当前的转化历史，用01统计然后Hash处理      |
| time_diff           | 前向时间差，用户距离上一次点击的时间差             |
| next_time_diff      | 后向时间差，用户距离下一次点击的时间差             |
| X_time_diff         | 用户距离上一次点击相同X的时间差                |
| next_X_time_diff    | 用户距离下一次点击相同X的时间差                |

其中，关键的几个点是重复数据要采用onehot方式构造特征，exptv转化率统计的是今天之前的所有数据，且采用的平滑方法是(x+k*means)/(1+k)，k是自己设置的参数，这一方法借鉴的是owenzhang的开源代码，而user_conver_history则是参考台大的开源代码。其实还有构造很多其他特征，但是最终的最优模型没有采用，所以没有一一列出。

### 2.2 特征选择

特征选择方面，我们的最优特征组合是

features = ['tfidf','user_clickApp_tillnow','user_clickCreat_cnt', 'user_conver_history','creativeID','advertiserID','adID','appID','education','clickHour','gender','connectionType','telecomsOperator', 'exptv_pos_connect','exptv_advertiser_pos','exptv_creative_pos','exptv_age_marriage', 'exptv_user_appPlatform','exptv_user_positionType','exptv_age_camgaign','exptv_userID_connectionType','exptv_userID_appCategory','exptv_telecomsOperator_clickHour_connectionType','exptv_sitesetID_connectionType_appID', 'dup_non','dup_first','dup_last','dup_mid','app_time_diff','positionID_time_diff','appCategory_time_diff','next_time_diff' 'next_app_time_diff','next_creativeID_time_diff','next_appCategory_time_diff','next_positionID_time_diff', 'cnttv_advertiser_pos','cnttv_age_pos','cnttv_creative_pos','cnttv_pos_connect']

其中，联合转化率中包含user的几个是复赛中新加入的，在初赛中没有效果，但在复赛中产生的很好的提升，我们分析原因是31号的数据中有大量之前从未出现过的用户，如果在训练时加入用户特征那么在预测的时候就能有提升，相当于一定程度的进行冷启动训练。

其他联合转化率大多是与positionID的组合，效果都还可以；通过前向后向时间差这种leakage特征也带来了较大的提升；用户对于app的tfidf给在决赛中给模型带来了一定的提升，这个提升在初赛中是没有显现出来的；重复数据的trick提升有限，且一定要转化成onehot特征，不过聊胜于无。最后，很多原始特征也是不能删去的，加入和不加入区别较大。



## 3. 建模方法

我们的模型主要是XGB和LGB两个，CNN和FFM只是参与average进行submission文件融合，RF主要是进行Stacking，stacking我们使用了随机森林、XGB、LGB三种模型，这三种模型进行小数据集训练、半随机特征群训练，能构造近十种基模型，stacking后提升明显。

### 3.1 单模型

首先是单模型，我们最好的单模型是采用了LGB的gbdt模型，采用上面列出的40个左右特征，以及经过清洗后的全集数据，30号作为验证集，单模型LGB效果可以达到0.1024，而单模型XGB用同样的特征只能达到0.1029，单模型FFM和单模型CNN线上预估均在0.1034左右，他们的作用是后来可以进行average，提升最后提交文件的分数。

### 3.2 Ensemble

然后是ensemble，我们的模型融合采用了两种方法，分别是stacking和average，均带来了3-4个万分点的提升，但是average具有一定的不稳定性，下面具体展开。

#### 3.2.1 Stacking

我们的stacking采用了以下几种训练基模型的方法：

1. 首先是训练集合多样性：

 ​采用20-24号五折交叉，对25-29进行预测。

 ​采用25-29号五折交叉，对25-29进行预测。

 ​全集17-29号五折交叉，对17-29进行预测，不过最后使用的是25-29的数据。

2. 然后是训练模型多样性：

 ​LGB

 ​XGB

 ​RF	

3. 最后是特征多样性：

 ​固定23维很重要的特征，再从特征群中随机采样其他特征，训练单模型XGB。

这样可以组合出很多种基模型，我们的stacking主要采用了lgb2024，lgb2529，xgb2024，xgb2529，RF2024，RF2529， lgb_all，xgb_all，RF_all这9列特征进行第二层LGB的训练，结果比单模型提升了近四个万分点。

然后我们又采用特征多样性来进行stacking，也就是固定23维重要特征，然后从特征群中采样另外30维特征，进行基模型训练，这样得到了10维特征进行stacking，这个stacking的结果虽然只有0.1025，但是在下面的average中产生了很好的效果。

#### 3.2.2 Average

average是直接对历史提交文件进行人工加权融合，我们发现数据分布差异性较大的但是分数相近的submission文件，进行加权融合会产生很大的提升，例如把stacking结果和最好的单模型进行融合，可以提升三个万分点；把CNN和FFM这两个表现不是很好的模型再和单模型XGB的结果进行融合，也比单模型XGB要提升3个万分点。

我们最终把stacking的最好结果去和最好的单模型LGB以及之前提交过的一些文件进行加权融合，达到了0.1017的成绩，而融合之前的表现最好的单个文件在0.1020左右，最后一天把随机特征stacking又加进去，以及提升后的xgb的submission文件，提升到了最终的分数。

## 4. 算法运行环境

我们的模型基本都是在17-29进行训练，30验证，31测试，运行环境是中科院自动化所模式识别实验室的公用计算集群，在不同的cpu节点、gpu节点都跑过，所以代码运行时间没有一个准确的估计，需要根据具体的运行环境决定。

代码至少需要60G的运行内存，CNN是基于tensorflow实现的，所以需要在CPU上进行embedding，在GPU上进行模型训练，其他模型均只占用CPU资源。

## 5. 其他观察和思考

关于比赛，最大的遗憾就是没有精力做FFM这个大家都很推崇的模型，感觉要是能把FFM做好，stacking分数会提升不少，且之后的average也会有很大的提升。
