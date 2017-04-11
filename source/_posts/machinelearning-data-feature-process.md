title: 机器学习中的数据清洗与特征处理综述
date: 2015/06/08
tags: [机器学习, 数据清洗, 特征处理]

---


### 概述

>注：本文摘取于机器学习 InAction 系列讲座课程，本文源地址请点击[链接](http://tech.meituan.com/machinelearning-data-feature-process.html)。

机器学习常见的处理框架如下图所示：

![](/images/ml_frame.jpg)

数据清洗和特征挖掘的工作是在灰色框中的部分，也就是灰色框中“数据清洗 --> 数据特征向量， 标注数据生成 --> 模型学习 --> 模型应用” 中的前两个步骤。

<!-- more -->

灰色框中蓝色箭头对应的是离线处理部分。主要工作是：

- 从原始数据，如文本、图像或者应用数据中清洗除特征数据和标注数据；
- 对清洗出来的特征和标注数据进行处理，例如 **样本采样，样本调权，异常点去除，特征归一化处理，特征变化，特征组合等过程**。最终生成的数据主要是提供给模型训练使用。

灰色框中的绿色箭头对应的是在线处理部分。所做的主要工作和离线方式类似，主要区别在于：

1. 不需要清洗标注数据，只需得到特征数据即可，在线模型使用特征数据预测出样本可能的标签。
2. 最终生成数据的用处。最终生成的数据主要用于模型的预测，而不是训练。

在离线的处理部分，可以进行较多的实验和迭代，尝试不同的样本采样、样本权重、特征处理方法、特征组合方法等，最终得到一个最优的方法，在离线评估好的方式之后，最终将确定的方案在线使用。

由于在线和离线方式不同，获取数据、存储数据的方式存在较大的差异。例如离线数据获取可以将数据存储在 Hadoop 分布式平台中，批量的进行分析处理等操作，并且容忍一定的失败。而在线服务获取数据需要稳定、延时小等，可以将数据键入索引、存入 KV 存储系统等。

###特征使用方案###

确定分析的目标之后，下一步需要确定使用哪些数据来达到目标。首先要确定哪些数据与目标相关。我们可以根据业务经验来选择特征，也可以采用特征选择、特征分析等方式选择。

确定好要使用哪些数据之后，我们要对选择的数据进行可用性评估，包括数据的获取难度，数据的规模，数据的准确率，数据的覆盖率等。

###特征获取方案###

当确定好要用的数据之后，要考虑的问题是，这些数据从哪里可以获取？

离线特征获取：
离线可以使用海量的数据，借助于分布式文件存储平台，例如 HDFS 等，使用如 MapReduce，Spark 等处理工具来处理海量的数据等。

在线特征获取方案：
在线特征比较注重获取数据的延时性，由于在线服务，需要在非常短的时间内获取相应的数据，对于查找性能非常高，可以将数据存储在索引、键值对中等。而查找性能与数据量存在矛盾，可以使用折中的方式，例如特征分层获取方案，如下图：

![](/images/rank_frame.jpg)

在粗排阶段，使用基础的特征，数据直接键入索引。在精排阶段使用一些个性化特征。


###特征与标注数据清洗###

在了解特征数据放在哪儿、怎样获取之后，下一步就是考虑如何处理特征和标注数据了。

###标注数据清洗###
清洗特征数据可以分为离线和在线方式两种。

离线清洗方式：
优点是方便评估新特征效果，缺点是实时性差，与线上实时环境有一定误差。对于实时特征难以得到恰当的权重。

在线清洗方式：
实时性强，完全记录线上实时数据，缺点是新特征加入需要一段时间做数据积累。

###样本采样与样本过滤###

特征数据只有在和标注数据合并之后，才能作为模型的训练集。下面介绍清洗标注数据。

数据采样：例如对于分类问题，选取正例，负例。对于回归问题，需要采集数据。对于采样得到的样本，根据需要，需要设定样本权重。当模型不能使用全部的数据来训练时，需要对数据进行采样，设定一定的采样率。采样的方法包括随机采样，固定比率采样等方法。

除采样外，经常对样本还需进行过滤，包括
1. 结合业务情况进行数据过滤，例如去除 crawler 抓取， spam， 作弊等数据。
2. 异常点检测，采用异常点检测算法对样本进行分析，常用的异常点检测算法包括

 - 偏差检测，例如聚类，最邻近等；
 - 基于统计的异常点检测算法，例如极差，四分位数据间距(箱线图)，均差，标准差等。这种方法适用于挖掘单变量的数值型数据。全距(Range)，又称为极差，是用来表示统计资料中的变异量数(measures of variation)，其最大值和最小值之间的差距。
 - 基于距离的异常点检测算法，主要通过距离方法来检测异常点，将数据集中与大多数点之间的距离大于某个阈值的点视为异常点，主要使用的距离度量方法有绝对距离(曼哈顿距离)、欧式距离和马氏距离等方法。
 - 基于密度的异常点检测算法，考察当前点周围密度，可以发现局部异常点，例如 LOF 算法。

###特征分类###

在分析完特征和标注的数据清洗方式后，下面具体介绍下特征的处理方法，先对特征进行分类，对不同的特征应该有不同的处理方法。

根据不同的分类方法，可以将特征分为

1. Low level 特征和 High level 特征
2. 稳定特征与动态特征
3. 二值特征、连续特征、枚举特征

Low leve 特征是较低级别的特征，主要是原始特征，不需要或者需要非常少的人工处理和干预，例如文本特征中的词向量特征，图像特征中的像素点，用户 id，商品 id 等。Low level特征一般维度比较高，不能用于过于复杂的模型。High level 特征是经过较复杂的处理，结合部分业务逻辑或者规则、模型得到的特征，例如人工打分，模型打分等特征，可以用于较复杂的非线性模型。Low level 比较针对性，覆盖面小。长尾样本的预测值主要受 high level 特征影响。高频样本的预测值主要受 low level 特征影响。

稳定特征是变化频率(更新频率)较少的特征，例如评价平均分，团购单价格等，在较长时间段内不会发生变化。动态特征是更新变化比较频繁的特征，有些甚至是实时计算得到的特征，例如距离特征、2小时销售等特征。针对这两种特征的不同可以针对性的设计特征存储和更新方式，例如对于稳定特征，可以建入索引，较长时间更新一次，如果做缓存的话，缓存时间可以较长。对于动态特征，需要实时计算或者准实时地更新数据，如果做缓存的话，缓存期过期时间需要设置的较短。

二值特征主要是 0/1 特征，及特征只取两中值：0 或者 1，例如用户 id 特征：目前某个 id 是否是某个特定的 id，词向量特征：某个特定的词是否在文章中出现等。连续特征是取值为有理数的特征，特征取值个数不定，例如距离特征，特征取值为是 0～正无穷。枚举值特征主要是特征有固定个数个可能值，例如今天周几。在实际的使用中，我们可能对不同类型的特征进行转换，例如将枚举特征或者连续特征处理为二值特征。枚举特征处理为二值特征技巧：将枚举特征映射为多个特征，每个特征对应于一个特定枚举值，例如今天周几，可以把它转换成7个二元特征：今天是否为周一，今天是否为周二，...，几天是否为周日。连续值处理为二值特征值：先将连续值离散化，再将离散化后的特征切分为 N 个二元特征，每个特征代表是否在这个区间内。

###特征处理与分析####

在对特征进行分类之后，下面介绍对特征常用的处理方法。包括：特征归一化，离散化，缺省值处理，特征降维法，特征选择法等。

###特征归一化，离散化，缺省值处理###

主要用于单个特征的处理：

- 归一化：不同的特征有不同的取值范围，在有些算法中，例如线性模型或者距离相关的模型聚类模型、knn模型等，特征的取值范围会对最终的结果产生较大的影响，例如二元特征的取值范围为[0,1]，而距离特征取值可能是[0,正无穷），在实际使用会对距离进行截断，例如，[0,300000],但是这两个特征由于取值范围不一致导致了模型可能会更偏向于取值范围大的特征，为了平衡取值范围不一致的特征，需要对特征进行归一化处理，将特征取值归一化到[0,1]区间。常用的归一化方法包括
 
	- 函数归一化，通过映射函数将特征值映射到[0,1]区间，例如最大最小值归一化方法，是一种线性的映射。还有通过非线性函数的映射，例如 log 函数等。
	- 分维度归一化,可以使用最大最小归一化，但是最大最小值选取的是所属类别的最大最小值，即使用的局部最大最小值，不是全局的最大最小值。
	- 排序归一化，不管原来的特征值是什么样的，将特征按大小排序，根据特征所对应的给予一个新的值。

- 离散化：在上面介绍过连续值的取值空间可能是无穷的，为了便于表示和在模型中处理，需要对连续特征值进行离散化处理。常用的离散化方法包括等值划分和等量划分。等值划分是将特征按照值域进行均分，每一段内的取值等同处理。例如某个特征的取值范围为[0,10],我们将其划分为10段，[0,1),[1,2),...,[9,10)。等量划分是根据样本总数进行均分，每个段等量个样本划分为1段。例如距离特征，取值范围为[0,30000),为了使每一段中包含的样本数量相同，我们就可以将它划分为[0,100),[100,300),[300,500),...,[10000,30000]，这样前面的区间划分比较密，后面的比较稀疏。
- 缺省值处理：有些特征取值可能因为无法采样或者没有观测值而缺失，例如距离特征，用户可能禁止获取地理位置后者位置获取失败，此时需要对这些特征做特殊的处理，赋予一个缺省值。缺省值的赋予方法常用的有单独表示，众数，平均值等。

###特征降维###
在介绍特征降维之前，先介绍下特征升维。在机器学习中，有一个VC维理论，VC维越高，打散能力越强，可容许的模型复杂度越高。在低维不可分的数据，映射到高维是可分的。可以想想，给你一堆物品，人脑是如何对这些物品进行分类，依然是找出这些物品的一些特征，例如，颜色，形状，大小等。然后，根据这些特征对物品做以归类，这其实就是个先升维后划分的过程。比如我们人脑识别香蕉。可能首先我们发现香蕉是黄色的。这是在颜色这个维度的一个切分。但是很多东西都是黄色的啊，例如哈密瓜。那么怎么区分香蕉和哈密瓜呢？我们发现香蕉形状是弯曲的。而哈密瓜是圆形的，那么我们就可以用形状来把香蕉和哈密瓜划分开了，即引入一个新维度：形状，来区分。这就是一个从“颜色”一维特征升维到二维特征的例子。

那问题来了，既然升维后模型能力能变强，那么是不是特征维度越高越好呢？为什么要进行特征降维&特征选择？主要是出于如下考虑：
1. 特征维数越高，模型越容易过拟合，此时更复杂的模型就不好用。
2. 相互独立的特征维数越高，在模型不变的情况下，在测试集上达到相同的效果表现所需要的训练样本的数目就越大。 
3. 特征数量增加带来的训练、测试以及存储的开销都会增大。
4. 在某些模型中，例如基于距离计算的模型KMeans，KNN等模型，在进行距离计算时，维度过高会影响精度和性能。
5. 可视化分析的需要。在低维的情况下，例如二维，三维，我们可以把数据绘制出来，可视化地看到数据。当维度增高时，就难以绘制出来了。
在机器学习中，有一个非常经典的维度灾难的概念。用来描述当空间维度增加时，分析和组织高维空间，因体积指数增加而遇到各种问题场景。例如，100个平均分布的点能把一个单位区间以每个点距离不超过0.01采样；而当维度增加到10后，如果以相邻点距离不超过0.01小方格采样单位超一单位超正方体，则需要10^20 个采样点。

正是由于高维特征有如上描述的各种各样的问题，所以我们需要进行特征降维和特征选择等工作。特征降维常用的算法有PCA，LDA等。特征降维的目标是将高维空间中的数据集映射到低维空间数据，同时尽可能少地丢失信息，或者降维后的数据点尽可能地容易被区分

- PCA算法：通过协方差矩阵的特征值分解能够得到数据的主成分，以二维特征为例，两个特征之间可能存在线性关系（例如运动的时速和秒速度），这样就造成了第二维信息是冗余的。PCA的目标是发现这种特征之间的线性关系，并去除。
- LDA算法：考虑label，降维后的数据点尽可能地容易被区分

###特征选择###

特征选择的目标是寻找最优特征子集。特征选择能剔除不相关(irrelevant)或冗余(redundant )的特征，从而达到减少特征个数，提高模型精确度，减少运行时间的目的。另一方面，选取出真正相关的特征简化模型，协助理解数据产生的过程。
特征选择的一般过程如下图所示：

![](/images/ml_feature_selection_frame.png)

主要分为产生过程，评估过程，停止条件和验证过程。

###特征选择-产生过程和生产特征子集方法###

- 完全搜索
	- 广度优先搜索（BFS）：广度优先遍历特征子空间。枚举所有的组合，穷举搜索，实用性不高。
	- 分支限界搜索（Branch and Bound）：穷举基础上加入分支限界。例如：剪掉某些不可能搜索出比当前最优解更优的分支。
   其他，如定向搜索 (Beam Search )，最优优先搜索 ( Best First Search )等
- 启发式搜索（Heuristic）
	- 序列前向选择（SFS）：从空集开始，每次加入一个最优。
	- 序列后向选择（SBS）：从全集开始，每次减少一个选最优。
	- 增L去R选择算法（LRS，Plus-L Minus-R Selection):从空集开始，每次加入L个，减去R个，选最优（L>R)或者从全集开始，每次减去R个，增加L个，选最优(L<R)。
