# 编写高质量python的建议

## 1.理解pythonic概念

### pythonic的定义

充分体现Python自身 特色的代码风格

### 代码风格

1.灵活的使用迭代器，又或者安全的关闭文件描述符使用with

```python
for i in alist:
	do_sth_with(i)
    
with open(path, 'r') as f:
	do_sth_with(f)

```

### 标准库

1.写Pythonic程序需要对标准库有充分的理解，特别是内置函数和内置数据类型

2.例如：字符串格式化，一般写法

```python
print 'Hello %s!' % ('Tom',)

#更python的写法
value = {'greet': 'Hello world', 'language': 'Python'}
print '%(greet)s from %(language)s.' % value

#最推荐的字符串格式方法，使用format
print '{greet} from {language}.'.format(greet = 'Hello world', language = 'Python')

```

### 编写自定义的库或者框架

1.包和模块的命名采用小写、单数形式，而且短小

2.包通常仅作为命名空间，如只包含空的__init__.py文件

## 2.编写pythonic代码

### 避免恶劣化代码

1.避免只用大小写来区分不同的对象，比如毫不相干的a和Ａ,建议变量名直观表示变量含义

２.避免使用容易引起混淆的名称，名称需要反应意义，如实例二优于实例一

```python
#实例一
def funA(list,num):
	for element in list:
        if num==element:
			return True
		else:
			pass
#实例二
def find_num(searchList,num):
	for listValue in searchList:
		if num==listValue:
			return True
		else:
			pass

```

3.为了方便代码的可读性变量名过长是可以接受的

### 深入理解python有利于编写pythonic代码

## 3.理解python与C语言的不同之处

1.分别使用python与{}分割代码块

2.’‘，’‘’‘，一个为字符一个为字符串，但是在python中没有区别

3.python中的三元操作符

```python
print x if x<y else y
```

4.switch...case在python中不存在，但是可以使用跳转表实现

```python
def f(x):
    return {
    0: "You typed zero.\n",
    1: "You are in top.\n",
    2: "n is an even number\n"
    }.get(n, "Only single-digit numbers are allowed\n"
```

## 4.在代码中适当添加注释

### 建议写法

1.使用块或者行注释的时候仅仅注释那些复杂的操作、算法，还有可能 别人难以理解的技巧或者不够一目了然的代码

2.注释和代码隔开一定的距离，同时在块注释之后最好多留几行空白再 写代码

3.给外部可访问的函数和方法（无论是否简单）添加文档注释

4.推荐在文件头中包含copyright申明、模块描述等，如有必要，可以考 虑加入作者信息以及变更记录

### 错误实例

1.代码即注释（不写注释）

2.注释与代码重复。注释应该是用来解释代码的功能、原因以及想 法的，而不是对代码本身的解释

3.利用注释语法快速删除代码。对于不再需要的代码，应该将其删除，而不是将其注释掉。

## 5.通过适当添加空行使代码布局更为优雅、合理

代码不是恒定的，只有风格才 能延续，能工作的代码和整洁、优雅的格式同样重要
<<<<<<< HEAD

1.在一组代码表达完一个完整的思路后，应该用空白分隔，即不要再用一段代码说明多件事，

2.尽量表示上下文语意的易理解性，相关的逻辑代码放在一起

3.避免过长的代码行，过长的部分可以使用括号进行连接，并保持连接的元素对齐

4.不需要为了对齐强行使用空格

## 6.编写函数的原则

1.函数设计尽量短小，避免长函数，避免上下拖动查看函数，在使用for,if,while等地方尽量控制在三层，避免嵌套过深

2.函数申明应该做到合理、简单、易于使用，函数名反应函数功能时，参数尽量也简洁明了

3.函数在升级过程中需要不断更改，设计时尽量兼容以后的版本，尽量不更改接口

4.一个函数只做一件事，为此在设计时尽量使抽象层级一直，尽量保证函数语句粒度的一致性

5.不要在函数中定义可变对象作为默认 值，使用异常替换返回错误，保证通过单元测试等

## 7.

=======
>>>>>>> cffa11115f6dbb5a56a34a2050fff178512ebbc0
