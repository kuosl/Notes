## python在内存管理上做了大量工作。

<!--是不是nginx也有类似的原理?-->

#### 小于等于256KB，取名为arena的**大块内存**，并按**系统页大小**，划分为多个pool。

- 每个pool继续分割成n个大小相同的block, 这是内存管理的最小单位。block大小是8的倍数。
  - 也就是说存储13个字节的对象，需要找到block为16的pool获取空闲块。
- 所有这些都用头信息和链表管理起来，以便快速查找空闲区域进行分配。

#### 大于256字节的对象，直接使用malloc在堆上分配内存。

---

不可变类型：int, long, str, tuple, frozenset

除了某些类型自带copy方法外，还可以:

* 使用标准库的copy模块进行深度复制
  * 循环引用会影响deepcopy操作。
* 序列化对象，如pickle, cPickle, marshal。



# 第2章 内置类型

数据类型：

* 空值 ：None
* 数字：bool, int, long, float, complex
* 序列：str, unicode, list, tuple
* 字典：dict
* 集合：set, frozenset

### 2.1 数字

#### bool

None, 0, 空str,  无元素的容器都是False

#### int

* sys.maxint
* PyIntBlock内存只利用 不回收，同时持有大量整数对象将导致内存暴涨。
* 用range创建一个巨大的数字列表，就需要足够多的PyIntBlock为数字对象提供存储空间。
* 换成xrange就不同了，每次迭代后，数字对象被回收，其战胜内存空闲出来并被复用，内存也就不会暴涨了。
* 其实在python3也使用xrange替换掉原来默认的range, 使用python2时需要注意。

#### long

当超出int限制时，会自动转换成long, **<u>作为变长对象</u>**，只要内存足够，足够存储无法想象的天文数字。

#### float

- 使用双精度浮点数，不能“精确”表示某些十进制的小数值，尤其是“四舍五入”的结果。

- 某些比对操作会出现莫名其妙的错误

  ```python
  	>>> 0.1 * 3 == 0.3
  	False
      
      >>> round(2.675, 2) # 并不会四舍五入
      2.67
  ```

  ​

#### str

```python
	>>> r"abc\x" 	# r前缀表示非转义的raw-string
	'abc\\x'
`	>>> "中国人"	# UTF-8字符串，Linux默认编码
	'\xe4\xb8\xad...'
    >>> u"中国人"	# unicode字符串
	u'\u4e2d\u56fd\u4eba'

```

**操作有：**

* 相加  “a” + "b"  --> "ab"

* 乘法: "a" * 3    --> "aaa"

* 合并：",".join(["a", "b", "c"])

* 按字符分割："a,b,c".split(",")

* 按行分割："a\n\b\r\nc".splitlines() --> ["a", "b", "c"]

* 分割后保留换行符："a\n\b\r\nc".splitlines(True) --> ["a\n", "b\r\n", "c"]

* 判断以特定子串开始或结束: "abc".startwith("ab"), "abc".endswith("bc")

* 大小写转换："abc".upper(), "Abc".lower()

* 可指定查找的起始和结束位置：

  "abcabc".find("bc")  --》 1

  "abcabc".find("bc", 2) # 从第3个字符中查找 --》4

* 删除前后空格：

  " abc".lstrip() 前删空

  "abc ".rstrip() 后删空

  " abc ".strip() 前后都删空

* 删除指定的前后缀字符：

  “abc”.strip("ac") --》"b" 

* 可指定替换次数：子串替换

  ”abcabc“.replace("bc", "BC") --> "aBCaBC"

  ”abcabc“.replace("bc", "BC"， 1) --> "aBCabc" # 只替换1次

* 将tab替换成空格：

  ”a\tbc“.expandtabs(4) -> "a    bc"

* 填充：

  "123".ljust(5, '0') --> "12300"

  "123".rjust(5, '0') --> "00123"

  "123".center(5, '0') --> "01230"

  "123".center(6, '0') --> "012300"

* 数字填充：

  "123".zfill(6) --> "000123"

  "123456".zfill(4) --> "123456"


#### 编码

python2默认采用ascii编码。

str, unicode都提供了encode和decode编码转换方法

注意 **python的codecs模块含有各种模块转换操作。**



字符串格式化：

```python
In [2]: "item:%(item) price is %(price)" % dict(item='apple', price='12yuan')
--------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-2-eaca45632cd0> in <module>()
----> 1 "item:%(item) price is %(price)" % dict(item='apple', price='12yuan')

ValueError: unsupported format character 'p' (0x70) at index 13

# 注意这里：格式化的%(item)s 后面有个表示接受字符串的s
# 同理 %((price)d
# 虽然后面值可以用dict担任，但前面模式串仍然后必须有%(xx)类型
In [3]: "item: %(item)s price is %(price)d yuan" % dict(item="apple", price=12)
Out[3]: 'item: apple price is 12 yuan'
```

**若前半拉的模块串中使用%, 那后面也应该使用%, 而非format**

**若前半拉的模块串包含花括号，那就应该使用format，而非%**



#### %的格式化只像是C语言， format更强大

##### format的功能：

####### 参数是字典时，模式串可以使用key, 否则花括号里是数组的索引值

* 命名参数

  ```python
  "{key}={value}".format(key='a', value=10)
  ```

* field多次使用

  ```python
  "after {key}={value}, then {key}=?".format(key='a', value=10)
  ```

* 千分位符号

* **左中右对齐**

  ```python
  In [20]: "[{0:<10}],[{0:^10}],[{0:>10}]".format("a")
  Out[20]: '[a         ],[    a     ],[         a]'

  # 带填充的
  In [21]: "[{0:*<10}],[{0:*^10}],[{0:*>10}]".format("a")
  Out[21]: '[a*********],[****a*****],[*********a]'
  ```

* 对象，成员

```python
In [9]: import sys
In [10]: "{0.platform}".format(sys)
Out[10]: 'linux2'
```

* 字典,  key

```python
In [12]: "item: {0[item]} price: {0[price]}".format(dict(item='apple', price=12))
Out[12]: 'item: apple price: 12'
```

* 列表

  ```python
  In [22]: "{0[5]}".format(range(10))
  Out[22]: '5'
  ```



#### 池化(了解即可，没那么神奇和不怎重要)

在python进程中，无数对象拥有一堆类似'\_\_name\_\_',  '\_\_doc\_\_' 这样的名字，池化有助于减少对象数量和内存消耗，提升性能。

```python
In [27]: s = "".join(['a', 'b', 'c'])

In [28]: s is "abc"
Out[28]: False

In [29]: intern(s) is "abc"
Out[29]: True

In [30]: intern(s) is intern(s)
Out[30]: True
```

当池化的对象不再有引用时，将被 回收

#### 字典

要判断两个字典的差异，使用视图是最简单的做法。

```python
In [1]: d1 = dict(a=1, b=2)
In [2]: d2 = dict(b=2, c=3)
In [3]: d1 & d2
TypeError: unsupported operand type(s) for &: 'dict' and 'dict'
        
In [4]: v1 = d1.viewitems()
In [5]: v2 = d2.viewitems()
# 接下来可对v1, v2 进行交集 &，并集 |，差集 -操作
# 还有 对称差集 ^（不会同时出现在v1和v2中）由此知二者是否相同 
# 还有
>>> ('a', 1) in v1

# 还有其它视图 viewitems, viewkeyss
```

