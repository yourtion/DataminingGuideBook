## 每加仑燃油可以跑多少公里？

最后，我们再来测试另一个广泛使用的数据集，卡内基梅隆大学统计的汽车燃油消耗和公里数数据。

它在1983年的美国统计联合会展中使用过，大致格式如下：

![](../img/chapter-4/chapter-4-45.png)

这个数据集做过一些修改。（下载：[mpgTrainingSet.txt](https://raw.githubusercontent.com/yourtion/DataminingGuideBook-Codes/master/chapter-4/mpgTrainingSet.txt)、[mpgTestSet.txt](https://raw.githubusercontent.com/yourtion/DataminingGuideBook-Codes/master/chapter-4/mpgTestSet.txt)）

我们要预测的是加仑燃油公里数（mpg），使用的数据包括汽缸数、排气量、马力、重量、加速度等。

![](../img/chapter-4/chapter-4-46.png)

数据集中有342条记录，50条测试记录，运行结果如下：

```python
>>> test('mpgTrainingSet.txt', 'mpgTestSet.txt')
56.00% correct
```

如果不进行标准化，准确率将只有32%。

![](../img/chapter-4/chapter-4-47.png)

> 我们应该如何提高预测的准确率呢？改进分类算法？增加训练集？还是增加特征的数量？我们将在下一章揭晓！
