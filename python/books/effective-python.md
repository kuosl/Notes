# 第1章 用Pythonic方式思考
## 第2条 遵守PEP8风格  
• 使用4个空格缩进  
• 多行另一起一行时，缩进4空格  
• 函数与类之间应用两空行相隔  
• 各方法一空行相隔  
• 函数，变量，属性使用linux风格命名  
• 受保护的实例属性，应以单下划线开头  
• 私有的实例属性，应以两下划线开头  
• 类与异常，使用大写开头命名  
• 模块级别的常量以全大写  
• 类中实例方法首个参数应是self，表示对象本身  
• 类方法首个参数应是cls，表示类本身  
**• 直接判断列表是否真值来判断列表是为空，如应if a: 而不是if len(a) == 0**  
• 不要编写单行的 if for while except语句  
• import 语句始终在文件开头  
• <u>引入模块时，应总使用绝对名称，而不应使用根据当前模块的路径而使用相对名称</u>，如：引入foo包中bar模块，应完整写出from bar import foo, 而不应简写import foo  
• <u>若非要相对名称来import， 则应from . import foo</u>  
• 将import语句分为三块：1标准库模块 2第三方模块 3自用模块   



## 第3条 了解bytes, str, unicode  

| python版本 | 含原始8位值 | 含unicode字符 |
| ---------- | ----------- | ------------- |
| python2    | **str**     | **unicode**   |
| Python3    | **bytes**   | **str**       |


* 两个python版本中， unicode并没有与特定二进制编码关联
* 到8位值转换是encode，到unicode转换是decode
* **一定要把编解码操作写在核心代码最外围。核心代码应使用unicode，且不要假定字符编码类型。**  
* python2中7位值的ascii值时，unicode等价str， 而在<u>python3中， bytes与str绝不等价</u>
* **python3中内置open函数默认使用utf-8编码，而python2中默认是二进制, 因此在python3中操作二进制文件应该指明'wb'或'rb'**



## 第4条 辅助函数取代复杂表达式（略）

## 第5条 切割序列 slice

切片时，start end越界不会出现问题，利用这一特性可限定最大长度  
```
first_twenty_items = a[:20]
last_twenty_items = a[-20:]
```
切片是深拷贝 
```
b = a[:]
assert b == a and b is not a
```



## 第6条 在单次切片操作内，不要同时指定start, end和stride

**如果三者全都指定，容易变得费解。**

另外，如果非要乃到stride(steps)时，尽量使用正值，同时忽略start和start索引。

如果一定需要配合start或end索引来使用stride，那么应该分成两步：

1.  先做步进式切片 						—— stride
2.  把第1步切片结果再做第二次范围切割        —— start, end

上述方法会多产生一份浅拷贝，可以考虑使用内置模块intertools的islice。



## 第7条 用列表推导来取代map filter
```python
# 列表推导比较简洁
a = [1,2,3,4]
squres = [x ** 2 for x in a]
# 使用map
squres =  map(lambda x: x**2, a)

# 加上条件过滤的列表推导与map，可以看出列表推导更简洁
squres = [x ** 2 for x in a if x%2==0]
squres =  map(lambda x: x**2, filter(lambda x: x%2==0, a))
```
* 对字典dict和集set，也支持列表推荐



## 第8条 不要使用含有两个以上表达式的列表推导（略）

## 第9条 用生成器表达式来改写数据量较大的列表推导
**列表推导较耗内存**
```python
# 很耗内存
value = [len(x) for x in open('/tmp/myfile.txt')]
print(value)
```
* **把实现列表推导所用的那种写法放在一对圆括号内，就构成了生成器表达式。**  
* 生成器表达式在求值时会立即返回一个迭代器，而不会深入处理文件中的内容。  
```python
it = (len(x) for x in open('/tmp/myfile.txt'))
print(it)
>>>
<generator object ... at ...>
#使用next函数操作上步返回的迭代器
print(next(it)）
print(next(it)）
...
```

## 第10条：尽量使用enumerate取代range
**内置的enumerate函数可以把各种迭代器包装为生成器，以便稍后产生输出值。**
*每次返回两个值：索引和值*

```python
flavor_list = ['vanilla', 'chocolate', 'pecan', 'strawberry']
for i, flavor in enumerate(flavor_list):
	print("%d: %s' % (i+1, flavor))
>>>
1: vanilla
2: chocolate
  ...
```
还可以直接指定enumerate开始计数时的值，从N开始
enumerate(flavor_list, 2)



## 第11条：使用zip函数同时遍历两个迭代器

**help(zip)**
	zip(seq1 [, seq2 [...]]) -> [(seq1[0], seq2[0] ...), (...)]

内置zip有两个问题：
1. <u>python2中的zip并不是生成器，它会把所有迭代器都平等的遍历一遍。因此需要使用内置模块itertools的izip函数</u>
2. 若迭代器参数长度不同，zip会在较短的长度停止。 此时考虑用itertools.zip_longest函数



## 第12条：不要在for和while循环后写else块

python提供了比较特殊的语法：循环后面可以直接编写else块
```python
for i in range(3):
	print('Loop %d" % i)
else
	print('Else block!")

>>> 正常执行，执行了else
Loop 0
Loop 1
Loop 2
Else block!
​```
```

可能使用人误解为：若循环没有正常执行才执行else. 
而**实际是循环中有break(即使是在最后一步break)，else才不执行**
	​```python
	for i in range(3):
		print i
		if i == 2: #在最后一步break
			break
	else:
		print 'xxxxxxxx'
	
	>>> 不会执行else
	0
	1
	2
	​```
**在for x in []: 或 while False: 之类的循环后的else会马上执行**



## 第13条：合理利用try/except/else/finally结构中的每个代码块
异常处理考虑四种不同的时机：try/except/else/finally
1. finally块: **如果既要向上传播异常，又要在异常时执行清理工作，则使用finally.**
  ```python
  handle = open('/tmp/random_data.txt') # 可能抛出IOError
  try:
  	data  = handle.open()			# 可能抛出UnicodeDecodeError
  finally:
  	handle.close()				    #  always runs after try
  ```
2. else块:  **try块没有发生异常，就执行else块**
  可以用来缩减try块中的代码量，并把没有发生异常时所要执行的语句与try/except代码块隔开
  必须有except块
  ```python
  def load_json_key(data, key):
  	try:
  		result_dict = json.loads(data) # may raise ValueError
  	except ValueError as e:
  		raise KeyError from e
  	else:
  		return result_dict[key]			# May raise KeyError
  ```




# 第2章 函数

## 第14条： 尽量用异常来表示特殊情况，而非返回None表示异常
* **使用None这个返回值来表示特殊意义的函数，容易使用调用者犯错，因为None和0及空字符串之类的值，在条件表达式都会评估为False**
  * 除非返回值为元组，比如：第一个返回值为状态，第二个返回值才是结果，但这种方式使用不太直观，也不太好，尽量别用。
* **函数在遇到特殊情况时，应该抛出异常，而不要返回None。调用者看到异常后，应该编写相应代码处理它们。**




## 第15条： 了解如何在闭包里使用外围作用域的变量

### 引用变量查找顺序：

1. 当前函数作用域
2. 任何外围作用域（例如，包含当前函数的其它函数）
3. 包含当前模块的作用域（也叫全局作用域，global scope)
4. 内置作用域(也就是包含len及str等函数的作用域)
5. **若没找到则抛出NameError异常**



### 定义变量赋值规则有所不同：  

1.  **在当前模块中若无定义，则在当前模块内定义这个变量，不会去上级（或外围）作用域去查找。**
2.  目的是防止函数的每个赋值操作污染外围作用域。

```python
#coding=utf-8

