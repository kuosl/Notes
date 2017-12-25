## 手工实现示例：
```python
raw_input = input("Prompt>")
if input == 'command_1':
	command_1()
elif input == 'command_2':
	command_2()
# more commands here...
else:
	print "Error"
```

## 使用自带cmd模块
```python
from cmd import Cmd

class MyPrompt(Cmd):

	def do_hello(self, args):
		"""Says hello. If you provide a name, it will greet you with it."""
		if len(args) == 0:
			name = 'stranger'
		else:
			name = args
		print "Hello, %s" % name

	def do_quit(self, args):
		"""Quits the program."""
		print "Quitting."
		raise SystemExit

if name == 'main':
  prompt = MyPrompt()
  prompt.prompt = '> '
  prompt.cmdloop('Starting prompt...')
```

## 运行效果：
> Starting prompt...
> help
> Documented commands (type help <topic>):
> hello  help  quit
> help hello
> Says hello. If you provide a name, it will greet you with it.
> hello austin
> Hello, austin

## 优点:
- 方便编写帮助信息
- 全集行历史管理
- 其它。。。

