## 使用Python/Java生成UUID

### 一、UUID简介

+ UUID(Universally Unique Identifier)全局唯一标识符,是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。按照开放软件基金会(OSF)制定的标准计算，用到了以太网卡地址、纳秒级时间、芯片ID码和许多可能的数字。由以下几部分的组合：当前日期和时间(UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒又生成一个UUID，则第一个部分不同，其余相同)，时钟序列，全局唯一的IEEE机器识别号（如果有网卡，从网卡获得，没有网卡以其他方式获得），UUID的唯一缺陷在于生成的结果串会比较长。

+ UUID出现的目的，是为了让分布式系统可以不借助中心节点，就可以生成UUID来标识一些唯一的信息；

  GUID，是Globally Unique Identifier的缩写，跟UUID是同一个东西，只是来源于微软。

+ 1个UUID是1个16字节（128位）的数字，在数据库中可以设置类型为char(16)

  为了方便阅读，通常将UUID表示成如下的方式：

  123e4567-e89b-12d3-a456-426655440000

  1个UUID被连字符分为五段，形式为8-4-4-4-12的32个字符。

  其中的字母是16进制表示，大小写无关。

+ UUID的格式是这样的：xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx

  N那个位置，只会是8,9,a,b

  M那个位置，代表版本号，由于UUID的标准实现有5个版本，所以只会是1,2,3,4,5

### 二、不同版本（Java只支持生成版本3和版本4的UUID）

#### 1. 基于时间的UUID

通过当前时间戳、机器MAC地址生成；

由于在算法中使用了MAC地址，这个版本的UUID可以保证在全球范围的唯一性。

但与此同时，因为它暴露了电脑的MAC地址和生成这个UUID的时间，这就是这个版本UUID被诟病的地方。

在python里面的使用的例子：

```python
>>> import uuid
>>> uuid.uuid1()
UUID('444b5cc0-ae5d-11e6-8d22-28924a431726')
>>> uuid.uuid1()
UUID('46a9bf21-ae5d-11e6-9549-28924a431726')
```

其中，最后的12个字符`28924a431726`就是电脑网卡的MAC地址

#### 2. DCE安全的UUID

DCE安全的UUID和基于时间的UUID算法相同，但会把时间戳的前4位置换为POSIX的UID或GID。

不过，在UUID的规范里面没有明确地指定，所以基本上所有的UUID实现都不会实现这个版本。

#### 3. 基于名字空间的UUID（MD5）

由用户指定1个namespace和1个具体的字符串，通过MD5散列，来生成1个UUID；

根据规范描述，这个版本的存在是为了向后兼容?平时这个版本我们也很少用到

在python里面的使用的例子：

```python
>>> import uuid
>>> uuid.uuid3(uuid.NAMESPACE_DNS, "myString")
UUID('21fc48e5-63f0-3849-8b9d-838a012a5936')
>>> uuid.uuid3(uuid.NAMESPACE_DNS, "myString")
UUID('21fc48e5-63f0-3849-8b9d-838a012a5936')
```

在java中使用的例子:

```java
System.out.println(UUID.nameUUIDFromBytes("myString".getBytes("UTF-8")).toString());
```

#### 4. 基于随机数的UUID

根据随机数，或者伪随机数生成UUID。这种UUID产生重复的概率是可以计算出来的，但随机的东西就像是买彩票。这个版本应该是平时大家无意中用得最多的版本了。

在python里面使用的例子：

```python
>>> import uuid
>>> uuid.uuid4()
UUID('e584539d-a334-4f15-9819-88d73fcf707d')
>>> uuid.uuid4()
UUID('76ec02cc-1b1d-4ad3-bd09-a4f6d67c7af4')

#可以使用replace函数将'-'去掉,变成大写
>>> return str(uuid.uuid4()).replace("-", "").upper()
```

以及Java中大家最熟悉的：

```java
System.out.println(UUID.randomUUID().toString());

//可以使用replace函数将'-'去掉，变成大写
return UUID.randomUUID().toString().replace("-", "").toUpperCase();
```

#### 5. 基于名字空间的UUID（SHA1）

和版本3一样，不过散列函数换成了SHA1

在python里面的使用的例子：

```python
>>> import uuid
>>> uuid.uuid5(uuid.NAMESPACE_DNS, "myString")
UUID('cd086011-6aac-5a06-a94a-0b67c59649ba')
>>> uuid.uuid5(uuid.NAMESPACE_DNS, "myString")
UUID('cd086011-6aac-5a06-a94a-0b67c59649ba')
```

### 三、总结

从几个版本的定义来看，感觉都不是特别完美，可能版本4是平时用得最多的，但是在现实的业务场景中，考虑到可读性、唯一性、长度，我们一般也不会选择UUID当做数据库的主键。

至于其他场景的应用，可以结合具体的场景，来使用各个版本的实现。