def sort_priority(values, group):
	"""sort values by prioriy group"""
	found = False
	def helper(x):
		if x in group:
			print("found !!!")
			found = True #此处并不是访问修改闭包（包围）函数的found而是重新定义了一个helper函数内部的found变量
			return (0, x)
		else:
			return (1, x)
	values.sort(key=helper)
	return found

if __name__ == '__main__':
	numbers = [8,3,1,2,5,4,7,6]
	group = {2,3,5,7}
	found = sort_priority(numbers, group)
	print("Found:", found)
	print(numbers)

>>>
Found!!!
Found!!!
Found!!!
Found!!!
Found: False # 并没有修改闭包内函数 之外 的found变量！
[2, 3, 5, 7, 1, 4, 6, 8]
```


仅在python3中，要想获取闭包内的数据，可以在内层空间中用nonlocal语句声明。

```python
# Python3
def sort_priority3(values, group):
	"""sort values by prioriy group"""
	found = False
	def helper(x):
		nonlocal found # 声明found在外面定义了
		if x in group:
			found = True
			return (0, x)
		else:
			return (1, x)
	values.sort(key=helper)
	return found

if __name__ == '__main__': numbers = [8,3,1,2,5,4,7,6]
	group = {2,3,5,7}
	found = sort_priority3(numbers, group)
	print("Found:", found)
	print(numbers)

>>>
Found: True
[2, 3, 5, 7, 1, 4, 6, 8]
```
**与nonlocal类似的是global语句声明，前者声明的是闭包的外围函数中的变量，而后者声明的是模块的全局变量。**
**python2也支持global语句，但不支持nonlocal。**

若想达到类似效果，可用以下技巧（**虽然不能直接给范围之外变量赋值，但可以改变变量内部内容，从而绕过上述限制**）

```python
# Python2
def sort_priority(values, group):
	"""sort values by prioriy group"""
    # 改外部变量声明成列表，字典等包含成员的变量
	found2 = [False,]
	def helper(x):
		if x in group:
             # 虽不可改变found2的值，但可以改变found2成员值
			found2[0] = True
			return (0, x)
		else:
			return (1, x)
	values.sort(key=helper)
	return found2[0]

if __name__ == '__main__':
	numbers = [8,3,1,2,5,4,7,6]
	group = {2,3,5,7}
	found2 = sort_priority(numbers, group)
	print("Found2:", found2)
	print(numbers)

>>>
Found: True
[2, 3, 5, 7, 1, 4, 6, 8]
```


**最后，为了代码可读性，尽量少用global及nonlocal**



## 第16条： 考虑用生成器yield来改写直接返回列表的函数
1. 使用生成器有时结果更清晰:直接使用yield返回结果
2. 节省资源，每次调用才产生一个结果，不必一下子把所有结果计算出来
3. **不方便之处就是不能重复调用,因为每次调用会改变内部某些状态**




## 第17条： 在参数上面迭代时，要多加小心

迭代器只产生一轮结果，在抛出StopIteration异常的迭代器或生成器上面继续迭代第二轮是不会有结果的。

**在已经迭代完的迭代器上继续迭代时，有时不报错。因为for循环，list构造器及标准库中许多其它函数都认为正常的操作过程中完全有可能出现StopIteration异常，这些函数没办法区别这个迭代器本来就没有输出或是已经用完了。**



## 第18条： 用数量可变的位置参数减少视觉杂讯

可以使用*args（可变数量参数）来在处理可选参数列表
* 不使用可变数量参数

```python
def log(msg, vals):
	if not vals:
		print(msg)
	else:
		val_str = ', '.join(str(x) for x in vals)
		print('%s: %s' % (msg, val_str))

log('my num are', [1,2])
log('hi', [])
log('hi')

>>>
   8 log('my num are', [1,2])
   9 log('hi', []) #必须使用一个空表占位
-> 10 log('hi') #否则出现异常
TypeError: log() takes exactly 2 arguments (1 given)
```

* 使用可变数量参数

```python
def log(msg, *args):
	if not args:
		print(msg)
	else:
		val_str = ', '.join(str(x) for x in args)
		print('%s: %s' % (msg, val_str))

log('my num are', [1,2])
log('hi', []) # 空表不再必要，留着它也会原样输出
log('hi')     # OK
# >>>
# my num are: [1, 2]
# hi: [] 
# hi

fav = [7, 33, 9]
log('Favortie colors', fav)
# >>>
# Favorite colrs: 7, 33, 9
```

**接受数量可变参数会带来两个问题：**
1. 可变参数在付给函数时，总是要先转化成元组。这意味着若拿生成器作为参数来调用，python必须先把生成器完整的迭代一轮，然后放在元组中，这可能消耗大量内存。
  **因此只有确认可变数量参数只是有限个数时才应该使用。**
2. **若以后为函数添加新的位置参数时，如果只修改函数定义而不修改旧有的调用代码，则会产生难以发现的问题**。因为新添加的位置参数会被它后面的可变数量参数掩盖。
  **为避免这种情况，应当使用关键字指定的参数来扩展这种接受*args的函数。**




## 第19条： 用关键字参数来表达可选的行为

**位置参数必须出现在关键字参数之前，关键字参数只能出现在位置参数之后。**
关键字参数有三个好处：
1. 容易理解
2. 可提供默认值
3. 上文提到的扩充数量可变参数函数




## 第20条： 用None和文档字符串来描述具有动态默认值的参数

**动态默认值参数：**比如想用当前时间（动态变化的）表示when的默认值

```python
def log(msg, when=datetime.now())
	'''肯定会执行失败，因为when只执行一次，在模块加载函数时确认when的值'''
	print("%s: %s" % (when, msg))
```
**在Python中若想正确的实现动态默认值，习惯上把默认值设为None，并在文档字符串docstring里面把None所对应的实际行为描述出来。编写函数代码时，若发现该参数值为None，则将其设为实际的默认值。**

```python
#coding=utf-8
from datetime import datetime
from time import sleep

def log(msg, when=None):
	"""
	Log a message whith a timestamp.

	Args:
		msg: message to print
		when: datetime of when the message occurred.
			Defaults to the present time.
	"""
	when = datetime.now() if when is None else when
	print("%s: %s" % (when, msg))

log('Hi there!')
sleep(0.1)
log('Hi again!')
# >>>
# 2017-11-11 11:11:11.0555 Hi there!
# 2017-11-11 11:11:11.1555 Hi Again!
```
**若参数的实际默认值是可变类型(mutable), 比如{}, []等动态的值，一定要使用None作为形参的默认值。**
**形参中指定的默认值只会在模块加载时评估一次，所有调用这个函数的地方都共享一个default值，造成逻辑问题。

```python
# 错误的默认值
def decode(data, default={})
	try:
		return json.load(data)
	except ValueError:
		return defult

# 正确的默认值应该是None
def decode2(data, default=None)
	"""Load JSON data from a string.
	Args:
		data: JSON data to decode.
		default: Value to return if decoding fails.
			Defaults to an empty dictionary.
	"""
	if default is None:
		default = {}
	try:
		return json.load(data)
	except ValueError:
		return defult
```



## 第21条： 用只能以关键字形式指定的参数来确保代码明晰

**python3中，可以定义一种只能以关键字形式来指定的参数，从而确保调用该函数的代码读起来会比较明确。**
**参数列表中的*号，标志着位置参数就此终结，之后的那些参数，都只能以关键字形式来指定。**

```python3
# Python 3
def safe_division(number, divisor, 
				   *,                    # 标志着位置参数就此终结，后面的参数只能以关键字形式指定
				  ignore_overflow=False, ignore_zero_division=False):
	pass

