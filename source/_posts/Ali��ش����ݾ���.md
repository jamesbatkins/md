title: 阿里天池大数据竞赛之资金流入流出预测笔记
date: undefine
tags: [大数据竞赛, 资金流入流出,笔记]

---



原先提交的结果是利用线性回归，因变量是每日申购赎回的总金额，自变量选择的是星期几，将每天是星期几和节假日作为特征进行线性回归。

因为这个申购赎回是由有周期性，与时间有关，同时与各种节假日也有关系。

简单的线性回归能够得到 185 分左右。

<!-- more -->

###2015-07-05 arima模型的使用###
今天想尝试的方式是在 ODPS 上利用 R 脚本实现 ARIMA 模型，进行资金预测。

对于平台 R 的利用很是不熟悉，运行脚本经常遇到各种错误，比如，对于数据框 frame 列数据的获取存在问题；对于 ARIMA 中添加个 xreg 参数总是会出现 no suitable ARIMA model found，网上查了下，好像是 R 低版本中的 bug。

还有一个比较傻的是，一直忘记了在 auto.arima() 中添加差分因子 d。

刚看了成绩，提高了 0.3 分。 (20150706 0:34)


###2015-07-07 回归检测与优化###

![grades](/images/record.png)

刚查完成绩，上升了 23 名， 192分。这次使用的仍然是线性回归，不过改动之处在于优化线性回归中的自变量选择，同时去掉数据中的异常数据。

恩，看完分数之后，一阵阵激动，给自己增加了许多信心。不要放弃，坚持。相信一定会有收获的。（20150707 0：33）