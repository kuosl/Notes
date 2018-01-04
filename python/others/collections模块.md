## defaultdict()

dictionary类似的对象，和dict不同主要体现在2个方面：
	* 可以指定key对应的value的类型。
	* 不必为默认值担心，换句话说就是不必担心有key没有value这回事。总会有默认的value.

```python3
from collections import defaultdict

s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]


# 元素为list类型, 默认为空list
d = defaultdict(list)

for k, v in s:
    d[k].append(v)

print(list(d.items()))


# 等价于上述defaultdict, 使用dict的setdefault方法
d_2 = {}
for k, v in s:
	# 当d_2中没有k时，设置[]为它的值
    d_2.setdefault(k, []).append(v)

print(list(d_2.items()))


# 当值不存在时，相当于直接在None.append, 当然异常
d_3 = {}

for k, v in s:
    d_3[k].append(v)

print(d_3.items())
```



## namedtuple()
namedtuple is a sub-class of tuple. namedtuple create an object with read-only attributes, which is similar to Class.

```
In [1]: from collections import namedtuple

In [2]: TPoint = namedtuple('TPoint', ['x', 'y'])

In [3]: p = TPoint(x=10, y=2)

In [4]: p
Out[4]: TPoint(x=10, y=2)

In [5]: p.x
Out[5]: 10

In [6]: p[0]
Out[6]: 10

In [7]: type(p)
Out[7]: __main__.TPoint

In [8]: repr(p)
Out[8]: 'TPoint(x=10, y=2)'

In [9]: for i in p:
   ...:     print(i)
      ...: 
	  10
	  2

In [10]: t = [22, 11]

# 使用_make来生成对象
In [11]: p = TPoint._make(t)

In [12]: p
Out[12]: TPoint(x=22, y=11)

# 属性是只读的
In [13]: p.x = 100
--------------------------------------------------------------------
AttributeError                     Traceback (most recent call last)
<ipython-input-13-1918fa5b021e> in <module>()
----> 1 p.x = 100

AttributeError: can't set attribute

# 将字典转换成namedtuple
In [14]: d = {'x': 44, 'y': 55}

In [15]: dp = TPoint(**d)

In [16]: dp
Out[16]: TPoint(x=44, y=55)
```
namedtuple最常用还是出现在处理来csv或者数据库返回的数据上。利用map()函数和namedtuple建立类型的_make（）方法。
