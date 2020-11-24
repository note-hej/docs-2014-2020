title: 词向量和主题模型的应用场景
date: 2016-09-18
tags: [NLP,词向量,主题模型]
---
本文主要讨论word2vec、doc2vec和topicModel的典型应用场景。

<!--more-->
## 词向量

### 社交网络推荐
一个个性化推荐的场景，给当前用户推荐他可能关注的『大V』。对一个新用户，此题基本无解，如果在已知用户关注了几个『大V』之后，相当于知道了当前用户的一些关注偏好，根据此偏好给他推荐和他关注过『大V』相似的『大V』，就是一个很不错的推荐策略。所以，如果可以求出来任何两个『大V』的相似度，上面问题就可以基本得到解决。

我们知道word2vec中两个词的相似度可以直接通过余弦来衡量，接下来就是如何将每个『大V』变为一个词向量的问题了。巧妙的地方就是如何定义doc和word，针对上面问题，可以将doc和word定义为：

    word ->   每一个『大V』就是一个词
    doc  ->   根据每一个用户关注『大V』的顺序，生成一篇文章

### 计算商品的相似度
在商品推荐的场景中，竞品推荐和搭配推荐的时候都有可能需要计算任何两个商品的相似度，根据浏览/收藏/下单/App下载等行为，可以将商品看做词，将每一个用户的一类行为序看做一个文档，通过word2vec将其训练为一个向量。

同样的，在计算广告中，根据用户的点击广告的点击序列，将每一个广告变为一个向量。变为向量后，用此向量可以生成特征融入到rank模型中。

### 作为另一个模型的输入
在nlp的任务中，可以通过将词聚类后，生成一维新的特征来使用。在CRF实体识别的任务中，聚类结果类似词性，可以作为特征来使用。在依存句法分析的任务中，哈工大ltp的[A Fast and Accurate Dependency Parser using Neural Networks](http://cs.stanford.edu/people/danqi/papers/emnlp2014.pdf)则是将词向量直接作为输入。

## 主题模型

### 短文本相关性
在自然语言处理和信息检索中，我们常常会遇到如下问题：给定查询词，计算查询词和文档之间的相关性。常用的计算方法就是不考虑词的相对顺序，使用BOW（Bag-Of-Words）模型把文档表示为词向量，然后计算文本之间的相似度。如果直接采用文档中词的TF-IDF构建文档特征向量，通过计算查询词特征向量和文档特征向量的余弦夹角，我们会发现Q1与D1、D2都相关，而Q2与D1、D2都不相关。显然，这与人对自然语言的理解并不相符：Q1和D2比较相关，都关于“苹果”这种水果；而Q2和D1比较相关，都关于“苹果”公司。

    Q1(苹果：水果) apple pie
    Q2(苹果：公司) iphone crack
    Q1(苹果：水果) apple computer inc. is a well know company located in california
    Q1(苹果：公司) the apple is the pomaceous fruit of the apple tree

之所以会出现这种差异，是因为上述文档特征向量构建方法没有“理解”文档的具体语义信息，单纯的将文档中的词表示为一个ID而已。通过主题模型，文档可以表示为一个隐含语义空间上的概率分布向量（主题向量），文档主题向量之间的余弦夹角就可以一定程度上反映文档间的语义相似度了。

### 推荐系统
主题模型的另一个主要应用场景是推荐系统。不管是电商网站的商品推荐，还是各大视频网站的视频推荐等，都可以简化为如下问题：给定用户-物品矩阵，根据用户行为数据，矩阵会得到部分“初始”值，如何“填满”矩阵中没有值的部分。

在各种眼花缭乱的推荐算法中，直接利用用户-物品矩阵进行推荐是最有效的方式（没有长年的用户、物品内容分析技术积累也一样可以快速做出效果），而这其中的两类主要算法都与主题模型有关系：

- 协同过滤。以基于用户的协同过滤为例，就是要向用户推荐与之相似的用户喜欢的物品，包含两个主要步骤：计算用户相似度和向用户推荐与自己最相似的用户喜欢的物品，难点在于计算用户相似度。如果不引入外部数据，最简单的计算用户u和v相似度的方法可以直接利用用户-物品矩阵的u行和v行，比如计算它们的余弦夹角。然而，真实的互联网数据中，用户-物品矩阵通常都非常稀疏，直接计算不能得到准确的结果。此时，常见的做法是对用户（或物品）进行聚类或者将矩阵投影到更低维的隐空间，在隐空间计算用户相似度可以更加准确。主题模型可以用来将用户-物品矩阵投影到隐空间。
- 隐含语义模型(Latent Factor Model, LFM)。该类方法本质上和主题模型是一致的，直观的理解是将用户-物品矩阵分解为用户-隐含语义（主题）矩阵和隐含语义（主题）-物品矩阵，通过更低维度的上述两个矩阵，来重构原始用户-物品矩阵，重构得到的矩阵将不再稀疏，可以直接用于推荐。

实际上，从以上的讨论中我们容易发现，当使用BOW模型处理文本，把文档数据表示成文档-词（Doc-Word）矩阵的时候，其表示结构和用户-物品（User-Item）矩阵结构是完全一致的。因此这两类数据可以使用同样的算法进行处理。使用隐含主题模型处理文档-词矩阵的时候，可以理解为把词聚类为主题，并计算各个文档和词聚类之间的权重。类似地，处理用户-物品矩阵的时候，可以理解为把物品聚类为主题，然后计算每个用户和各个聚类之间的权重。

## 参考资料：
- [word2vec在工业界的应用场景](http://x-algo.cn/index.php/2016/03/12/281/)
- [大规模主题模型及其在腾讯业务中的应用](http://www.flickering.cn/nlp/2015/03/Peacock：大规模主题模型及其在腾讯业务中的应用/)