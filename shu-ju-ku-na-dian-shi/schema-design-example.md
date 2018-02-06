# MongoDB Schema Design Example \(电商网站\)

在学习传统的关系型数据库 \(Relational Database, RDB\) 的数据模型设计 \(Schema Design\) 时，课本通常强调规范化 \(Normalize\) 数据模型，减少甚至消除冗余数据；然而在实践中，我们会通过适当增加数据冗余 \(Denormalize\)，利用空间换取时间，提高数据库效率。简而言之，在关系型数据库领域，使用数据冗余通常有种野路子的感觉。而在 MongoDB 这种基于文档 \(Document-based\) 的数据库中设计数据模型时，合理使用冗余数据是一种正统做法。本文用例来自于 **MongoDB In Action** 中，希望通过这篇文章可以让你感受到：

1. MongoDB 为什么适用于网络应用的数据模型设计
2. Denormalization 在 MongoDB 模型设计中的应用

### 设计需求

#### 实体及其关系

一个电商网站，主要包含**商品**、**商品类别**、**用户**、**用户反馈**、**订单**等五个实体，它们的关系如下图所示：

!\[image-1\]\(placeholder\)

#### 需求举例

##### 商品需求1：每种类型的商品有些自定义的属性

```js
// product 部分属性
{
  details: {
    weight: 47,
    weight_units: "lbs",
    model_num: 4039283402,
    manufacturer: "Acme",
    color: "Green"
  }
}
```

这个需求的难点在于，不同的商品，它的自定义属性不同。如相机镜头有 f-stop 范围、焦距、最小对焦距离等属性，计算机有 CPU、显卡、内存储、外存储等属性，这些属性的特点是非常分散且动态变化。这在关系型数据库中令人十分头疼，需要通过 Entity-Attribute-Value \(EAV\) 等精心设计却有些笨拙的方式来实现。而在 MongoDB 中，只需要一个文档就可以解决，这来源于 MongoDB schema-less 的特性，同一个 collection 中的不同 documents 可以拥有不同的属性。

##### 商品需求2：每个商品需要记录自己的历史价格

```js
// product 部分属性
{
  pricing: {
    retail: 589700,
    sale: 429700
  },
  price_history: [
    {
      retail: 529700,
      sale: 429700,
      start: new Date(2010, 4, 1),
      end: new Date(2010, 4, 8)
    },
    {
      retail: 529700,
      sale: 529700,
      start: new Date(2010, 4, 9),
      end: new Date(2010, 4, 16)
    }
  ]
}
```

要记录商品的历史价格，对于关系型数据库来说并非是一件头疼的事情，仅需将 price 规范化成一个实体，与 price 有关的信息如商品、零售价、销售价、时间区间等信息都放入其中即可。但这种方式没有考虑到两个事实

1. 商品信息与历史价格信息通常是一同获取的。
2. 获取商品及其历史价格信息需要一次 join 操作，而该操作在大表上有一定性能开销

在 MongoDB 上，这种问题可以通过去规范化来解决，既然商品信息与历史价格信息是息息相关的，就把它们放到一个 document 中，同时省略 join 操作的开销。

##### 商品需求3：每个商品都属于某个主类别，同时属于主类的所有祖先 \(ancestors\) 类别







