> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/357414033)

导语
--

说起相似度检索 TopK 的问题，相信很多算法 er 在实际工程中会经常遇到，对此我们一般的解决方案是暴力检索，循环遍历所有向量计算相似度然后得出 TopK。但当向量数量级达到百万千万甚至上亿级别，这时候你再用暴力检索就会显得很呆 ... ...

![](https://pic4.zhimg.com/v2-2f0ca1d575cc08101954e008ee755515_1440w.jpg)

Faiss 的出现就很好地解决了这个问题，笔者总结了在工程中使用 Faiss 的一些经验，记录下给需要的童鞋（语言为 Python，因为本菜鸡不会 C++）。动动小手给点个赞呗。

1. 什么是 Faiss？
-------------

Faiss 的全称是 [Facebook AI Similarity Search](https://link.zhihu.com/?target=https%3A//github.com/facebookresearch/faiss)，是 FaceBook 的 AI 团队针对大规模相似度检索问题开发的一个工具，使用 C++ 编写，有 python 接口，**对 10 亿量级的索引可以做到毫秒级检索的性能**。

简单来说，**Faiss 的工作，就是把我们自己的候选向量集封装成一个 index 数据库，它可以加速我们检索相似向量 TopK 的过程，其中有些索引还支持 GPU 构建，可谓是强上加强。**

![](https://pica.zhimg.com/v2-553c0b75a11b60bf6a682412d451bcac_b.jpg)

### 1.1 Faiss 的安装

强烈建议安装 faiss 1.7.3 以上版本，新版本已经没有了老版本难以安装各种 bug 层出不穷的风险，只需要简单的 [pip](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=pip&zhida_source=entity) 安装即可：

```
pip install faiss-cpu==1.7.3
# pip install faiss-gpu==1.7.2  # gpu安装，绝大部分情况下cpu已经能hold住了，后续会专门写关于faiss-gpu的使用

```

而且新版本也加强了对 windows 的支持，妈妈再也不用担心我的 faiss 抛出各种 swigfaiss 错误了~

2. Faiss 简单上手
-------------

首先，Faiss 检索相似向量 TopK 的工程基本都能分为三步：

1.  得到向量库；
2.  用 faiss 构建 index，并将向量添加到 index 中；
3.  用 [faiss index](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=faiss+index&zhida_source=entity) 检索。

好吧...... 这貌似和废话没啥区别，参考把大象装冰箱需要几个步骤。本段代码摘自 Faiss 官方文档，很清晰，**基本所有的 index 构建流程都遵循这个步骤**。

*   第一步，得到向量：

```
import numpy as np
d = 64                                           # 向量维度
nb = 100000                                      # index向量库的数据量
nq = 10000                                       # 待检索query的数目
np.random.seed(1234)             
xb = np.random.random((nb, d)).astype('float32')
xb[:, 0] += np.arange(nb) / 1000.                # index向量库的向量
xq = np.random.random((nq, d)).astype('float32')
xq[:, 0] += np.arange(nq) / 1000.                # 待检索的query向量

```

*   第二步，构建索引，这里我们选用暴力检索的方法 FlatL2，L2 代表构建的 index 采用的相似度度量方法为 L2 范数，即[欧氏距离](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E6%AC%A7%E6%B0%8F%E8%B7%9D%E7%A6%BB&zhida_source=entity)：

```
import faiss          
index = faiss.IndexFlatL2(d)             
print(index.is_trained)         # 输出为True，代表该类index不需要训练，只需要add向量进去即可
index.add(xb)                   # 将向量库中的向量加入到index中
print(index.ntotal)             # 输出index中包含的向量总数，为100000 

```

*   第三步，检索 TopK 相似 query：

```
k = 4                     # topK的K值
D, I = index.search(xq, k)# xq为待检索向量，返回的I为每个待检索query最相似TopK的索引list，D为其对应的距离
print(I[:5])
print(D[-5:])

```

*   打印输出为：

```
>>> 
[[  0 393 363  78] 
 [  1 555 277 364] 
 [  2 304 101  13] 
 [  3 173  18 182] 
 [  4 288 370 531]]  
[[ 0.          7.17517328  7.2076292   7.25116253]  
 [ 0.          6.32356453  6.6845808   6.79994535]  
 [ 0.          5.79640865  6.39173603  7.28151226]  
 [ 0.          7.27790546  7.52798653  7.66284657]  
 [ 0.          6.76380348  7.29512024  7.36881447]]

```

### **2.1 夹带私货，Faiss 检索小工具**

写了一个简单的基于 pandas DataFrame 的 Faiss 类，用起来还是蛮方便的，小伙伴们如果对 Faiss 有一定了解可以很方便的使用：

[GitHub - mechsihao/FaissSearcher: a common faiss searcher](https://link.zhihu.com/?target=https%3A//github.com/mechsihao/FaissSearcher)

3. Faiss 常用 index 优缺点及使用场景
--------------------------

Faiss 之所以能加速，是因为它用的检索方式并非精确检索，而是模糊检索。既然是模糊检索，那么必定有所损失，我们用**召回率来表示模糊检索相对于精确检索的损失**。

在我们实际的工程中，候选向量的数量级、index 所占内存的大小、检索所需时间（是离线检索还是在线检索）、index 构建时间、检索的召回率等都是我们选择 index 时常常需要考虑的地方。

首先，我建议关于 Faiss 的所有索引的构建，**都统一使用 faiss.index_factory**，基本所有的 index 都支持这种构建索引方法。

以第二章的代码为例，构建 [index 方法](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=index%E6%96%B9%E6%B3%95&zhida_source=entity)和传参方法建议修改为：

```
dim, measure = 64, faiss.METRIC_L2
param = 'Flat'
index = faiss.index_factory(dim, param, measure)

```

*   三个参数中，dim 为向量维数；
*   **最重要的是 [param 参数](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=param%E5%8F%82%E6%95%B0&zhida_source=entity)，它是传入 index 的参数，代表需要构建什么类型的索引；**
*   measure 为度量方法，目前支持两种，欧氏距离和 [inner product](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=inner+product&zhida_source=entity)，即内积。**因此，要计算[余弦相似度](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E4%BD%99%E5%BC%A6%E7%9B%B8%E4%BC%BC%E5%BA%A6&zhida_source=entity)，只需要将 vecs 归一化后，使用内积度量即可。**
*   20220102 更新，现在 faiss 官方支持八种度量方式，分别是：

*   METRIC_INNER_PRODUCT（内积）
*   METRIC_L1（[曼哈顿距离](https://link.zhihu.com/?target=https%3A//blog.csdn.net/hy592070616/article/details/121569933%3Fspm%3D1001.2014.3001.5501)）
*   METRIC_L2（欧氏距离）
*   METRIC_Linf（[无穷范数](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E6%97%A0%E7%A9%B7%E8%8C%83%E6%95%B0&zhida_source=entity)）
*   METRIC_Lp（p 范数）
*   METRIC_BrayCurtis（[BC 相异度](https://zhuanlan.zhihu.com/p/440130486?utm_id=0)）
*   METRIC_Canberra（[兰氏距离 / 堪培拉距离](https://link.zhihu.com/?target=https%3A//blog.csdn.net/hy592070616/article/details/122271656)）
*   METRIC_JensenShannon（[JS 散度](https://link.zhihu.com/?target=https%3A//blog.csdn.net/hy592070616/article/details/122387046%3Fspm%3D1001.2014.3001.5501)）

接下来笔者将介绍几种最核心的 index 类型的用法及优缺点，当然 faiss 支持的 index 类型非常多，但是以下这些 index 属于 faiss 最核心的几种基本 index，大部分其他 index 是在这些核心 index 思想上的扩展、补充和改进，比如在 PQ 思想基础上的改进有 SQ、OPQ、LOPQ，基于 LSH 的改进有 ALSH 等等，使用方法和下面介绍的类似。

**3.1 Flat ：暴力检索**

*   优点：该方法是 Faiss 所有 index 中最准确的，召回率最高的方法，没有之一；
*   缺点：速度慢，占内存大。
*   使用情况：向量候选集很少，在 50 万以内，并且内存不紧张。
*   注：虽然都是暴力检索，**faiss 的暴力检索速度比一般程序猿自己写的暴力检索要快上不少**，所以并不代表其无用武之地，建议有暴力检索需求的同学还是用下 faiss。
*   构建方法：

```
dim, measure = 64, faiss.METRIC_L2
param = 'Flat'
index = faiss.index_factory(dim, param, measure)
index.is_trained                                   # 输出为True
index.add(xb)                                      # 向index中添加向量

```

**3.2 IVFx Flat ：倒排暴力检索**

*   优点：IVF 主要利用倒排的思想，在文档检索场景下的[倒排技术](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E5%80%92%E6%8E%92%E6%8A%80%E6%9C%AF&zhida_source=entity)是指，一个 kw 后面挂上很多个包含该词的 doc，由于 kw 数量远远小于 doc，因此会大大减少了检索的时间。在向量中如何使用倒排呢？可以拿出每个聚类中心下的向量 ID，每个中心 ID 后面挂上一堆[非中心向量](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E9%9D%9E%E4%B8%AD%E5%BF%83%E5%90%91%E9%87%8F&zhida_source=entity)，每次查询向量的时候找到最近的几个中心 ID，分别搜索这几个中心下的非中心向量。通过减小搜索范围，提升搜索效率。
*   缺点：速度也还不是很快。
*   使用情况：相比 Flat 会大大增加检索的速度，建议百万级别向量可以使用。
*   参数：IVFx 中的 x 是 k-means 聚类中心的个数
*   构建方法：

```
dim, measure = 64, faiss.METRIC_L2 
param = 'IVF100,Flat'                           # 代表k-means聚类中心为100,   
index = faiss.index_factory(dim, param, measure)
print(index.is_trained)                          # 此时输出为False，因为倒排索引需要训练k-means，
index.train(xb)                                  # 因此需要先训练index，再add向量
index.add(xb)                                     

```

**3.3 PQx ：乘积量化**

*   优点：利用乘积量化的方法，改进了普通检索，将一个向量的维度切成 x 段，每段分别进行检索，每段向量的检索结果取交集后得出最后的 TopK。因为 PQ 算法把原来的[向量空间](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E5%90%91%E9%87%8F%E7%A9%BA%E9%97%B4&zhida_source=entity)分解为若干个低维向量空间的[笛卡尔积](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E7%AC%9B%E5%8D%A1%E5%B0%94%E7%A7%AF&zhida_source=entity)，并对分解得到的低维向量空间分别做量化（quantization）。这样每个向量就能由多个低维空间的量化 code 组合表示，因此速度很快，而且占用内存较小，召回效果也还可以。
*   缺点：召回率相较于暴力检索，下降较多。
*   使用情况：内存及其稀缺，并且需要较快的检索速度，不那么在意召回率
*   参数：PQx 中的 x 为将向量切分的段数，因此，**x 需要能被向量维度整除**，且 x 越大，切分越细致，[时间复杂度](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6&zhida_source=entity)越高
*   构建方法：

```
dim, measure = 64, faiss.METRIC_L2 
param =  'PQ16' 
index = faiss.index_factory(dim, param, measure)
print(index.is_trained)                          # 此时输出为False，因为倒排索引需要训练k-means，
index.train(xb)                                  # 因此需要先训练index，再add向量
index.add(xb)          

```

**3.4 IVFxPQy [倒排乘积](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E5%80%92%E6%8E%92%E4%B9%98%E7%A7%AF&zhida_source=entity)量化**

*   优点：工业界大量使用此方法，各项指标都均可以接受，利用乘积量化的方法，改进了 IVF 的 k-means，将一个向量的维度切成 x 段，每段分别进行 k-means 再检索。
*   缺点：集百家之长，自然也集百家之短
*   使用情况：一般来说，超大规模数据的情况下，各方面没啥特殊的极端要求的话，**最推荐使用该方法！**
*   参数：IVFx，PQy，其中的 x 和 y 同上
*   构建方法：

```
dim, measure = 64, faiss.METRIC_L2  
param =  'IVF100,PQ16'
index = faiss.index_factory(dim, param, measure) 
print(index.is_trained)                          # 此时输出为False，因为倒排索引需要训练k-means， 
index.train(xb)                                  # 因此需要先训练index，再add向量 index.add(xb)       

```

**3.5 LSH 局部敏感[哈希](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E5%93%88%E5%B8%8C&zhida_source=entity)**

*   原理：哈希对大家再熟悉不过，向量也可以采用哈希来加速查找，我们这里说的哈希指的是局部敏感哈希（Locality Sensitive Hashing，LSH），不同于传统哈希尽量不产生碰撞，局部敏感哈希依赖碰撞来查找近邻。[高维空间](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E9%AB%98%E7%BB%B4%E7%A9%BA%E9%97%B4&zhida_source=entity)的两点若距离很近，那么设计一种[哈希函数](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E5%93%88%E5%B8%8C%E5%87%BD%E6%95%B0&zhida_source=entity)对这两点进行哈希计算后分桶，使得他们哈希分桶值有很大的概率是一样的，若两点之间的距离较远，则他们哈希分桶值相同的概率会很小。
*   优点：训练非常快，支持分批导入，index 占内存很小，检索也比较快
*   缺点：召回率非常拉垮。
*   使用情况：候选向量库非常大，离线检索，内存资源比较稀缺的情况
*   构建方法：

```
dim, measure = 64, faiss.METRIC_L2  
param =  'LSH'
index = faiss.index_factory(dim, param, measure) 
print(index.is_trained)                          # 此时输出为True
index.add(xb)       

```

**3.6 HNSWx （最重要的放在最后说）**

*   优点：该方法为基于图检索的改进方法，检索速度极快，10 亿级别秒出检索结果，而且召回率几乎可以媲美 Flat，最高能达到惊人的 97%。检索的时间复杂度为 loglogn，几乎可以无视候选向量的量级了。并且支持分批导入，极其适合线上任务，毫秒级别体验。
*   缺点：构建索引极慢，占用内存极大（是 Faiss 中最大的，大于原向量占用的内存大小）
*   参数：HNSWx 中的 x 为构建图时每个点最多连接多少个节点，x 越大，构图越复杂，查询越精确，当然构建 index 时间也就越慢，x 取 4~64 中的任何一个整数。
*   使用情况：不在乎内存，并且有充裕的时间来构建 index
*   构建方法：

```
dim, measure = 64, faiss.METRIC_L2   
param =  'HNSW64' 
index = faiss.index_factory(dim, param, measure)  
print(index.is_trained)                          # 此时输出为True 
index.add(xb)

```

4. 使用 Faiss 时踩过的坑和一些经验
----------------------

**4.1 以下是我的同事对不同 index 做过的一些[对比实验](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E5%AF%B9%E6%AF%94%E5%AE%9E%E9%AA%8C&zhida_source=entity)结果：**

![](https://pica.zhimg.com/v2-ee956dee3cb177af3281fba6ee5e1b86_r.jpg)![](https://pica.zhimg.com/v2-b9e534c3995604e4d3ea5e077d0c37de_r.jpg)

**4.2 Faiss 可以组合传参**

Faiss 内部支持先将向量 PCA 降维后再构建 index，param 设置如下：

```
param = 'PCA32,IVF100,PQ16'

```

代表将向量先降维成 32 维，再用 IVF100 PQ16 的方法构建索引。

同理可以使用：

```
param = 'PCA32,HNSW32'

```

可以用来处理 HNSW 内存占用过大的问题。

**4.3 Faiss 在构建索引时，有时生成的 vecs 会很大，向 index 中添加的向量很有可能无法一次性放入内存中，怎么办呢？**

*   这时候，索引的可分批导入 index 的性质就起了大作用了；
*   如何来知道一种 index 是否可以分批 add 呢？一般来说在未对 index 进行 train 操作前，如果一个 index.is_trained = True，那么它就是可以分批 add 的；
*   如果是 index.is_trained = False，就不能分批导入，当然，其实强行对 index.is_trained = False 的索引分批 add 向量是不会报错的，只不过内部构建索引的逻辑和模型都在第一个 batch 数据的基础上构建的，比如 PCA 降维，其实是拿第一个 batch 训练了一个 PCA 模型，后续 add 进入的新向量都用这个模型降维，这样会**导致后续 batch 的向量失真，影响精度，当然如果不在意这点精度损失那也可以直接 add；**
*   **由上可得，只要一个 index 的 param 中包含 PCA，那他就不能分批 add；**
*   **可以分批导入的 index 为：HNSW、Flat、LSH。**

**4.4 如果我们的需求，既想 PCA 降维减小 index 占用内存，还想分批 add 向量，该怎么办？**

可以使用 sklean 中的增量 pca 方法，先把数据降维成低维的，再将降维后的向量分批 add 进索引中，增量 pca 使用方法和 pca 一致：

```
from sklearn.decomposition import IncrementalPCA

```

**4.5 关于 HNSW**

HNSW 虽好，可不要贪杯哦，这里坑还是蛮多的：

*   HNSW 的构建过程有时很短，有时却又很长，500 万量级的向量快时可以 15 分钟构建完毕，慢则有可能花费两小时，这和 HNSW 构建多层图结构时的[生成函数](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E7%94%9F%E6%88%90%E5%87%BD%E6%95%B0&zhida_source=entity)有关。
*   老版本 faiss 的 HNSW 在使用以下这种构建索引方式时，有一个 bug，这个 bug 会导致如果 measure 选用内积的方法度量，但最后构建的索引的度量方式仍然是欧氏距离：

```
index = faiss.index_factory(dim, param, measure)  

```

如果想构建以内积（余弦相似度）为基准的 HNSW 索引，可以这样构建：

```
index = faiss.IndexHNSWFlat(dim, x,measure)  # measure 选为内积，x为4~64之间的整数

```

*   所以直接建议使用新版本 faiss，版本最好 > 1.7，无此 bug。
*   HNSW 占用内存真的很大，500 万条 768 维的向量构建的索引占用内存 17G，而同样数据下 LSH 仅占用 500M，emmm 所以自行体会吧。
*   HNSW 的检索候选可能不会很多，笔者的经验是一般 500w 的候选集，用 HNSW64 构建索引，检索 top1000 的时候就开始出现尾部重复现象，这其实就是 HNSW 在他的构建图中搜索不到近邻向量了，所以最后会用一个重复的 id 将尾部 padding，让输出 list 补满至 1000 个，虽然 IVF、PQ 都会出现这个问题，但是 HNSW 会特别明显，这个和算法本身的原理有关。

**4.6 Faiss 所有的 index 仅支持[浮点数](https://zhida.zhihu.com/search?content_id=167555093&content_type=Article&match_order=1&q=%E6%B5%AE%E7%82%B9%E6%95%B0&zhida_source=entity)为 float32 格式**

Faiss 仅支持浮点数为 np.float32 格式，其余一律不支持，所以用 Faiss 前需要将向量数据转化为 float32，否则会报错！**这也告诉大家，想用降低精度来实现降低 index 内存占用是不可能的！**

**5.Faiss 构建索引时，如何权衡数据量级和内存占用来选择 index 呢？**
-------------------------------------------

这里有一份 Faiss 的官方文档，里面写的很清楚：

[Faiss 官网，如何选择 index 类型](https://link.zhihu.com/?target=https%3A//github.com/facebookresearch/faiss/wiki/Guidelines-to-choose-an-index)

关于 Faiss 中 index 的更多详细信息，请参考官网：

[Faiss index factory 介绍](https://link.zhihu.com/?target=https%3A//github.com/facebookresearch/faiss/wiki/The-index-factory)![](https://pic1.zhimg.com/v2-977e033895dcf390cc234f9ba7aa3c92_b.jpg)

点个关注吧，ball ball 兄弟们了，能不能过千粉就看大家了。

都看到这儿啦，求赞、求喜欢、求收藏，快三连～