In [3]: safe_division(1,2,True, False)    # 错误调用示例，只能指定两个位置参数
>>>
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-3-87d11d5dc098> in <module>()
----> 1 safe_division(1,2,True, False)
TypeError: safe_division() takes 2 positional arguments but 4 were given
In [4]: safe_division(1,2,ignore_overflow=True, ignore_zero_division=False)

In [5]: safe_division(1,2)                # 不指定位置参数，使用默认参数
>>>

```

**python2 中没有以上的语法支持。不过可以在参数列表中使用**参数机制，并且令函数在遇到无效的调用时抛出TypeErrors, 从而实现以上类似功能。
从但是同以上python3的方式相比，python2可以接受任意数量的关键字参数，因此需要在函数实现排除不想要的位置参数。
用pop方法把期待的关键字参数从kwargs字典中取走，若字典的键里面没有那个关键字，那么pop的第二个参数就会成为默认值。
最后为了防止调用者提供无效参数值，需要确认kwargs字典里面已经没有关键字参数了。

```python2
# Python 2
def safe_division(numberm, divisor, **kwargs)L
	ignore_overflow = kwargs.pop('ignore_overflow', False)
	ignore_zero_division = kwargs.pop('ignore_zero_division', False)
	if kwargs:	# 查看是否还有参数剩余，这些就是调用者提供的多余未期待参数
		raise TypeError('Unexpected ** kwargs: %r' % kwargs)
	# ...
```
同Python3版本的函数一样不接受位置参数。



# 第3章 类与继承
Python提供了继承，多态，封闭等各种OOB特性。



## 第22条： 尽量用辅助类来维护程序的状态，而不要用字典和元组

程序状态较简单时才用字典或元组，否则使用辅助类完成状态管理。



## 第23条：简单的接口应该是传递函数，而不是类的实例

函数可以像参数一样传递，因为函数是一级对象。
若定义类的__call__方法，则类的实例就可以像普通函数一样调用。
若需要函数保存运行状态，则应该把函数定义为类，并实现__call__, 而非闭包。
这样代码更清晰，而且__call__明确了类的实例作为闭包使用。



## 第24条：以@classmethod的形式的多态去通用地构建对象

在python中不仅对象支持多态，类也支持多态。

多态，使得继承体系中的多个类都能以各自所独有的方式来实现某个方法，有相同的接口，但执行不同的行为。

```python
#########################################################
## 实现一套MapReduce流程	################################
#########################################################

# 1. 定义公共基类表示输入的数据
class InputData(object):
	def read(self):
		raise NotImplementedError

# 1.1 具体的子类，从磁盘读取数据
class PathInputData(InputData)
	def __init__(self, path):
		super().__init__()
		self.path = path
	def read(self):
		return open(self.path).read()

# 可能有InputData的其它子类，比如从网络读取并解压数据

###################################################################################

# 2. 为MapReduce工作线程定义一套类似的抽象接口，以便用标准方式处理输入数据
class Workder(object):
	def __init__(self, input_data):
		self.input_data = input_data
        self.result = None
    def map(self):
        raise NotImplementedError
    def reduce(self, other):
        raise NotImplementedError
    
# 2.1 定义具体的Worker子类，以实现MapReduce功能
# 实现简单的换行符计数器
class LineCounterWorker(Worker):
    def map(self):
		data = self.input_data.read()
        self.result = data.count('\n')
    def reduce(self, other):
		self.result += other.result
       ################################################################################################
#怎样构建对象并协调MapReduce流程呢？
#手工构建相关对象，并通过某些辅助函数将这些对象联系起来。
################################################################################################

# 为目录下每个文件创建一个PathInputData对象
def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))

# 为每个PathInputData对象创建一个LineCountWorker
def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
	return workers

# 执行这些Workers,进行Map-Reduce过程，多线程执行最终汇总运行结果
def excute(workers):
    # the map process
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join() #  主线程等待工作线程结束
    
    # the reduce process
    first, rest = workers[0], workers[1:]
    for worker in rest:
        first.reduce(worker)
    return first.result

# 最后
def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return excute(workers)
```



上述代码缺点是**不够通用**，**若要编写其它的InputData或Worker子类，就必须重写generate_inputs, create_workers和mapreduce函数**，以便与之匹配。—— 要解决这个问题，必须使用一种通用的方式来构建对象，**最佳方案是使用@classmethod形式的多态**。

```python
class GenericInputData(object):
    def read(self):
        raise NotImplementedError
	# config是一份含有配置参数的字典，具体的GenericInputData子可以解读这些参数
    # 这是类方法，针对是子类而言，不针对某个对象
    # 注意第一个参数是cls, 而不是self(类方法与成员方法的区别)
    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(GenericInputData):
 	def __init__(self, path):
		super().__init__()
		self.path = path
    def read(self):
        return open(self.path).read()
    # 重写父类的类方法，创建具体的InputData实例
    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

###################################################################################
class GenericWorker(object):
    def map(self):
        raise NotImplementedError
    def reduce(self, other):
        raise NotImplementedError
    # 这个类方法，可否用静态成员函数来理解？
    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        # input_class.generate_inputs是个类级别的多态方法
        for input_data in input_class.generate_inputs(config):
            # 这里使用cls形式构造GenericWorker对象，而不是以前那样使用__init__方法。
            workers.append(cls(input_data))
        return workers

# 实现过程与以前相同
class LineCountWorker(GenericWorker):
	def __init__(self, input_data):
		self.input_data = input_data
        self.result = None
    def map(self):
		data = self.input_data.read()
        self.result = data.count('\n')
    def reduce(self, other):
		self.result += other.result
        
###################################################################################
# 执行这些Workers,进行Map-Reduce过程，多线程执行最终汇总运行结果
def excute(workers):
    # the map process
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join() #  主线程等待工作线程结束
    
    # the reduce process
    first, rest = workers[0], workers[1:]
    for worker in rest:
        first.reduce(worker)
    return first.result

###################################################################################
# 最后重写mapreduce函数，使其变得完全通用
def mapreduce(worker_class, input_class, config)通用:
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)
###################################################################################

### 测试 ###
with TemporaryDirectory() as tmpdir:
    write_test_files(tmpdir)
    config = {'data_dir': tmpdir}
    result = mapreduce(LineCountWorker, PathInputData, config)
```

**总结：**

*   python中每个类只能有一个构造器，那就是\_\_init\_\_方法。

*   通过@classmethod机制，可以用一种与构造器相仿的方法来构造类的对象。

    ​

## 第25条：用super初始化父类

初始化父类的传统方法，是在子类里用子实例(\_\_init\_\_)直接显示调用父类\_\_init\_\_方法。在多重继续下，直接调超类的\_\_init\_\_可能产生无法预知的行为。

下面的例子中说明，子类的初始化函数（\_\_init\_\__)**对各个基类的初始化函数执行顺序只取决于子类初始化内部调用顺序，而与子类定义头部中对各基类顺序无关——这可能造成岐义。**

```python
class MyBaseCls(object):
    def __init__(self, value):
        self.value = value
class TimesTwo(object):
    def __init__(self):
        self.value *= 2
class PlusFive(object):
    def __init__(self):
        self.value += 5
class OneWay(MyBaseCls, TimesTwo, PlusFive):
    def __init__(self, value):
        MyBaseCls.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)
class AnotherWay(MyBaseCls, PlusFive, TimesTwo):
    def __init__(self, value):
        MyBaseCls.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

foo = OneWay(5)
print "5 * 2 + 5 == %d" % foo.value
>> 15
foo = AnotherWay(5)
print "(5 + 5) * 2 == %d" % foo.value
>> 15
```

还有问题是出现在**钻石型继承结构（比如两个基类又是同一类的子类）：这个子类就会多次调用它两个基类的共同基类的初始化函数。**

```python
# 钻石型继承关系，多次调用上层某个基类的初始化函数(__init__)
class MyBaseCls(object):
    def __init__(self, value):
        self.value = value

