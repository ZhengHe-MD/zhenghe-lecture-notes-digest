# Compound Indexes

MongoDB 支持对 document 的多个 fields 建立索引，这种索引叫作 compound index。Compound Index 可以用来优化多种查询、排序操作。

#### 引入：学生记录举例

假设我们一个学生 \(students\) collection，里面的每个 document 结构如下：

```json
{
    "_id": "some object id",
    "name": "zhangsan",
    "major": "economics",
    "age": 15
}
```

如果这个 collection 主要接受的查询只涉及一个 field, 如：

```js
> db.students.find({ name: "zhangsan" })
> db.students.find({ age: { $gte: 15 } })
> db.students.find({ major: "economics" })
> db.students.find({ age: { $gte: 15 } }).sort({ age: 1 })
```

那么我们建立 single-field index 就可以解决问题。但如果常用查询包含多个字段，如：

```js
> db.students.find({ major: "economics", age: { $gte: 15 } })
> db.students.find({ major: "math", age: { $lte: 17 } }).sort({ age: 1 })
```

如果数据量很且分布较开，这时候就可以通过建立 compound index 来提高查询效率：

```js
> db.students.createIndex({ major: 1, age: 1 })
```

具体有关 compound index 的一些细节，查阅它的[官方文档](https://docs.mongodb.com/manual/core/index-compound/)即可，这里不再赘述。

#### Inside the Compound Index

绝大多数 MongoDB 中的 Indexes 的背后都是 B-tree。single-field index 的结构大致如下：

![](/assets/Screen Shot 2018-04-03 at 10.32.08 AM.jpg)

而 compound index 的结构大致如下：

![](/assets/Screen Shot 2018-04-03 at 10.32.14 AM.jpg)

##### 适合的查询

**prefixes**

假设建立如下 compound index:

```js
> db.students.createIndex({ name: 1, major: 1, age: 1 })
```

那么只要是符合 name -&gt; major -&gt; age 顺序，且包含第一个 field --- name 的任意查询，如：

```js
> db.students.find({ name: "zhangsan" })
> db.students.find({ name: "zhangsan", major: "math" })
> db.students.find({ name: "zhangsan", age: 1 })
> db.students.find({ name: "zhangsan", major: "math", age: 1 })
```

但一旦没有第一个 field，则 index 失效，从 compound index 的结构图上不难理解，第一个 field 对应的每个叶子节点 \(leaf node\) 都可以包含有后面的 fields 的所有可能值，此时使用 compound index 与直接扫描 collection 的效率基本持平。

**sort**

假设建立如下 compound index:

```js
> db.students.createIndex({ major: 1, age: -1 })
```

这时候，以下排序查询可以利用到 compound index:

```js
> db.students.find().sort({ major: 1, age: -1 })
> db.students.find().sort({ major: -1, age: 1 })
```

但不符合顺序的排序查询则无法利用到 compound index:

```js
> db.students.find().sort({ major: 1, age: 1 })
> db.students.find().sort({ major: -1, age: -1 })
```

从 compound index 的结构图上不难理解，顺序访问 leaf node 可以得到符合 **{ major: 1, age: -1 }** 的查询；逆序访问 leaf node 可以得到符合 **{ major: -1, age: 1 }** 的查询；要完成剩余两种查询则无法利用 compound index 得到。

### 参考

* [MongoDB: compound index](https://docs.mongodb.com/manual/core/index-compound/)
* [Concatenated Index](https://use-the-index-luke.com/sql/where-clause/the-equals-operator/concatenated-keys)
* [Optimizing MongoDB Compound Indexes](https://emptysqua.re/blog/optimizing-mongodb-compound-indexes/)



