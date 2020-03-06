## 后端分页思想（MySQL百万级数据分页）

### 一、方法

1.  直接使用数据库提供的SQL语句

   + 语句样式: MySQL中,可用如下方法: SELECT * FROM 表名称 LIMIT M,N

   - 适应场景: 适用于数据量较少的情况(元组百/千级)
   - 原因/缺点: 全表扫描,速度会很慢 且 有的数据库结果集返回不稳定(如某次返回1,2,3,另外的一次返回2,1,3). Limit限制的是从结果集的M位置处取出N条输出,其余抛弃

2. 建立主键或唯一索引, 利用索引(假设每页10条)

   - 语句样式: MySQL中,可用如下方法: SELECT * FROM 表名称 WHERE id_pk > (pageNum*10) LIMIT M
   - 适应场景: 适用于数据量多的情况(元组数上万)
   - 原因: 索引扫描,速度会很快. 有朋友提出: 因为数据查询出来并不是按照pk_id排序的，所以会有漏掉数据的情况

3.  基于索引再排序

   - 语句样式: MySQL中,可用如下方法: SELECT * FROM 表名称 WHERE id_pk > (pageNum*10) ORDER BY id_pk ASC LIMIT M
   - 适应场景: 适用于数据量多的情况(元组数上万). 最好ORDER BY后的列对象是主键或唯一所以,使得ORDERBY操作能利用索引被消除但结果集是稳定的(稳定的含义,参见方法1)
   - 原因: 索引扫描,速度会很快. 但MySQL的排序操作,只有ASC没有DESC(DESC是假的,未来会做真正的DESC,期待...)

4.  基于索引使用prepare

   第一个？表示pageNum，第二个？表示每页元组数

   - 语句样式: MySQL中,可用如下方法: PREPARE stmt_name FROM SELECT * FROM 表名称 WHERE id_pk > (？* ？) ORDER BY id_pk ASC LIMIT M
   - 适应场景: 大数据量
   - 原因: 索引扫描,速度会很快. prepare语句又比一般的查询语句快一点。

5. 利用MySQL支持ORDER操作可以利用索引快速定位部分元组,避免全表扫描

   比如: 读第1000到1019行元组(pk是主键/唯一键)

   ```text
   SELECT * FROM your_table WHERE pk>=1000 ORDER BY pk ASC LIMIT 0,20
   ```

6. 利用"子查询/连接+索引"快速定位元组的位置,然后再读取元组

   比如(id是主键/唯一键,蓝色字体时变量)

   利用子查询示例:

   ```text
   SELECT * FROM your_table WHERE id <=
   (SELECT id FROM your_table ORDER BY id desc LIMIT ($page-1)*$pagesize ORDER BY id desc
   LIMIT $pagesize 
   ```

   利用连接示例:

   ```text
   SELECT * FROM your_table AS t1
   JOIN (SELECT id FROM your_table ORDER BY id desc LIMIT ($page-1)*$pagesize AS t2
   WHERE t1.id <= t2.id ORDER BY t1.id desc LIMIT $pagesize; 
   ```

   mysql大数据量使用limit分页，随着页码的增大，查询效率越低下。

### 二、测试

直接用limit start, count分页语句， 也是我程序中用的方法：

```text
select * from product limit start, count 
```

当起始页较小时，查询没有性能问题，我们分别看下从10， 100， 1000， 10000开始分页的执行时间（每页取20条）。

如下：

```text
select * from product limit 10, 20   0.016秒 
select * from product limit 100, 20   0.016秒
select * from product limit 1000, 20   0.047秒
select * from product limit 10000, 20   0.094秒
```

我们已经看出随着起始记录的增加，时间也随着增大， 这说明分页语句limit跟起始页码是有很大关系的，那么我们把起始记录改为40w看下（也就是记录的一般左右）

```text
select * from product limit 400000, 20   3.229秒 
```

再看我们取最后一页记录的时间

```text
select * from product limit 866613, 20   37.44秒 
```

像这种分页最大的页码页显然这种时间是无法忍受的。

从中我们也能总结出两件事情：

+ limit语句的查询时间与起始记录的位置成正比

+ mysql的limit语句是很方便，但是对记录很多的表并不适合直接使用

### 三、优化