class TimesTwo(MyBaseCls):
    def __init__(self, value):
        MyBaseCls.__init__(self, value)
        self.value *= 2

class PlusFive(MyBaseCls):
    def __init__(self, value):
        MyBaseCls.__init__(self, value)
        self.value += 5

class OneWay(TimesTwo, PlusFive):
    def __init__(self, value):
        TimesTwo.__init__(self, value)
        PlusFive.__init__(self, value)

foo = OneWay(5)
print "5 * 2 + 5 == 15 and is ", foo.value
# 结果错误， 因为MyBaseCls.__init__第二次调用时，又将value设置为5了 
#>> 5 * 2 + 5 == 15 and is 10
```

引用python2的**super方式解决钻石型继承问题**

**注意MRO计算顺序是从右向左的！！！**

<u>语法是： super(ThisClsName, self).xxx()</u>

```python
class MyBaseCls(object):
    def __init__(self, value):
        self.value = value

class TimesFive(MyBaseCls):
    def __init__(self, value):
        super(TimesFive, self).__init__(value)
        self.value *= 5

class PlusTwo(MyBaseCls):
    def __init__(self, value):
        super(PlusTwo, self).__init__(value)
        self.value += 2

class GoodWay(TimesFive, PlusTwo):
    def __init__(self, value):
        super(GoodWay, self).__init__(value)

# MRO计算顺序是从右向左的！！！
foo = GoodWay(5)
print "GoodWay is: 5 * (5 + 2) == 35 and is ", foo.value
#>> GoodWay is: 5 * 5 + 2 == 27 and is 35

```

python2的super必须传入本类的名称。而**python3可以<u>使用\_\_class\_\_可替代本类名称</u>**。

<u>语法是： super(\_\_class\_\__, self).xxx()</u>

```python
class MyBaseCls(object):
    def __init__(self, value):
        self.value = value

class TimesFive(MyBaseCls):
    def __init__(self, value):
        super(__class__, self).__init__(value)
        self.value *= 5

class PlusTwo(MyBaseCls):
    def __init__(self, value):
        super(__class__, self).__init__(value)
        self.value += 2

class GoodWay(TimesFive, PlusTwo):
    def __init__(self, value):
        super(__class__, self).__init__(value)


foo = GoodWay(5)
print("GoodWay is: 5 * 5 + 2 == 27 and is ", foo.value)
#>> GoodWay is: 5 * 5 + 2 == 27 and is  35
```



**总结：应该使用内置super函数初始化父类。**



## 第26条：只在使用Mix-in组件制作工具类时进行多重继承

在python中虽然可以但不鼓励多重继承，若一定要利用多重继承的便利及封闭特性，则应考虑编写mix-in类。

mix-in类是一种小型的类，它只定义其它类可能需要提供的一套附加方法，而不定义自己的实例属性，此外它也不要求使用者调用自己的\_\_init\_\_初始函数。

**也就是接口类而已**

## 第27条：多用public属性，少用private属性

以两个下划线开关的属性是private字段。

子类无法访问父类的private字段。

若既想封闭又想让子类可以访问到字段，则应该使用protected字段。（只是一种约定，即使用单下划线开头的字段)



## 第28条：继承collections.abc以实现自定义的容器类型

(仅在python3中有这个子模块)使用collections.abc模块，可以方便地编写自定义的容器类。

这里定义了一系列的抽象基类，它们提供了每一种容器所应具体的常用接口。如果自定义的容器类忘记实现这些接口，collections.abc模块会指出这些错误。

它并不会帮你实现具体的通用接口，它只是提供了一些抽象接口，确保用户编写了必备函数成员。

**仅是个接口规范！**



# 第4章 元类及属性

metaclass只是模糊地描述了一种高于类，超乎于类的概念。

## 第29条：用纯属性取代get和set方法

有时为了必要的封装或保护，可能会提供一些getter/setter函数。比如：

```python
class OldResistor(object):
    def __init__(self, ohms):
        self._ohms = ohms
    def get_ohms(self):
        return self._ohms
    def set_ohms(self, ohms):
        self._ohms = ohms
# 对于自增操作很麻烦
r0 = OldResistor(2)
r0.set_ohms(r0.get_ohms() + 3)
```



**可以使用@property修饰器和setter方法来做。**

```python
class VolateResistance(Resistor)：
	def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0

     @property
    def voltage(self): # 对应于getter接口
        return self._voltage
    
    @voltage.setter # 对应于setter接口
    def voltage(self, voltage):
        self._voltage = voltage
        self.currnet = self._voltage / self._ohms #  为简单设置操作之外添加了一点额外操作
```



## 第30条：考虑使用@property来代替属性重构

@property可以为现有的实例属性添加新的功能



## 第31条：内描述符来改写需要复用的@property方法

@property修饰器明显的缺点就是不便于复用。

若类的若干属性访问接口实现过程都一样，使用@property必须将这些过程重复编写N次，仅过程中的变量名不一样而已。

为复用这些相同的过程，python的描述符(descriptor)：python对访问操作会进行一定的转译，转译方式是按照描述符协议来确认的。**描述符可以提供\_\_get\_\_和\_\_set\_\_方法。**

```python
class Grade(object):
    def __get__(*args, **kwargs):
        # ...
    def __set__(*args, **kwargs):
        # ...
class Exam(object):
    # class attribute
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()


exam = Exam()
# 下列运行过程
exam.writing_grade = 40
# , 会被python转译为:
Exam.__dict__['writing_grade'].__set__(exam, 40)
# 获取属性
print(exam.writing_grade)
# , 会被解析为
Exam.__dict__['writing_grade'].__get__(exam)
```

之所以**有这样的转译是因为object类的\_\_getattribute\_\_方法**。参见下条。

## 第32条：用\_\_getattr\_\_, \_\_getattribute\_\_和\_\_setattr\_\_实现所需生成的属性

类的属性访问钩子函数1：**类对象的实例字典中找不到查询的属性，那么系统就会调用\_\_getattr\_\_。**

```python
class LazyDB(object):
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        # set new attribute
        value = "Value for %s" % name
        setattr(self, name, value)
        return value

data = LazyDB()
print('Before:', data.__dict__)
print(data.foo)
print('Before:', data.__dict__)
print(data.__dict__)

#>>>
('Before:', {'exists': 5})
Value for foo
('Before:', {'foo': 'Value for foo', 'exists': 5})
{'foo': 'Value for foo', 'exists': 5}
```

类的属性访问钩子函数1：**即使在类对象的实例字典中找到了查询的属性，那么系统也会先调用\_\_getattribute\_\_。**

```python
class ValidatingDB(object):
    def __init__(self):
        self.exists = 5
	# 任何时候都先调用这个函数
    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        try:
            return super().__getattribute__(name)
        except AttributeError:
            value = "Value for %s" % name
            # set new attribute
            setattr(self, name, value)
            return name

data = ValidatingDB()
print('exists:', data.exists)
print('foo:  ', data.foo)
print('foo:  ', data.foo)
# >>>
# Called __getattribute__(exists)
# exists: 5
# Called __getattribute__(foo)
# foo:   foo
# Called __getattribute__(foo) # 此时已经调用了setattr
# foo:   Value for foo
```

实现通用的功能时，我们经常会在Python代码里使用内置的hasattr函数来判断对象是否已经拥有了相关的属性，并用内置的getattr函数来获取属性值。

**若类实现了\_\_getattribute\_\_函数，那么每次在对象上面调用hasattr或getattr时，\_\_getattribute\_\_都会执行。**

**使用\_\_getattribute\_\_要特别小心：若在它的实现内部访问了self.data，则会导致自我无限循环**。

```python
class BrokenDictionaryDB(object):
    def __init__(self, data):
        self._data = data
    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        # 下面这行代码会导致无限循环
        return self._data[name]