- 随机搜索
	- 随机产生序列选择算法(RGSS， Random Generation plus Sequential Selection)：随机产生一个特征子集，然后在该子集上执行SFS与SBS算法。
	- 模拟退火算法( SA， Simulated Annealing )：以一定的概率来接受一个比当前解要差的解，而且这个概率随着时间推移逐渐降低
	- 遗传算法( GA， Genetic Algorithms )：通过交叉、突变等操作繁殖出下一代特征子集，并且评分越高的特征子集被选中参加繁殖的概率越高。
	
随机算法共同缺点:依赖随机因素，有实验结果难重现。

###特征选择-有效性分析###

对特征的有效性进行分析，得到各个特征的特征权重，根据是否与模型有关可以分为

1.与模型相关特征权重，使用所有的特征数据训练出来模型，看在模型中各个特征的权重，由于需要训练出模型，模型相关的权重与此次学习所用的模型比较相关。不同的模型有不同的模型权重衡量方法。例如线性模型中，特征的权重系数等。

2.与模型无关特征权重。主要分析特征与label的相关性，这样的分析是与这次学习所使用的模型无关的。与模型无关特征权重分析方法包括(1)交叉熵，(2)Information Gain，(3)Odds ratio，(4)互信息，(5)KL散度等

在机器学习任务中，特征非常重要
根据经验，80%的效果有特征带来的。下图是随着特征数的增加，最终模型预测值与实际值的相关系数变化。

![](/images/ml_feature_increase.jpg)