1. 对limit分页问题的性能优化方法

   利用表的覆盖索引来加速分页查询

   我们都知道，利用了索引查询的语句中如果只包含了那个索引列（覆盖索引），那么这种情况会查询很快。

   因为利用索引查找有优化算法，且数据就在查询索引上面，不用再去找相关的数据地址了，这样节省了很多时间。另外Mysql中也有相关的索引缓存，在并发高的时候利用缓存就效果更好了。

   在我们的例子中，我们知道id字段是主键，自然就包含了默认的主键索引。现在让我们看看利用覆盖索引的查询效果如何。

   这次我们之间查询最后一页的数据（利用覆盖索引，只包含id列），如下：

   ```text
   select id from product limit 866613, 20 0.2秒 
   ```

   相对于查询了所有列的37.44秒，提升了大概100多倍的速度

   那么如果我们也要查询所有列，有两种方法，一种是id>=的形式，另一种就是利用join，看下实际情况：

   ```text
   SELECT * FROM product WHERE ID > =(select id from product limit 866613, 1) limit 20
   ```

   查询时间为0.2秒！

   另一种写法

   ```text
   SELECT * FROM product a JOIN (select id from product limit 866613, 20) b ON a.ID = b.id
   ```

   查询时间也很短！

2. 复合索引优化方法

   例子：

   数据表 collect ( id, title ,info ,vtype) 就这4个字段，其中 title 用定长，info 用text, id 是逐渐，vtype是tinyint，vtype是索引。这是一个基本的新闻系统的简单模型。现在往里面填充数据，填充10万篇新闻。最后collect 为 10万条记录，数据库表占用硬1.6G。

   OK ,看下面这条sql语句：

   ```text
   select id,title from collect limit 1000,10;
   ```

   很快；基本上0.01秒就OK，再看下面的

   ```text
   select id,title from collect limit 90000,10;
   ```

   从9万条开始分页，结果？

   8-9秒完成，my god 哪出问题了？其实要优化这条数据，网上找得到答案。看下面一条语句:

   ```text
   select id from collect order by id limit 90000,10;
   ```

   很快，0.04秒就OK。 为什么？因为用了id主键做索引当然快。网上的改法是：

   ```text
   select id,title from collect where id>=(select id from collect order by id limit 90000,1) limit 10;
   ```

   这就是用了id做索引的结果。可是问题复杂那么一点点，就完了。看下面的语句

   select id from collect where vtype=1 order by id limit 90000,10; 很慢，用了8-9秒！

   到了这里我相信很多人会和我一样，有崩溃感觉！vtype 做了索引了啊？怎么会慢呢？vtype做了索引是不错，你直接

   ```text
   select id from collect where vtype=1 limit 1000,10;
   ```

   是很快的，基本上0.05秒，可是提高90倍，从9万开始，那就是0.05*90=4.5秒的速度了。和测试结果8-9秒到了一个数量级。

   从这里开始有人提出了分表的思路，思路如下：

   建一个索引表： t (id,title,vtype) 并设置成定长，然后做分页，分页出结果再到 collect 里面去找info 。 是否可行呢？实验下就知道了。

   10万条记录到 t(id,title,vtype) 里，数据表大小20M左右。用

   ```text
   select id from t where vtype=1 order by id limit 90000,10;
   ```

   很快了。基本上0.1-0.2秒可以跑完。为什么会这样呢？我猜想是因为collect 数据太多，所以分页要跑很长的路。limit 完全和数据表的大小有关的。其实这样做还是全表扫描，只是因为数据量小，只有10万才快。OK， 来个疯狂的实验，加到100万条，测试性能。加了10倍的数据，马上t表就到了200多M，而且是定长。还是刚才的查询语句，时间是0.1-0.2秒完成！分表性能没问题？

   错！因为我们的limit还是9万，所以快。给个大的，90万开始

   ```text
   select id from t where vtype=1 order by id limit 900000,10;
   ```

   看看结果，时间是1-2秒！why ?

   分表了时间还是这么长，非常之郁闷！有人说定长会提高limit的性能，开始我也以为，因为一条记录的长度是固定的，mysql 应该可以算出90万的位置才对啊？可是我们高估了mysql 的智能，他不是商务数据库，事实证明定长和非定长对limit影响不大？怪不得有人说到了100万条记录就会很慢，我相信这是真的，这个和数据库设计有关！

   难道MySQL 无法突破100万的限制吗？？？到了100万的分页就真的到了极限？

   答案是： NO 为什么突破不了100万是因为不会设计mysql造成的。下面介绍非分表法，来个疯狂的测试！一张表搞定100万记录，并且10G 数据库，如何快速分页！

   好了，我们的测试又回到 collect表，开始测试结论是：

   30万数据，用分表法可行，超过30万他的速度会慢道你无法忍受！当然如果用分表+我这种方法，那是绝对完美的。但是用了我这种方法后，不用分表也可以完美解决！

   答案就是：复合索引！ 有一次设计mysql索引的时候，无意中发现索引名字可以任取，可以选择几个字段进来，这有什么用呢？

   开始的

   ```text
   select id from collect order by id limit 90000,10; 
   ```

   这么快就是因为走了索引，可是如果加了where 就不走索引了。抱着试试看的想法加了 search(vtype,id) 这样的索引。

   然后测试

   ```text
   select id from collect where vtype=1 limit 90000,10; 
   ```

   非常快！0.04秒完成！

   再测试:

   ```text
   select id ,title from collect where vtype=1 limit 90000,10; 
   ```

   非常遗憾，8-9秒，没走search索引！

   再测试：search(id,vtype)，还是select id 这个语句，也非常遗憾，0.5秒。