data = BrokenDictionaryDB({'foo':3})
data.foo

#>>> 错误演示
Called __getattribute__(foo)
Called __getattribute__(_data)
Called __getattribute__(_data)
Called __getattribute__(_data)
		... ...
    
##########################################################################################

class FixedDictionaryDB(object):
    def __init__(self, data):
        self._data = data
    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        # 解决办法是采用super().__getattribute__方法，从实例的属性字典里面直接获取_data
        data_dict = super().__getattr__('_data')
        return data_dict[name]

data = FixedDictionaryDB({'foo':3})
data.foo
# >>> 修正后
Called __getattribute__(foo)
```

只要对实例的属性赋值，**无论直接赋值还是通过内置的setattr函数，都会触发\_\_setattr\_\_方法。**

**\_\_setattr\_\_方法的使用陷阱与\_\_getattribute\_\_类似，内部直接设置属性也会导致无限循环。**

**修正方法：那么应用直接通过super()来做，以避免无限循环。**



## 第33条：用元类来验证子类（略）

## 第34条：用元类来注册子类（略）

## 第35条：用元类来注解类的属性（略）



# 第5章并发与并行

* 并行：计算机确实在同一时间做很多不同的事，多CPU同时执行多个程序, 在同一时刻发生 
* 并发：计算机似乎在同一时间做很多不同的事，OS交错执行程序的方式, 在同一时间间隔发生

## 第36条 使用subprocess模块管理子进程
> *据说是最好用的子进程管理模块*   

* 用Popen启动进程，然后用communicate读取子进程的输出/错误信息, 并等待子进程终止  

```python
#src/36-1.py
#encoding=utf-8
import subprocess
proc = subprocess.Popen(['echo', 'Hello from the child'], stdout=subprocess.PIPE)
out, err = proc.communicate()
print(out.decode('utf-8'))

>>>
Hello from the child
```

*communicate(self, input=None): 与子进程进行通信, 向input发送数据，返回tuple(stdout, stderr), 从stdout和sterr获取输出。*



* 子进程会独立于父进程运行，父进程指python解释器。父进程可定期查询子进程状态，同时处理其它事务。

```python
#src/36-1.py
#encoding=utf-8
import subprocess
proc = subprocess.Popen(['sleep', '2'], stdout=subprocess.PIPE)
while proc.poll() is None:
    print('Working...')
    # some time-consuming work here 
print('Exit status', proc.poll())

>>>
Working...
...
('Exit status', 0)
```
* 父进程同时运行多个子进程

```python
#src/36-3.py
#encoding=utf-8
import subprocess
from time import time

def run_sleep(period):
    proc = subprocess.Popen(['sleep', str(period)], stdout=subprocess.PIPE)
    return proc

start = time()
procs = []
for i in range(10):
    proc = run_sleep(1)
    procs.append(proc)

for proc in procs:
    proc.communicate()

end = time()
print("Finished in %.3f seconds" % (end-start))
```

> 可以给communicate添加timeout参数，以防死锁或失去响应.

## 第37条 用线程执行阻塞式I/O, 但不要用它做平行计算
### GIL: global interpreter lock 全局解释器锁
	* 来源于python设计之初，为了数据安全的考虑
	* 每个进程只有一个GIL

**pyton多线程的执行方式**
	1. 获取GIL
	2. 执行代码直至sleep 或 python虚拟机将其挂起
		2.1 pyton2.x： 遇到IO操作或ticks计数达到100
		2.2 python3.x GIL改使用计时器
	3. 释放GIL

> 多线程频繁的获取，释放GIL带来的资源消耗，使多线程不适合CPU密集型的多任务执行。 只适合于IO密集型的多任务。

## 第38条 在线程中使用Lock来防止数据竞争

上述python多线程的执行方式第2条说明，**GIL并不保证<u>多线程</u>的IO读写若不加锁同样出现数据竞争。**
> 比如一个线程的IO代码块执行了一部分时，也会被迫让出GIL, 另一个线程获取了GIL，然后对同一数据区域的读写操作。

**使用互斥锁来保护数据结构，如Lock类, 使同一时刻只能一个线程获得锁。**  
> 另一个线程获取了GIL，但没有获取Lock同样也不能执行加锁保护的读写代码区域



## 第39条 用Queue来协调各线程之间的工作

内置的queue模块的类Queue能解决事务协作出的自编队列中的各种问题。主要因为：  
1. 自编输入队列无数据时会不断的尝试读取导致CPU利用率较低： Queue类的get()操作会阻塞住直到新数据加入
2. 自编输出队列若无法及时处理，会导致数据膨胀过多：Queue类的put()操作在达到一定数据量时也会阻塞，直到数据被消费(Queue在定义时指定大小)

```python
#src/39-1.py
#encoding=utf-8
from queue import Queue
from threading import Thread
from time import sleep

queue = Queue()

def consumer():
    print('Consumer waiting')
    queue.get()
	sleep(2)
    print('Consumer done')

thread = Thread(target=consumer)
thread.start()

print('Producer putting')
queue.put(object()) # Runs before queue.get()
thread.join()
print('Producer done')

>>>
Consumer waiting
Producer putting
Consumer done
Producer done
```

### Queue的线程管理方法：
1. Queue.join()等待所有线程结束
2. 其它线程中需要调用Queue.task_done()，用来告知主控线程来退出本身线程，否则本线程会阻塞在get()/put()方法上
  > Queue.task_done()调用之后，本线程会立即退出，不管queue中有无数据或空间  
3. 不再需要thread.join()

```python
#src/392.py
#encoding=utf-8
from queue import Queue
from threading import Thread

in_queue = Queue()

def consumer1():
	print('Consumer1 waiting')
	print('Consumer1 working')
	# doing work
	# ...
	print('Consumer1 done')
	in_queue.task_done() # done third

def consumer2():
	print('Consumer2 waiting')
	print('Consumer2 working')
	# doing work
	# ...
	print('Consumer2 done')
	in_queue.task_done() # done third

Thread(target=consumer1).start()
in_queue.put(object()) # Done first

Thread(target=consumer2).start()
in_queue.put(object()) # Done first

print('Producer waiting') 

in_queue.join() # Done fouth, 
				#X  block until all items in the Queue have been gotten and processed
				# until all consumer threads called task_done
print('Producer done')
```

## 第40条 考虑用协程来并发的运行多个函数

#### 生成器：
	1. 相对于列表生成式而言: 列表生成式一次性产生返回值，内存消耗大，容量有限。
	2. 在循环中不断推导后续元素，一边循环一边计算的机制, 称为生成器。

```python
>>> L = [x * x for x in range(10)]
>>> L是列表
>>> g = (x * x for x in range(10))
>>> g
<generator object xxxx >
>>> g.next()
>>> 0
>>> g.next()
>>> 1
>>> g.next()
>>> 4
...
# 正常的方式是使用for循环
for n in g:
	print n
0
1
4
9
...
```
#### 函数转换为生成器generator
1. 如果一个函数包含yield关键字，那么这个函数不再是一个普通函数，而是一个generator.  
2. generator与一般函数执行流程不同
  2.1 函数是顺序执行，遇到return或最后一条语句就返回。
  2.2 而generator在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回yield语句处继续执行。
3. generator执行方法：
  3.1 next不需要向生成器传递参数
  3.2 send可以传入参数
  3.3 生成器函数开始时需要执行g.send(None)或next(g)

```python
def odd():
	yield 1
	yield 3
	yield 5

>>> o = odd()
>>> print next(o)
1 #下次执行位置yield 3
>>> print next(o)
3 #下次执行位置yield 5
>>> print next(o)
5 #执行完毕
>>> print next(o) #若放在循环语句中，此时会退出循环
Traceback (most recent call last)
...
```

### 协程
协程实际上是对生成器的一种扩展。
协程执行过程中，在子程序内部可以中断，然后转而执行别的子程序，在适当时候再返回接着执行。有点类似CPU中断，与通过栈实现的函数调用不同。
yield是一个类似return的关键字，迭代一次遇到yield时就返回yield后面（右边）的值，下一次执行从yield语句的下一行执行。
优势：  
	1. 执行效率高，因为子程序切换（函数）还是线程切换，由程序自身控制，没有切换线程的开销。  
	2. 不需要多线程的锁机制，因为只有一个线程，不存在数据竞争。  
	3. 资源占用少，没有线程启动和运行的资源消耗。   
	4. 对多CPU：多进程+协程


协程的执行步骤示例：
```python
1  def minimize():
2  	current = yield
3  	while True:
4  		value = yield current
5  		current = min(value, current)
6 
7  it = minimize()
8  next(it)				# 调用next函数，将生成器推进到第一条yield表达式之前，此时并未执行第1个yield
9  print(it.send(10))  	# 执行第1个yield，将外部值给yield左值current, 并标记下次调用开始执行的位置为第3行“while True:”
10 print(it.send(4))	   # 执行到第2个yield。这个yield将外部值传给左值value, 并返回右值current,  
						   # 同时将下一次执行位置标记为这个yield的下一行（第5行）
11 print(it.send(22))	  # 执行第5行，同时左传更新为传入值4, 返回current值，又将下次执行位置标记为第5行

>>>
10
4
4
```

**python2中限制：**

1. 不支持yield from表达式，需要在另一地方，多写一层循环

**python2中需要借助循环语句替代yield from** 

```python
def delegated():
	yield 1
	yield 1

#python2
def composed():
	yield 'A'
	for value in delegated(): #必须写个循环替代yield from
		yield value
	yield 'B'
```

**python3中直接使用yield from**

```python
#python3
def composed():
	yield 'A'
	yield from delegated()
	yield 'B'
```

2.  python2的生成器不支持return语句，需要将最后的return的参数使用自定义的Exception子类包装起来，然后用try/except raise抛出。
  **python2代码的协和函数模拟return**

```python
#python 2
class MyRtn(Exception):
    def __init__(self, val):
        self.val = val

def delegated():
    yield 1
    raise MyRtn(2) # return 2 #in python 3
    yield 'Not reached'

def composed():
    try:
        for val in delegated():
            yield val
    except MyRtn as e:
        output = e.val
    yield output * 4

print list(composed())

>>>
[1, 8]
```

**在python3中直接使用return**

```python
#python 3
class MyRtn(Exception):
    def __init__(self, val):
        self.val = val

def delegated():
    yield 1
    return 2 #in python 3
    yield 'Not reached'

def composed():
    output = yield from delegated()
    yield output * 4

print(list(composed()))
```

## 第41条 考虑用concurrent.future来实现真正的平行计算

**python2.x中不自带这个库， 需要安装pip install futures**  

1. 最原始的单进程单线程形式:

```python
from time import time

def gcd(pair):
	a, b = pair
	low = min(a, b)
	for i in range(low, 0, -1):
		if a%i ==0 and b %i == 0:
			return i

numbers =   [(1963309,  2265973),   (2030677,   3814172),
			 (1551645,   2229620),   (2039045,   2020802)]
start   =   time()
results =   list(map(gcd,   numbers))
end =   time()
print('Took %.3f    seconds'    %   (end    -   start))
```

2. 使用多线程运行CPU密集的任务反而因为GIL的存在更加效率低下：  

```python
from concurrent.futures import ThreadPoolExecutor
from time import time

def gcd(pair):
	a, b = pair
	low = min(a, b)
	for i in range(low, 0, -1):
		if a%i ==0 and b %i == 0:
			return i

numbers =   [(1963309,  2265973),   (2030677,   3814172),
			 (1551645,   2229620),   (2039045,   2020802)]

start   =   time()
pool = ThreadPoolExecutor(max_workers=2)
results = list(pool.map(gcd, numbers))
end =   time()
print('Took %.3f    seconds'    %   (end    -   start))
```

3. 内置的concurrent.future模块利用另外一个名叫multiprocessing的内置模块，以子进程的形式平行地运行多个解释器，从而令python能够利用多核cpu来提升速度。    
  改用上一例子中的另一个函数ProcessPoolExcutor: 

```python
from concurrent.futures import ProcessPoolExecutor 
from time import time

def gcd(pair):
	a, b = pair
	low = min(a, b)
	for i in range(low, 0, -1):
		if a%i ==0 and b %i == 0:
			return i

numbers =   [(1963309,  2265973),   (2030677,   3814172),
			 (1551645,   2229620),   (2039045,   2020802)]

start   =   time()
pool = ProcessPoolExecutor(max_workers=2)
results = list(pool.map(gcd, numbers))
end =   time()
print('Took %.3f    seconds'    %   (end    -   start))
```

三个代码运行结果：  

	In [10]: run 411.py
	Took 0.439    seconds
	
	In [11]: run 412.py
	Took 0.579    seconds
	
	In [12]: run 413.py
	Took 0.270    seconds


## 第42条 用functools.wraps定义函数修饰器
* 有两个缺点： help(func)显示不出函数说明信息；print(func)显示是是wrapper信息而不再是原函数信息  

```python
	# the wrapper without using functools.wraps
	def trace(func):
		# define inside a nest wrapper
		def wrapper(*args, **kwargs):
			res = func(args, kwargs)
			return res
		return wrapper #return the nested wrapper
	
	# usage the decorator: 
	@trace
	def func():
		'''this is the func help msg'''
		return 2
	
	# 缺点1 cannot print orginal func info
	print(func)
	>>>
	<function wrapper at 0x7f5c9b122500> #打印的是wrapper而不是function
	
	# 缺点2 cannot print func help str
	help(func)
	>>>
	# 没有内容
	Help on function wrapper in module __main__:
	wrapper(*args, **kwargs)
		# define iniside a nest wrapperjkk
```
* 使用functools重修正以下俩问题  
  **functools.wraps修饰器会把内部函数相关的重要元数据全部复制到外围函数**

```python
+	from functools import wraps
	# the wrapper using functools.wraps
	def trace(func):
+		@wraps
		def wrapper(*args, **kwargs):
			res = func(args, kwargs)
			return res
		return wrapper #return the nested wrapper
	
	# usage the decorator: 
	@trace
	def func():
		'''this is the func help msg'''
		return 2
	
	print(func)
	>>>
	<function func at 0x7f9d69006488>	

	help(func)
	>>>
	Help on function func in module __main__:

	func(*args, **kwargs)
		this is func help str
		
```

## 第43条 使用contextlib和with语句来改写可复用的try/finally
* 使用With,比使用try/finally更简洁, 省去了后者一些重复代码。
  *前提是类支持with语句*  

```python
lock = Lock()
with lock:
	print "lock is held"

#改成try/finally
lock = Lock()
lock.acquire()
try:
	print "lock is held"
finally:
	lock.release()
```

* 常规方法是：把with后面的值定义新类的对象，为类定义特殊方法：__enter__和__exit__

```python
class ExprCls(object):
    def __enter__(self):
        print "will enter with"
    def __exit__(self, exc_type, exc_val, exc_tb):
        print "will exit with"

if __name__ == '__main__':
    expr = ExprCls()
    with expr:
        print "     inside with"
    print "     outside with"

>>>
will enter with
     inside with
will exit with
     outside with
```

* **内置的contextlib模块提供了名叫contextmanager的修饰器**，可使函数支持with
  修饰器里的函数通过yield向with语句返回一个值，赋值给as后的变量

```python
from contextlib import contextmanager

@contextmanager
def expr():
    try:
        print "will enter with"
        yield
    finally:
        print "will exit with"

if __name__ == '__main__':
    with expr() as exp: # exp will be None, cuz nothing behind yield
        print "     inside with"
        if exp is None:
            print("exp is None")
        else:
            print(expr(exp))
    print "     outside with"

>>>
will enter with
     inside with
exp is None
will exit with
     outside with
```

```python
@contextmanager
def log_level(level, name):
	logger = logging.getLogger(name)
	old_level = logger.getEffectiveLevel()
	logger.setLevel(level)
	try:
		yield logger
	finally:
		logger.setLevel(old_level)

#使用with+as，将log_level的调用限定在局部
with log_level(logging.DEBUG, 'my-log') as logger:
	logger.debug('This is debug msg WILL print')
logger.debug('This is debug msg BUT will NOT print')
```
* 更普遍的用法

```python
with open('xxx') as handle:
	print handle.read()
```

## 第44条 用copyreg实现可靠的pickle操作
内置的pickle模块将python对象序列化及反序列化, 在程序之间传递python对象  
1. 通过copyreg模块注册xx函数，使类**新加的字段**具有默认值。
  *要求后来增加的字段在__init__中具体默认值*  
```python
	# 使用情景： GameState类后来又加了新字段，导致反序列化失
	def pickle_game_state(game_state):
		kwargs = game_state.__dict__
		return unpickle_game_state, (kwargs,)
	# 定义 unpickle_game_state这个辅助函数，它是对构造函数GameState.__init__的简单封装
	def unpickle_game_state(kwargs):
		return GameState(kwargs)
	# 注册pickle_game_state函数
	copyreg.picke(GameState, pickle_game_state)
	# 要求构造函数__init__新加的字段具有默认值如：new_para = 2
```

2. 用版本号来管理类, 处理类后来**删除某些字段** 
  * 从现在类中删除某些字段，会导致新类接受多余参数，导致新类与旧类不兼容

```python
	# 使用情景： GameState加了新字段，导致反序列化失败。
	def pickle_game_state(game_state):
		kwargs = game_state.__dict__
	  + # 修改步骤1注册的xx函数，在xx中添加版本信息version参数
	  + kwargs['version'] = 2
		return unpickle_game_state, (kwargs,)
	# 定义 unpickle_game_state这个辅助函数，它是对构造函数GameState.__init__的简单封装
	def unpickle_game_state(kwargs):
	   + version = kwargs.pop('version', 1)
	   + if version == 1:
	   + 	kwargs.pop('new_para')
	   + return GameState(kwargs)
	# 注册pickle_game_state函数
	copyreg.picke(GameState, pickle_game_state)
```

3. 修正类更名后无法反序列化
  * 换名之后会导致反序列化时找不到原来的类（被更名了）
  * copyreg.picker(NewGameState, xx)


## 第45条 应该用datetime模块，而不是time模块处理本地时间
python提供了两种处理时间的模块：
* time模块依赖OS（时区）运行, 由于时区和夏令时间等问题，它处理时间比较麻烦。
* datetime模块 借助pytz模块基于UTC操作 可以比较方便的处理时区转换等问题，因此处理时间转换时优先考虑
* 应把时间表示成UTC格式，然后对其执行各种转换操作，最后再把它转回本地时间。


## 第46条内置算法和数据结构

*  数据结构
  * 双向队列，可以在头尾才能常数级别的插入删除数据，而list只能在尾部达到这样的性能
  ```python
  fifo = deque()
  fifo.append(1)
  x = fifo.popleft()
  ```
  * 有序字典保留插入顺序，普通dict的key是基于快速哈希，在相同内容的dict迭代可能出现不同的输出顺序。  
  ```python
  a = OrderDict()
  a['foo'] = 1
  a['bar'] = 2
  b = OrderDict()
  b['foo'] = 'red'
  b['bar'] = 'blue'
  for v1, v2 in zip(a.values(). b.values()):
  	print(v1, v2)
  >>>
  1 red
  2 blue
  ```
  * 带有默认值的字典
  ```python
  stats = defaultdict(int) # default int value is 0
  stats['my_counter'] += 1
  ```
  * 堆队列（优先级队列） heapq模块提供了heappush, heappop, nsmallest等函数，能在标准的list类型创建堆
  ```python
  a = []
  heappush(a, 5)
  heappush(a, 3)
  heappush(a, 7)
  heappush(a, 4)
  #a此时已经是顺序排列的队列了
  a
  >>>
  [3, 4, 5, 7]
  print( heappop(a), heappop(a), heappop(a), heappop(a))
  >>>
  3 4 5 7
  # 返回最N小的列表
  print(nsmallest(1, a))
  >>>
  [3]
  ```
  * 二分查找bisect模块， bisect_left(x, 7788) 返回待搜索值在序列中的插入点  
  ```python
  # list的查找时间与长度成正比
  x = list(range(10**6))
  i =  x.index(7788)
  # bisect二分查找, 对数级别时间复杂度
  from bisect import bisect_left
  i = bisect_left(x, 777888)
  ```


* 与迭代器相关的工具 itertools模块
  * 连接迭代器
    * chain 连成一个大的迭代器
    * cycle 无限循环替代器中元素
    * tee 产生n个相同的迭代器
    ```python
    i1, i2 = tee('abcdefg', 2)
    print(list(i1))
    print(list(i2))
    ```
    * zip_longest 只有python3有
    ```python
    # python 3
    print(list(zip('aaa', '2222')))
    >>>	[('a', '2'), ('a', '2'), ('a', '2')]
    print(list(zip_longest('aaa', '2222')))
    >>>	[('a', '2'), ('a', '2'), ('a', '2'), (None, '2')]
    ```
    * count(start=0, step=1) 返回从start开始的迭代器
  * 过滤迭代器
    * islice 切片操作 
      * islice(iterable, stop) 或 islice(iterable, start, stop, step)
    * dropwhile: dropwhile(条件判断函数，iterable), 返回为假的迭代器
      * takewhile: 与上相反
    * ifilter函数: python2有， python3, 与takewhile类似，但多了ifilter(None, iterable)，此时返回iterable中的真值，例如从range(10)中去掉了0值
    * ifilterfalse函数: python3有， python2无, 与dropwhile类似，但多了filterfalse(None, iterable)
    * compress(data, selections): compress('abcdef', [1,0,1,1,0,0]) ==> 'acd'
    * izip 类似zip函数，但返回是iterable: list(izip('abc', '123')) ==> a1 b2 c3
    * izip_longest类似于zip与zip_longest, 不够用None填充
    * groupby(iterable, 归类条件函数)
    ```python
    for i , k in groupby(['aa', 'bb', 'aaa', 'bbb'], len):
    	print i, list(k)
    >>> 
    	2 ['aa', 'bb']
    	3 ['aaa', 'bbb']
    ```
  * 组合迭代器
    * product(*iterable[, repeat=x]) 返回迪卡尔积 product('ab', '12') ==> a1 a2 b1 b2，若参数多了repeat=x，则表示参数repeat之前的元素是x倍，可以认为是一种参数简写形式
    * permutations(iterable, [, r]) ,返回长度为r的iterable元素的组合索引，若r省略则表示长度为长度等于iterable
    * combination(iterable, r) 返回长度为r的子序列

## 第47条 在重视精度的场合使用decimal
自带模块decimal的Decimal比普通浮点数的优点是：
	1. 精度高，据说是28位，精度Decimal > python3 > python2
		若Decimal精度还不够用，使用Fraction，它包含于fractions模块中。
	2. 可自定义四舍五入, Decimal对象内置函数quantize(Decimal('0.7788'), rounding=ROUND_UP)

## 第48条 安装pip模块
	pip freeze
	pip install xxx=1.0.0
	pip uninstall xx

## 第49条 为每个函数，类和模块编写文档字符串
* 模块文档
* 类文档， 类成员函数文档
* 函数文档
* 以上都可以通过__doc__属性来访问

## 第50条 用包来安排模块，并提供稳固的api
* 包的构成
  * 把__init__.py放在源文件目录下，就构成一个包
  * 目录中的文件成为包的子模块
  * 包的目录下可以包含其它包, 包就是包含其它模块的模块

*  包的两大用途
  * 名称空间
  * 稳固的API
    > 为包或模块编写名为__all__的特殊属性，来减少对外暴露的API  
    > 只有__all__中出现的名称才能被其它模块导入  

## 第51条 为自编的模块定义根异常，以将调用者与API相隔离 
**模块所抛出的异常，同函数与类一样，都是接口的一部分**
首先定义一个根异常，直接派生于Exception
```python
# my_module.py
class Error(Exception):
	"""Base-class for all exceptions raised by this module"""
```
调用者通过try/except语句来捕捉本模块产生的任何异常，这有三个好处：
1. 调用者可根据捕获的本模块抛出的异常确认是否对本模块正确调用
2. 若本模块抛出了根异常之外的异常，则说明本模块代码有问题, 这需要在根异常之后再加一个
  > exception Exception as e:
3. 便于本模块异常逻辑的细化编写, 继续编写更具体的异常子类


## 第52条 用适当的方式打破模块之间的循环依赖关系(导入依赖)
python在执行import时，
	1. 在由sys.path所指定的路径中，搜寻待引入的模块
	2. 从模块中加载代码, 检查语法
	3. 创建与对应的空模块对象
	4. 把这个空的模块对象添加到sys.modules里
	5. 运行模块对象中的代码，以定义其内容

**第5步决定了import时是深度优先策略执行**    

```python
#dialog.py
import app

class Dialog(object):
    def __init__(self, save_dir):
        self.save_dir = save_dir
    # ...

save_dialog = Dialog(app.prefs.get('save_dir'))

def show():
    # ...
    pass

	##############################################

# app.py
import dialog

class Prefs(object):
    # ...
    def get(self, name):
        # ...
        pass

prefs = Prefs()
dialog.show()

>>> import app
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
----> 1 import app

/hda1/STUDY/python/effective-python/521/app.py in <module>()
      1 # app.py
----> 2 import dialog

/hda1/STUDY/python/effective-python/521/dialog.py in <module>()
----> 9 save_dialog = Dialog(app.prefs.get('save_dir'))

AttributeError: 'module' object has no attribute 'prefs'

###########################################
出错原因： 我的理解有无问题？？？
	1. 先import app, app作为空模块对象添加到sys.modules里, 然后执行1,2,3,4, 然后执行步骤5, 包含代码:
		import了dialog
	2. app 又 import了dialog, 由于是import的机制是深度优先导入，因此dialog经过了1,2,3,4, 进而执行步骤5,包含代码:
		save_dialog = Dialog(app.prefs.get('save_dir'))
	3. 第2步试图读取空模块对象内容app.prefs, 执行出错
		AttributeError: 'module' object has no attribute 'prefs
```

 循环依赖出现的原因是若模块进行到第4步就可以被其它模块import到，但此模块的内容只
1. 调整引用顺序, 让app.py模块代码执行，先填充内容，再import其它可能访问它内容的模块。

```python
# app.py
-import dialog
class Prefs(object):
	# ...

prefs = Prefs()

+import dialog # Moved
dialog.show()
```

	> 缺点是容易出现问题，而且不符合PEP8风格

2. 只在模块中给出函数，类和常量的定义，而不运行这些函数。
  每个模块提供configure函数，等其它模块都引用完毕后，再在该模块上面调用一次configure，而这个configure函数则会访问其它模块的属性
  > 缺点是需要每个模块提取出configur步骤，代码变的复杂

3. 动态import，在函数内部使用import, 在需要时再import。 
  > 缺点是有一定执行开销；在循环中会有反复import; 异常时调试不方便。

## 第53条 用虚拟环境隔离项目，并重建其依赖关系
pip安装默认是全局的，安装/升级可能导致各个软件包之间的依赖问题。  
将全局pip软件包迁移到virtualenv步骤：   
1. 在全局执行pip freeze > requirements.txt
2. 在virtualenv中执行 pip install -r requirements.txt


# 第8章 部署

## 第54条 考虑用模块级别的代码来配置不同的部署环境 
使用sys和os模块，或者通过全局变量来判断开发还是产品部署环境，然后在模块执行不同的代码。

## 第55条 通过repr字符串来输出调试信息
1. 直接打印内置的python类型，不能显示类型信息.例如：不能区分'5'和5
2. 对内置类型，repr有反操作eval函数
3. print("%r") 会调用repr函数，print("%s") 会调用str函数，如果没有__str__则%s也会调用repr函数
4. 自定义类中编写__repr__方法，定义更详细的调试信息
5. 可对任意对象查询__dict__属性，以观察其内部信息

## 第56条使用unittest模块来测试全部代码
1. 定义TestCase子类，在中针对每个待测试的行为定义测试方法，测试方法以test开头，它们使用的辅助函数不能以test开头
2. TestCase子类 setUp和tearDown可以准备测试环境和清理测试环境，比如临时目录文件等。
3. 必须同时编写单元测试和集成测试，单元测试李代桃僵程序子功能，集成测试3. 必须同时编写单元测试和集成测试，单元测试李代桃僵程序子功能，集成测试3. 必须同时编写单元测试和集成测试，单元测试检验程序中的子功能，集成测试则检验模块之间的交互行为。


## 第57条 使用pdb进行交互调试
1. import pdb之后，调用set_trace()打上断点，然后再运行代码
2. 主要命令
  * bt 查看调用堆栈
  * up 回到堆栈上一级
  * down： 进入下一级堆栈
  * step (s) 进入被调函数
  * finish (f) 跳出被调函数，回到上层调用函数
  * next (n) 下一条语句, 不进入被调函数
  * return (r) 继续运行，跳出调试过程
  * continue (c) 继续运行到下一断点

## 第58条 先分析性能再优化 
1. 先做性能分析，使用cProfile模块的runcal方法来分析程序的性能，它会按函数调用关系单独统计每个函数的运行信息
```python
import cProfile as Profile
profile = Profile()
profile.runcall(test_func, datax)
```
2. 然后使用stats对象来筛选性能分析数据，并打印上一步的分析结果
```python
from pstats import Stats
stats = Stats(profiler)
stats.strip_dirs() # 去掉无关的路径信息
stats.sort_stats('cumulative').print_stats() #按累计运行时间排序
stats.sort_stats('name').print_stats() #按函数名
```
> ncalls 调用次数  
> cumtime: 累计时间，包含它调用的其它函数破费的时间  
> **tottime**: 累计时间, 不包含它调用其它函数耗费的时间  
> **直接python -m profile xxx.py** 可打印以上这些信息  

## 第59条 使用tracemalloc来掌握内存使用和泄露情况
在python的默认实现方式cpython中，内存管理是通过引用计数处理的。有时会保留太多的引用导致内存耗尽。
* 使用gc模块 获取当前内存使用状态， 缺点是信息不够详细
* python3.4内置tracemalloc, 在python2.x则需要第三方如pytracemalloc

---


