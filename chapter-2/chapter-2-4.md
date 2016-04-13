## Python推荐模块

我将本章学到的内容都汇集成了一个Python类，虽然[代码](https://github.com/yourtion/DataminingGuideBook-Codes/tree/master/chapter-2/recommender.py)有些长，我还是贴在了这里：

```python
import codecs 
from math import sqrt

users = {"Angelica": {"Blues Traveler": 3.5, "Broken Bells": 2.0,
                      "Norah Jones": 4.5, "Phoenix": 5.0,
                      "Slightly Stoopid": 1.5,
                      "The Strokes": 2.5, "Vampire Weekend": 2.0},
         
         "Bill":{"Blues Traveler": 2.0, "Broken Bells": 3.5,
                 "Deadmau5": 4.0, "Phoenix": 2.0,
                 "Slightly Stoopid": 3.5, "Vampire Weekend": 3.0},
         
         "Chan": {"Blues Traveler": 5.0, "Broken Bells": 1.0,
                  "Deadmau5": 1.0, "Norah Jones": 3.0, "Phoenix": 5,
                  "Slightly Stoopid": 1.0},
         
         "Dan": {"Blues Traveler": 3.0, "Broken Bells": 4.0,
                 "Deadmau5": 4.5, "Phoenix": 3.0,
                 "Slightly Stoopid": 4.5, "The Strokes": 4.0,
                 "Vampire Weekend": 2.0},
         
         "Hailey": {"Broken Bells": 4.0, "Deadmau5": 1.0,
                    "Norah Jones": 4.0, "The Strokes": 4.0,
                    "Vampire Weekend": 1.0},
         
         "Jordyn":  {"Broken Bells": 4.5, "Deadmau5": 4.0,
                     "Norah Jones": 5.0, "Phoenix": 5.0,
                     "Slightly Stoopid": 4.5, "The Strokes": 4.0,
                     "Vampire Weekend": 4.0},
         
         "Sam": {"Blues Traveler": 5.0, "Broken Bells": 2.0,
                 "Norah Jones": 3.0, "Phoenix": 5.0,
                 "Slightly Stoopid": 4.0, "The Strokes": 5.0},
         
         "Veronica": {"Blues Traveler": 3.0, "Norah Jones": 5.0,
                      "Phoenix": 4.0, "Slightly Stoopid": 2.5,
                      "The Strokes": 3.0}
        }


class recommender:

    def __init__(self, data, k=1, metric='pearson', n=5):
        """ 初始化推荐模块
        data   训练数据
        k      K邻近算法中的值
        metric 使用何种距离计算方式
        n      推荐结果的数量
        """
        self.k = k
        self.n = n
        self.username2id = {}
        self.userid2name = {}
        self.productid2name = {}
        # 将距离计算方式保存下来
        self.metric = metric
        if self.metric == 'pearson':
            self.fn = self.pearson
        #
        # 如果data是一个字典类型，则保存下来，否则忽略
        #
        if type(data).__name__ == 'dict':
            self.data = data

    def convertProductID2name(self, id):
        """通过产品ID获取名称"""
        if id in self.productid2name:
            return self.productid2name[id]
        else:
            return id

    def userRatings(self, id, n):
        """返回该用户评分最高的物品"""
        print ("Ratings for " + self.userid2name[id])
        ratings = self.data[id]
        print(len(ratings))
        ratings = list(ratings.items())
        ratings = [(self.convertProductID2name(k), v)
                   for (k, v) in ratings]
        # 排序并返回结果
        ratings.sort(key=lambda artistTuple: artistTuple[1],
                     reverse = True)
        ratings = ratings[:n]
        for rating in ratings:
            print("%s\t%i" % (rating[0], rating[1]))

    def loadBookDB(self, path=''):
        """加载BX数据集，path是数据文件位置"""
        self.data = {}
        i = 0
        #
        # 将书籍评分数据放入self.data
        #
        f = codecs.open(path + "BX-Book-Ratings.csv", 'r', 'utf8')
        for line in f:
            i += 1
            #separate line into fields
            fields = line.split(';')
            user = fields[0].strip('"')
            book = fields[1].strip('"')
            rating = int(fields[2].strip().strip('"'))
            if user in self.data:
                currentRatings = self.data[user]
            else:
                currentRatings = {}
            currentRatings[book] = rating
            self.data[user] = currentRatings
        f.close()
        #
        # 将书籍信息存入self.productid2name
        # 包括isbn号、书名、作者等
        #
        f = codecs.open(path + "BX-Books.csv", 'r', 'utf8')
        for line in f:
            i += 1
            #separate line into fields
            fields = line.split(';')
            isbn = fields[0].strip('"')
            title = fields[1].strip('"')
            author = fields[2].strip().strip('"')
            title = title + ' by ' + author
            self.productid2name[isbn] = title
        f.close()
        #
        #  将用户信息存入self.userid2name和self.username2id
        #
        f = codecs.open(path + "BX-Users.csv", 'r', 'utf8')
        for line in f:
            i += 1
            #print(line)
            #separate line into fields
            fields = line.split(';')
            userid = fields[0].strip('"')
            location = fields[1].strip('"')
            if len(fields) > 3:
                age = fields[2].strip().strip('"')
            else:
                age = 'NULL'
            if age != 'NULL':
                value = location + '  (age: ' + age + ')'
            else:
                value = location
            self.userid2name[userid] = value
            self.username2id[location] = userid
        f.close()
        print(i)

    def pearson(self, rating1, rating2):
        sum_xy = 0
        sum_x = 0
        sum_y = 0
        sum_x2 = 0
        sum_y2 = 0
        n = 0
        for key in rating1:
            if key in rating2:
                n += 1
                x = rating1[key]
                y = rating2[key]
                sum_xy += x * y
                sum_x += x
                sum_y += y
                sum_x2 += pow(x, 2)
                sum_y2 += pow(y, 2)
        if n == 0:
            return 0
        # 计算分母
        denominator = (sqrt(sum_x2 - pow(sum_x, 2) / n)
                       * sqrt(sum_y2 - pow(sum_y, 2) / n))
        if denominator == 0:
            return 0
        else:
            return (sum_xy - (sum_x * sum_y) / n) / denominator

    def computeNearestNeighbor(self, username):
        """获取邻近用户"""
        distances = []
        for instance in self.data:
            if instance != username:
                distance = self.fn(self.data[username],
                                   self.data[instance])
                distances.append((instance, distance))
        # 按距离排序，距离近的排在前面
        distances.sort(key=lambda artistTuple: artistTuple[1],
                       reverse=True)
        return distances

    def recommend(self, user):
       """返回推荐列表"""
       recommendations = {}
       # 首先，获取邻近用户
       nearest = self.computeNearestNeighbor(user)
       #
       # 获取用户评价过的商品
       #
       userRatings = self.data[user]
       #
       # 计算总距离
       totalDistance = 0.0
       for i in range(self.k):
          totalDistance += nearest[i][1]
       # 汇总K邻近用户的评分
       for i in range(self.k):
          # 计算饼图的每个分片
          weight = nearest[i][1] / totalDistance
          # 获取用户名称
          name = nearest[i][0]
          # 获取用户评分
          neighborRatings = self.data[name]
          # 获得没有评价过的商品
          for artist in neighborRatings:
             if not artist in userRatings:
                if artist not in recommendations:
                   recommendations[artist] = (neighborRatings[artist]
                                              * weight)
                else:
                   recommendations[artist] = (recommendations[artist]
                                              + neighborRatings[artist]
                                              * weight)
       # 开始推荐
       recommendations = list(recommendations.items())
       recommendations = [(self.convertProductID2name(k), v)
                          for (k, v) in recommendations]
       # 排序并返回
       recommendations.sort(key=lambda artistTuple: artistTuple[1],
                            reverse = True)
       # 返回前n个结果
       return recommendations[:self.n]
```

**运行示例**

首先构建一个推荐类，然后获取推荐结果：

```python
>>> r = recommender(users)
>>> r.recommend('Jordyn')
[('Blues Traveler', 5.0)]
>>> r.recommend('Hailey')
[('Phoenix', 5.0), ('Slightly Stoopid', 4.5)]
```

### 新的数据集

现在让我们使用一个更为真实的数据集。Cai-Nicolas Zeigler从图书漂流站收集了超过100万条评价数据——278,858位用户为271,379本书打了分。这份数据（匿名）可以从[这个地址](http://www.informatik.uni-freiburg.de/~cziegler/BX/)获得，有SQL和CSV两种格式。由于特殊符号的关系，这些数据无法直接加载到Python里。我做了一些清洗，可以从[这里下载](http://guidetodatamining.com/guide/ch2/BX-Dump.zip)。

CSV文件包含了三张表：

* 用户表，包括用户ID、位置、年龄等信息。其中用户的姓名已经隐去；
* 书籍表，包括ISBN号、标题、作者、出版日期、出版社等；
* 评分表，包括用户ID、书籍ISBN号、以及评分（0-10分）。

上文Python代码中的loadBookDB方法可以加载这些数据，用法如下：

```python
>>> r.loadBookDB('/Users/raz/Downloads/BX-Dump/')
1700018
>>> r.recommend('171118')
```

**注意** 由于数据集比较大，大约需要几十秒的时间加载和查询。

### 项目实践

只有运行调试过书中的代码后才能真正掌握这些方法，以下是一些实践建议：

1. 实现一个计算曼哈顿距离和欧几里得距离的方法；
2. 本书的网站上有一个包含25部电影评价的[数据集](http://guidetodatamining.com/guide/ch2/Movie_Ratings.csv)，实现一个推荐算法。
