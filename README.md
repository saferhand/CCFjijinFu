# CCF-大数据竞赛-基金间的相关性预测-三蹦子-复赛15名

最终得分： 0.84048060

队员: 一叶障目, 我想给你生鸡蛋

<a href = "https://www.datafountain.cn/competitions/312/details/data-evaluation">赛题介绍</a>

### 代码执行
1. 执行 genXY.R，生成训练特征；
2. 执行 lgbTrainVal.R，将数据分成训练集，验证集和测试集， 训练LGB模型， 利用验证集的表现来调试参数；
3. 执行 lgbFull.R，利用第二步得到的参数在所有可用训练数据上训练LGB模型并在测试集上预测；
4. 执行meanCorrection.R，用历史均值替代部分LGB预测的基金对相关性。 
原理是有些基金对相关性在所给数据的时间长度内基本没有改变。这个步骤能取到些许的提升（0.0001~0.0005）。
    
### 基本思路
1. 一开始直接分析给定correlation， 发现基本是RANDOM WALK， 连续两天的correlation的相关性很高。 
而且很多人也提到了， 题目算的correlation其实就是未来61天return的(weighted) pearson correlation。
因此， 可以认为重点是在fund_return的数据上， 而且时间相隔越远的数据应该影响越小。
这一步， 其实我心里很虚，因为我觉得这可能是个不可预测的问题。
2. 这里我将每个基金对， 每天对应的CORRELATION作为一个观察值， 之后用lightgbm训练。
3. 我这里构造的特征很简单， 只利用到了fund_return的数据。 就是算过去两个基金过去不同时间长度的局部pearson correlation。
最终选定了三个时间长度， 11天，16天， 21天。
另外， 我还加了两个categorical variables： i,j, 分别表示当前基金对对应基金i和基金j。
也就是说总共只有5个特征， 2离散+3连续。

### 关于线下验证集
我选择了2017-12-14（给定相关性数据的最后一天）作为验证集， 然后在这之前的61天数据丢弃， 用剩下的数据来做训练集。
这样我测试发现线下的分数基本和线上分数吻合， 在线下不超过0.84的情况下。
一旦线下超过0.84的时候， 我的模型就过拟合了。

### 其他尝试
1. 首先我看过一些fund_return时间序列图， 基本认定都是white noise, 也符合金融里一般的假设，也就说直接预测股票回报率其实是不靠谱。
大部分股票的volatility倒是会一直在变， 就目前给的时间段看，是先大后小，可以去查查看2015年底股市发生了什么。
2. 初赛尝试过DEEP LEARNING，表现不行， 可能设置的网络不大对。 复赛内存消耗太大，我内存16G， 而且我的渣渣2G显存也选择了放弃。
3. 为了减少内存消耗，尝试过将基金对分成不同块来训练模型， 结果并不理想。 
3. 我试过加入根据fund_benchmark_return和index_return构造的数据， 但是结果反而更差了， 不管是本地还是线上。 
我觉得我可能缺的就是这一块的处理， 导致分数上不去。
4. 传统时间序列模型， 比如multivariate (dynamic) volitility model也是有在估计回报率相关性， 但是与题目相关性其实定义不一样， 不能用来预测。
尝试过，效果很糟糕。
5. 目前我还有个想法没去实现的就是， 构造fund_return和index_return的对应关系， 
相当于用index_return去降维fund_return, 原因是很多基金都是市场指数的组合。
接下来我们只需预测市场指数的相关性， 然后用以上关系进一步推基金间相关性就好了。
当然这样分几步训练模型也许累积误差会很大。







