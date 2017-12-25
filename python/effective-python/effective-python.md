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
	• 直接判断列表是否真值来判断列表是为空，如应if a: 而不是if len(a) == 0  
	• 不要编写单行的 if for while except语句  
	• import 语句始终諈文件开头  
	• 引入模块时，应总使用绝对名称，而不应使用根据当前模块的路径而使用相对名称，如：引入foo包中bar模块，应完整写出from bar import foo, 而不应简写import foo  
	• 若非要相对名称来import， 则应from . import foo  
	• import语句分为三块：1标准库模块 2第三方模块 3自用模块   


## 第3条 了解bytes, str, unicode  
| python版本 | 含原始8位值| 含unicode字符 |  
| -------	|	------	|	----	|
| python3	|	bytes	| str	|
| python2	|	str	| unicode	|


* 两个python版本中， unicode并没有与特定二进制编码关联
* 到8位值转换是encode，到unicode转换是decode
* 一定要把编解码操作写在核心代码最外围。核心代码应使用unicode，且不要假定字符编码类型。  
* python2中7位值的ascii值时，unicode等价str， 而在python3中， bytes与str绝不等价
* python3中内置open函数默认使用utf-8编码，而python2中默认是二进制, 因此在python3中操作二进制文件应该指明'wb'或'rb'

## 第4条 辅助函数取代复杂表达式
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

## 第6条 音效切片操作内，还要同时指定start, end和stride
容易变得费解  

## 第7条 用列表推导来取代map filter
```python
# 列表推荐比较简洁   
a = [1,2,3,4]
squres = [x ** 2 for x in a]
squres =  map(lambda x: x**2, a)
squres = [x ** 2 for x in a if x%2==0]
squres =  map(lambda x: x**2, filter(lambda x: x%2==0, a))
```
* 对字典dict和集set，也支持列表推荐

## 第8条 不要使用含有两个以上表达式的列表推导
## 第9条 用生成器表达式来改写数据量较大的列表推导
**列表推导较耗内存**
```python
# 很耗内存
value = [len(x) for x in open('/tmp/myfile.txt')]
print(value)
```
* 把实现列表推导所用的那种写法放在一对圆括号内，就构成了生成器表达式。  
* 生成器表达式在求值时会立即返回一个迭代器，而不会深入处理文件中的内容。  
```python
it = (len(x) for x in open('/tmp/myfile.txt'))
print(it)
>>>
<generator object ... at ...>
#使用next函数操作上步返回的迭代器
print(next(it)
print(next(it)
...
```

## 第10条：尽量使用enumerate取代range
内置的enumerate函数可以把各种迭代器包装为生成器，以便稍后产生输出值。
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
1. python2中的zip并不是生成器，它会把所有迭代器都平等的遍历一遍。因此需要使用内置模块itertools的izip函数
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
	```

可能使用人误解为：若循环没有正常执行才执行else. 
而实际是循环中有break(即使是在最后一步break)，else才不会执行
	```python
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
	```
在for x in []: 或 while False: 之类的循环后的else会马上执行



## 第13条：
异常处理考虑四种不同的时机：try/except/else/finally
1. finally块: 既要将异常向上传播，又要在异常发生时执行清理工作
	```python
	handle = open('/tmp/random_data.txt') # 可能抛出IOError
	try:
		data  = handle.open()			# 可能抛出UnicodeDecodeError
	finally:
		handle.close()				    #  always runs after try
	```
2. else块: try块没有发生异常，就执行else块
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

## 第14条： 尽量用异常来表示特殊情况。而不要返回None
* 使用None这个返回值来表示特殊意义的函数，容易使用调用者犯错，因为None和0及空字符串之类的值，在条件表达式都会评估为False
	* 除非返回值为无组，比如：第一个返回值为状态，第二个返回值才是结果，但这种方式使用不太直观
* 函数在遇到特殊情况时，应该抛出异常，而不要返回None。调用者看到异常后，应该编写相应代码处理它们。

## 第15条： 了解如何在闭包里使用外围作用域的变量
在表达式中引用变量时，python解释器将按以下顺序查找：
	1. 当前函数作用域
	2. 任何外围作用域（例如，包含当前函数的其它函数）
	3. 包含当前代码的模块的作用域（也叫全局作用域，global scope)
	4. 内置作用域(也就是包含len及str等函数的作用域)
	5. 若没找到则抛出NameError异常

给变量赋值时，规则有所不同：  
	**在当前模块中若无定义，则在当前模块内定义这个变量，不会去上级（或外围）作用域去查找。**
	目的是防止函数的每个赋值操作污染外围作用域
示例代码：  

```python
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
Found: False 
[2, 3, 5, 7, 1, 4, 6, 8]
```
python3中，要想获取闭包内的数据，可以在内层空间中用nonlocal语句声明。

```python
def sort_priority3(values, group):
	"""sort values by prioriy group"""
	found = False
	def helper(x):
		nonlocal found
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
与nonlocal类似的是global语句声明，前者声明的是闭包的外围函数中的变量，而后者声明的是模块的全局变量。
python2也支持global语句。
python2不支持nonlocal语句，若想达到类似效果，可用以下技巧：

```python
def sort_priority(values, group):
	"""sort values by prioriy group"""
	found2 = [False,]
	def helper(x):
		if x in group:
			found2[0] = True #像是在强制的读取
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
最后，为了代码可读性，尽量少用global及nonlocal

## 第16条： 考虑用生成器来改写直接返回列表的函数
	1. 使用生成器有时结果更清晰:直接使用yield返回结果
	2. 节省资源，每次调用才产生一个结果，不必一下子把所有结果计算出来
	3. 不方便之处就是不能重复调用,因为每次调用会改变内部某些状态。

## 第17条： 在参数上面迭代时，要多加小心

## 第18条：
## 第19条：
## 第20条：
## 第21条：
## 第22条：
## 第23条：
## 第24条：
## 第25条：
## 第26条：
## 第27条：
## 第28条：

# 第4章 元类及属性

## 第29条：
## 第30条：
## 第31条：
## 第32条：
## 第33条：
## 第34条：
## 第35条：


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

上述python多线程的执行方式第2条说明，GIL并不保证多线程和的IO读写若不加锁同样出现数据竞争。
> 比如一个线程的IO代码块执行了一部分时，也会被迫让出GIL, 另一个线程获取了GIL，然后对同一数据区域的读写操作。

使用互斥锁来保护数据结构，如Lock类, 使同一时刻只能一个线程获得锁。  
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

**请大家把意见发到slguo@fortinet.com**

---
TODOLIST:
- [ ] **收集内部培训意义，归纳整理** 
- [ ] 补充Chpt10~35