### 四、综述

综上：如果对于有where 条件，又想走索引用limit的，必须设计一个索引，将where 放第一位，limit用到的主键放第2位，而且只能select 主键！

完美解决了分页问题了。可以快速返回id就有希望优化limit ， 按这样的逻辑，百万级的limit 应该在0.0x秒就可以分完。看来mysql 语句的优化和索引时非常重要的！

### 五、附录（mysql索引类型）

FULLTEXT、NORMAL、SPATIAL、UNIQUE的详细介绍

1. 分类

   + Normal 普通索引

     表示普通索引，大多数情况下都可以使用

   + Unique 唯一索引

     表示唯一的，不允许重复的索引，如果该字段信息保证不会重复例如身份证号用作索引时，可设置为unique

     约束唯一标识数据库表中的每一条记录，即在单表中不能用每条记录是唯一的（例如身份证就是唯一的），Unique(要求列唯一)和Primary Key(primary key = unique + not null 列唯一)约束均为列或列集合中提供了唯一性的保证，Primary Key是拥有自动定义的Unique约束，但是每个表中可以有多个Unique约束，但是只能有一个Primary Key约束。

   + Full Text 全文索引

     表示全文收索，在检索长文本的时候，效果最好，短文本建议使用Index,但是在检索的时候数据量比较大的时候，现将数据放入一个没有全局索引的表中，然后在用Create Index创建的Full Text索引，要比先为一张表建立Full Text然后在写入数据要快的很多

     FULLTEXT 用于搜索很长一篇文章的时候，效果最好。用在比较短的文本，如果就一两行字的，普通的 INDEX 也可以。

   + SPATIAL 空间索引

     空间索引是对空间数据类型的字段建立的索引，MYSQL中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING、POLYGON。MYSQL使用SPATIAL关键字进行扩展，使得能够用于创建正规索引类型的语法创建空间索引。创建空间索引的列，必须将其声明为NOT NULL，空间索引只能在存储引擎为MYISAM的表中创建

2. btree索引和hash索引的区别

   + BTREE（B树（可以是多叉树）） {主流使用}

   + HASH（key,value） 这种方式对范围查询支持得不是很好

     hash 索引结构的特殊性，其检索效率非常高，索引的检索可以一次定位，不像B-Tree 索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以 Hash 索引的查询效率要远高于 B-Tree 索引。
     可 能很多人又有疑问了，既然 Hash 索引的效率要比 B-Tree 高很多，为什么大家不都用 Hash 索引而还要使用 B-Tree 索引呢？任何事物都是有两面性的，Hash 索引也一样，虽然 Hash 索引效率高，但是 Hash 索引本身由于其特殊性也带来了很多限制和弊端，主要有以下这些：

     + Hash 索引仅仅能满足”=”,”IN”和”<=>”查询，不能使用范围查询。

       由于 Hash 索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。

     + Hash 索引无法被用来避免数据的排序操作

       由于 Hash 索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算

     + Hash 索引不能利用部分索引键查询

       对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用

     + Hash 索引在任何时候都不能避免表扫描

       前面已经知道，Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash 值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果

     + Hash 索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高

       对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下

3. 选择

   在实际操作过程中，应该选取表中哪些字段作为索引？为了使索引的使用效率更高，在创建索引时，必须考虑在哪些字段上创建索引和创建什么类型的索引,有7大原则：

   + 选择唯一性索引
   + 为经常需要排序、分组和联合操作的字段建立索引
   + 为常作为查询条件的字段建立索引
   + 限制索引的数目
   + 尽量使用数据量少的索引
   + 尽量使用前缀来索引
   + 删除不再使用或者很少使用的索引
   + 经常更新修改的字段不要建立索引（针对mysql说，因为字段更改同时索引就要重新建立，排序，而Orcale好像是有这样的机制字段值更改了，它不立刻建立索引，排序索引，而是根据更改个数，时间段去做平衡索引这件事的）
   + 不推荐在同一列建多个索引

