

#	day01 基本入门

##	概念

* database: 数据库，对应于mysql中的数据库
* collection： 集合，对应于mysql中的表
* document： 文档，对应于mysql中表记录的row
* filed： 字段， 对应于Mysql中表的列
* index： 索引， 对应于mysql中的索引
* mongodb不支持表关联

##	数据库

* 查看数据库

  ```
  show dbs
  ```

* 查看当前数据库

  ```
  db
  ```

* 切换当前连接的数据库

  ```
  use 数据库名
  ```

* 数据库命名规范

  * 不能是空字符串（"")。
  * 不得含有' '（空格)、.、$、/、\和\0 (空字符)。
  * 应全部小写。
  * 最多64字节。

* 系统保留的数据库

  * **admin**： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
  * **local:** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意collection
  * **config**: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

## 文档Document

​	文档是一组键值(key-value)对(即 BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

​	一个简单的文档如下：

```json
{ "name":"jack", "age": 12}

```

   * 文档中的键/值对是有序的

   * 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。

*	MongoDB区分类型和大小写。

*	MongoDB的文档不能有重复的键。

*	文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

  文档键命名规范：

  * 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
  * .和$有特别的意义，只有在特定环境下才能使用。
  * 以下划线"_"开头的键是保留的(不是严格要求的)。

## 集合collection

集合是文档的一个集， 类似于关系型数据库中的表。可以将多个Document插入到一个集合中。同一个集合中

例如：

```
{"site":"www.baidu.com"}
{"site":"www.google.com","name":"Google"}
{"site":"www.runoob.com","name":"菜鸟教程","num":5}
```

* 集合的命名
  * 集合名不能是空字符串""。
  * 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。
  * 集合名不能以"system."开头，这是为系统集合保留的前缀。
  * 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。

###	capped collection

* capped collections 就是固定大小的collection。
* Capped collections 是高性能自动的维护对象的插入顺序。它非常适合类似记录日志的功能和标准的 collection 不同，你必须要显式的创建一个capped collection，指定一个 collection 的大小，单位是字节。collection 的数据存储空间值提前分配的。
* Capped collections 可以按照文档的插入顺序保存到集合中，而且这些文档在磁盘上存放位置也是按照插入顺序来保存的，所以当我们更新Capped collections 中文档的时候，更新后的文档不可以超过之前文档的大小，这样话就可以确保所有文档在磁盘上的位置一直保持不变。
* 由于 Capped collection 是按照文档的插入顺序而不是使用索引确定插入位置，这样的话可以提高增添数据的效率。MongoDB 的操作日志文件 oplog.rs 就是利用 Capped Collection 来实现的。
* 在 capped collection 中，你能添加新的对象。
* 能进行更新，然而，对象不会增加存储空间。如果增加，更新就会失败 。
* 使用 Capped Collection 不能删除一个文档，可以使用 drop() 方法删除 collection 所有的行。
* 删除之后，你必须显式的重新创建这个 collection。
* 在32bit机器中，capped collection 最大存储为 1e9( 1X109)个字节。

###	创建集合

* 创建集合的指令

  ```json
  db.createCollection("mycollection", {capped:true, size:100000})
  ```

##	查看系统元数据

数据库的信息是存储在集合中。它们使用了系统的命名空间：

```
dbname.system.*	
```

在MongoDB数据库中名字空间 <dbname>.system.* 是包含多种系统信息的特殊集合(Collection)，如下:

| 集合命名空间             | 描述                                      |
| :----------------------- | :---------------------------------------- |
| dbname.system.namespaces | 列出所有名字空间。                        |
| dbname.system.indexes    | 列出所有索引。                            |
| dbname.system.profile    | 包含数据库概要(profile)信息。             |
| dbname.system.users      | 列出所有可访问数据库的用户。              |
| dbname.local.sources     | 包含复制对端（slave）的服务器信息和状态。 |

对于修改系统集合中的对象有如下限制。

在{{system.indexes}}插入数据，可以创建索引。但除此之外该表信息是不可变的(特殊的drop index命令将自动更新相关信息)。

{{system.users}}是可修改的。 {{system.profile}}是可删除的。



##	Mongodb的数据类型

| 数据类型           | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| String             | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。 |
| Integer            | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。 |
| Boolean            | 布尔值。用于存储布尔值（真/假）。                            |
| Double             | 双精度浮点值。用于存储浮点值。                               |
| Min/Max keys       | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。 |
| Array              | 用于将数组或列表或多个值存储为一个键。                       |
| Timestamp          | 时间戳。记录文档修改或添加的具体时间。                       |
| Object             | 用于内嵌文档。                                               |
| Null               | 用于创建空值。                                               |
| Symbol             | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。 |
| Date               | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID          | 对象 ID。用于创建文档的 ID。                                 |
| Binary Data        | 二进制数据。用于存储二进制数据。                             |
| Code               | 代码类型。用于在文档中存储 JavaScript 代码。                 |
| Regular expression | 正则表达式类型。用于存储正则表达式。                         |



# day02	基本增删改查

##	连接mongodb

* 连接url：

  ```
  mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
  ```

##	创建数据库	

use命令，如果数据库存在，则切换数据库,

```
use mydb
```

创建数据库以后，需要插入数据后，show dbs命令才能显示数据库

## 	删除数据库

```
db.dropDatabase()
```

该指令删除当前数据库，默认为test

例如，删除 jack数据库，需要先切换到jack，然后执行该指令

````
> use jack
switched to db jack
> db.dropDatabase()
{ "dropped" : "jack", "ok" : 1 }
>
````

##	 创建集合

###	创建指令

```
db.createCollection(name, options)
```

参数说明：

* name: 要创建的集合名称
* options: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段        | 类型 | 描述                                                         |
| :---------- | :--- | :----------------------------------------------------------- |
| capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔 | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。   |
| size        | 数值 | （可选）为固定集合指定一个最大值（以字节计）。 **如果 capped 为 true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

###	创建集合的案例

在 test 数据库中创建 runoob 集合：

```
use test
db.createCollection('runoob', {capped:true, size:1000})
show collections
```

###	自动创建集合

插入文档后，集合会自动创建

```js
use test
db.jack.insert({name:'jack', age:22})   // 执行后会自动创建一个名为jack的集合		
```



##	删除集合

```javascript
db.collection.drop()
```

 删除test数据库中的jack集合

```js
use test
db.jack.drop()
```

##	插入文档

* insert()
* insertOne()
* insertMany()
* save()  指定ID则更新数据或者插入， 入股哦不指定id，则与Insert()相同

```
db.COLLECTION_NAME.insert({})
```

插入一条记录到jack集合中

```javascript
use test
db.jack.insert({'name':'jack', age:12, description:'这是一个男人'})
```

##	更新文档

MongoDB 使用 **update()** 和 **save()** 方法来更新集合中的文档。接下来让我们详细来看下两个函数的应用及其区别。

###	update方法

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

###	update方法实例

我们在集合 col 中插入如下数据：

```javascript
>db.col.insert({
    title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
```

将title更新为MongoDB

````javascript
db.col.update({title:'MongoDB 教程'}, {$set:{title:'MongoDB'}})
````

以上语句只会修改第一条发现的文档，如果你要修改多条相同的文档，则需要设置 multi 参数为 true

```javascript
>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
```

### save() 方法

save() 方法通过传入的文档来替换已有文档。语法格式如下：

```javascript
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

​	参数说明：

- **document** : 文档数据。
- **writeConcern** :可选，抛出异常的级别。

### SAVE实例	

```javascript
> db.col.find().pretty()
{
        "_id" : ObjectId("5d29e802a8aee1467021e534"),
        "title" : "fdfsfdsf",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

替换以上的title

```javascript
db.col.save(
{
        "_id" : ObjectId("5d29e802a8aee1467021e534"),
        "title" : "我用save犯法改写了",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
})
```

### 更多实例

只更新第一条记录：

db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );

全部更新：

db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );

只添加第一条：

db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );

全部添加进去:

db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );

全部更新：

db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );

只更新第一条记录：

db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );