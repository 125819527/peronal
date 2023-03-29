# asyncio协程

## 迭代器

1.可迭代：实现__iter__(iterable)称为可迭代对象，调用iter来构建迭代器(iterator)

2.同时满足__iter__与__next__则满足迭代协议，根据协议迭代器必须是可迭代的，即迭代器是一种可迭代对象

3.多次调用next(iterator)来获取值

4.最后捕获StopIteration异常来判断迭代结束



 注：可迭代对象满足iter返回可迭代对象，可以是本身也可以是其他迭代器，而迭代器需要同时满足iter与next

### 自定义迭代器

- 初始化时传入可迭代对象，用于取数据
- 初始化迭代进度
- 调用__next__时有元素迭代
  - 有则返回迭代元素，同时更新迭代
  - 无则迭代结束，抛出stopiteration异常

 

```python
#类似实现range功能，for后面对象必须是可迭代对象
class myiterator():
    def __init__(self, max_index):
        self.index = -1
        self.max_index = max_index

    def __iter__(self):
        return self

    def __next__(self):
        self.index += 1
        if self.index < self.max_index:
            return self.index
        else:
            raise StopIteration

for i in myiterator(3):
    print(i)
```

```python
#这种写法则不是可迭代对象，for会报错
class myiterator():
    def __init__(self, max_index):
        self.index = -1
        self.max_index = max_index

    # def __iter__(self):
    #     return self

    def __next__(self):
        self.index += 1
        if self.index < self.max_index:
            return self.index
        else:
            raise StopIteration


myiterators = myiterator(3)
for i in myiterators:
    print(1)

```

### 迭代器的意义  

#### 存在两种可迭代对象

- 只实现__iter__,需要返回额外的迭代器支持，支持多次迭代，每次都是全新的迭代器
- 同时实现iter,next,只能迭代一次，每次返回都是本身同一个迭代对象 

#### 迭代器中的iter对象的作用

1.在list,str等方法都只包含iter，返回包含iter与next的可迭代对象，即对于使用for循环，迭代器的iter非必须

 2.迭代器实现iter，与循环解耦，即一个可迭代对象可以构建任意多个不同的迭代器（即主要由实现iter的可迭代对象构建迭代器，每一次都是不同的对象），一种迭代器可以应用于任意多个可迭代对象

#### 迭代器的应用

1.很多迭代器串联，形成处理数据的管道，称为数据流，一次只通过一份数据，避免一次性加载所有数据流

2.迭代器开始承担数据处理责任，通过迭代器获取数据，原理数据存储

```python
#放弃循环，处理数据，实现了生成器
class Random():
    def __iter__(self):
        return self

    def __next__(self):
        return random()
```

## yield关键字定义生成器

### yield关键字

1.只能用在函数内

2.使用后函数会变为生成器函数，加()执行只会返回生成器对象 

3.将含有yield的函数称为生成器函数，器调用的返回结果称为生成器

```python
def gen():
    print(1)
    if e:
        yield
g=gen()
print(type(g))
<class 'generator'>
```

#### 对函数的修改

1.yield修改了生成器函数的性质，调用生成器函数不执行其中代码，而是返回对象

2.生成器内部的代码需要生成器对象，作用过类似于类、

```python
def gen(meet):
    print(1)
    if meet:
        print(2)
        yield 666
        print(3)
    print(4)
    return 5
#生成器
g=gen(True)
print(next(g))
#1 2 666 ，返回遇到yield被暂停
print(next(g))#next再次激活
#StopIteration: 5,return的值被异常带回
```

#### 生成器的状态

1.调用生成器函数得到对象，对象处于初始

2.使用next()调用生成器对象，对应的生成器函数代码开始运行，处于运行态

3.遭遇yield，语句右边作为返回值，生成器在yield暂停，再次next时，从记录位置运行

4.结束抛出stopiteration异常，返回值只能作为异常的值抛出，对已结束的调用next,直接返回stopiteration，不含返回值  

## 生成器进化为协程

### 函数的运行机制

定义函数，得到函数对象

```python
def func():
    x=1
    print(x)
func

<function __main__.func()>
```

函数中的代码保存在代码对象

```python
func.__code__

<code object func at 0x0000025F7E7FB660, file "C:\Users\W6347\AppData\Local\Temp\ipykernel_55200\2361538272.py", line 1>
```

代码对象随着函数对象一起创建，是函数对象一个重要属性，代码对象中的属性以co_开头

```python
for attr in dir(func_code):
    if attr.startswith('co_'):
        print(f'{attr}\t:{getattr(func_code,attr)}')
```

```python
co_argcount	:0
co_cellvars	:()
co_code	:b'd\x01}\x00t\x00|\x00\x83\x01\x01\x00d\x00S\x00'
co_consts	:(None, 1)
co_filename	:C:\Users\W6347\AppData\Local\Temp\ipykernel_55200\2361538272.py
co_firstlineno	:1
co_flags	:8259
co_freevars	:()
co_kwonlyargcount	:0
co_lnotab	:b'\x00\x01\x04\x01'
co_name	:func
co_names	:('print',)
co_nlocals	:1
co_posonlyargcount	:0
co_stacksize	:2
co_varnames	:('x',)
```

函数对象和代码对象保存函数的基本信息，函数运行时，产生帧对象保存运行时的状态，每次调用函数自动产生一个帧对象，记录当次运行的状态

```python
import inspect#获取函数时的运行帧
def foo():
    return inspect.currentframe()
f1=foo()
f1

<frame at 0x0000025F7DD1D510, file 'C:\\Users\\W6347\\AppData\\Local\\Temp\\ipykernel_55200\\1887293945.py', line 3, code foo>
```

```python
from objgraph import show_backrefs
show_backrefs(foo.__code__)
#查看函数对象，代码对象，帧对象之间的关系
```

![image-20220814223232142](asyncio协程.assets/image-20220814223232142.png)

### 函数运行帧

函数多个调用，因此程序运行期存在多个帧对象，函数调用关系先执行后退出，帧对象之间的关系也先入后出，因此运行帧称为栈帧，一个线程只有一个函数运行栈

### 生成器函数有何不同

1.调用生成器函数不会直接运行，即不会创建帧对象并且压入函数栈，而是得到生成器对象

 2.生成器自身封装一个帧，并不是调用时生成，使用next迭代时，都使用该帧保存状态

```python
def gen_foo():
    for _ in range(10):
        yield inspect.currentframe()
gf = gen_foo()
gi_frame=gf.gi_frame
frames=list(gf)
print(gf.gi_frame)
for f in frames:
    print(f is gi_frame)
#在运行中生成器一直使用一个帧
```

3.next调用时生成器对象入栈，返回时帧出栈，迭代结束后被销毁

### 同步与异步

##### 调用普通函数

1.调用函数，构建帧对象并入栈

2.运行结束，帧对象出战并销毁

##### 生成器函数

1.创建生成器，构建帧对像

2.通过next触发执行，帧入栈

3.遇到yield：帧出栈，将出入栈分开

4.迭代结束，帧出栈并销毁

### 总结

1.数据的迭代器：针对一个包含很多元素的数据集，逐个返回其中的元素

2.生成器迭代器：针对一个包含很多代码的函数，分段执行其中的代码

3.让一个函数多次迭代执行代码是生成器迭代器作用，将重点集中为迭代执行的代码，就是协程

## 基于生成器的协程

### yield表达式

1.作为表达式，则可以解析一个值

2.使用next驱动生成器时，yield表达式总为None,因为next不接受入参

3.send方法接受入参，在生成器回复运行时，将入参作为yield表达式的值

4，对于刚创建好的生成器，第一次总需要send(None),使其在yield除暂停，称为prime

```python
def show():
    print('开始')
    x=yield
    print(x)
g=show_yield_value()#刚创建生成器不属于运行状态
g.send(None)#即第一个必须None值，或者next(启动)
```

5.send是为了协程增加的api，即将生成器视为协程，用send，视为迭代器，用next

6.生成器作为迭代器时，结束在元素迭代完毕后，而作为协程是一个任务，新增close()结束协程,通过传入异常

```python
def task():
    yield 1
    yield 2
    yield 3
a=task()
a.send(None)
a.close()
a.send(None)

#报错，StopIteration:
```

7.结束协程也是使用异常实现

```python
def gen():
    while True:
        try:
            x=yield
         except GeneratorExit:
            print(1)
            return #跳出循环，终结协程
        else:
            print(x)
gen().send(None)#激活
gen.close()#传入GeneratorExit异常

gen.throw(KeyboardInterrupt)#throw可以传递任意异常，协程内捕获
```

## yield与yield from

### 生成器的3种模式

1.pull 不断往外产出数据，迭代器

```python
def pull_style():
    while still_have_data:
        yield data
```

2.push 不断向内发送数据

```python
def pull_style():
    while still_have_data:
        input = yield output
```

3.任务式（AsyncIO里的协程）,从前两个的数据驱动变为任务驱动，重点变为，什么时候yield:

1.遇到什么事需要yield

2.在出栈时如何设置回调函数，从而调用事件

3.谁完成了事件，使协程再次入栈满足条件

4.如何感知到协程入栈满足条件，调用send再次入栈

### yield from的具体实现

```python
result = yield from expr#跟表达式，表达式返回可迭代对象，即yield from后接可迭代对象
```

yield from方法实现

```python
_i = iter(expr)

while 1:
    try:
        _y=_i.send(None)
    except StopIteration as e:
        _r = _e.value
        break
    else:
        yield _y
result=_r
#可见yield from实现透传的功能，连接其上下级，将下级的return加入异常返回给上级
```

## 普通函数到协程

### 普通函数遇到阻塞

1.普通函数按顺序执行，遭遇阻塞会等待，使用sleep模拟

```python

def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result = big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
def big_step():
    ...
    print(f'    begin small_step')
    small_coro = small_step()#因为加入yield ，结果从函数调用结果修改为生成器
    while True:
        try:
            x = small_coro.send(None)
        except StopIteration as e:
            small_result = e.value
            break
        else:
            pass
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    yield sleep,2
    print('      努力工作中')
    return 123

one_task()
```

2.最底层函数无法消除阻塞，则向上抛出

```python
def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result = big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
def big_step():
    ...
    print(f'    begin small_step')
    small_coro = small_step()#因为加入yield ，结果从函数调用结果修改为生成器
    while True:
        try:
            x = small_coro.send(None)
        except StopIteration as e:
            small_result = e.value
            break
        else:
            pass
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000
#引入阻塞后，返回阻塞到上级
def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    yield sleep,2
    assert time.time() - t1 > 2,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return 123

one_task()

```

3.但是阻塞抛出未处理，通过断言判断条件未满足，则再次向上抛出处理

```python
def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_coro = big_step()
    while True:
        try:
            x = big_coro.send(None)
        except StopIteration as e:
            big_result = e.value
            break
        else:
            func,arg=x
            func(arg)
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
def big_step():
    ...
    print(f'    begin small_step')
    small_coro = small_step()#因为加入yield ，结果从函数调用结果修改为生成器
    while True:#while true这段类似于yield from的实现方法
        try:
            x = small_coro.send(None)
        except StopIteration as e:
            small_result = e.value
            break
        else:
            yield x
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000
#引入阻塞后，返回阻塞到上级
def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    yield sleep,2
    assert time.time() - t1 > 2,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return 123

one_task()
```

4.向上抛出的过程类似于yield from的透传功能，即yield from可以替换

```python

def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_coro = big_step()
    while True:
        try:
            x = big_coro.send(None)
        except StopIteration as e:
            big_result = e.value
            break
        else:
            func,arg=x
            func(arg)
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
def big_step():
    ...
    print(f'    begin small_step')
#     small_coro = small_step()#因为加入yield ，结果从函数调用结果修改为生成器
#     while True:
#         try:
#             x = small_coro.send(None)
#         except StopIteration as e:
#             small_result = e.value
#             break
#         else:
#             yield x
    small_result = yield from small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

#引入阻塞后，返回阻塞到上级
def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    yield sleep,2
    assert time.time() - t1 > 2,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return 123

one_task()
```

5.one_task处理阻塞依然会阻塞整体，则

- 为了消除阻塞再次向上抛出 
- 为了统一更换为yield from

```python

def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result=yield from big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
def big_step():
    ...
    print(f'    begin small_step')
    small_result = yield from small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

#引入阻塞后，返回阻塞到上级
def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    yield from YieldFromAble((sleep,2))
    assert time.time() - t1 > 2,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return 123

one_task()
```

因为最底层只使用yield为了统一格式，定义类

```python
class YieldFromAble():
    def __init__(self,obj):
        self.value=obj
    def __iter__(self):
        yield self#因为实现了yield的都是生成器函数，则，添加yield后，调用iter得到一个生成器

```

```python
a=YieldFromAble((sleep,2))
b=iter(a)
b
out:<generator object YieldFromAble.__iter__ at 0x000002361F03E510>
```

### 阶段总结

1.协程并不能自己消除阻塞
2.协程具有传染性
3.协程通过yield把阻塞换个方式传递给了上游
4.阻塞依然需要被解决

### 基于生成器的协程的升级

1..最高层也抛出阻塞后，也成为了生成器，直接调用会返回对象，则定义类

```python
class Task:
    def __init__(self,coro):
        self.coro=coro
    
    def run(self):
        print('------------')
        while True:
            try:
                x = self.coro.send(None)
            except StopIteration as e:
                result = e.value
                break
            else:
                func,arg=x.value
                func(arg)
        print('-----------')
t=Task(one_task())
t.run()
```

2.现在task运行时会运行阻塞，而不是交出执行权，因为使用的while true，则重构

```python
from time import sleep

class Task:
    def __init__(self,coro):
        self.coro=coro
        self._done=False
        self._result=None
    
    def run(self):
        print('------------')
        if not self._done:#此时run一次只会运行一次而后出栈，再次run入栈
            try:
                x = self.coro.send(None)
            except StopIteration as e:
                result = e.value
                self._done=True
            else:
                func,arg=x.value
                func(arg)
        else:
            print('task is done')
            
        print('-----------')

        
class YieldFromAble():
    def __init__(self,obj):
        self.value=obj
    def __iter__(self):
        yield self#因为实现了yield的都是生成器函数，则，添加yield后，调用iter得到一个生成器



        
def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result=yield from big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
def big_step():
    ...
    print(f'    begin small_step')
    small_result = yield from small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

#引入阻塞后，返回阻塞到上级
def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    yield from YieldFromAble((sleep,2))
    assert time.time() - t1 > 2,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return 123

t=Task(one_task())
t.run()
```

3.之后python引入自己的写成对象asyncio，升级只需要替换关键字,官方相关源码:

```python
def __await__(self):
    if not self.done():
        self._asycio_future_blocking=True
        yield self
    if not selg.done():
        raise RuntimeError()
    return self.result()
__iter__=__await__
```

```python
from time import sleep
class Task:
    def __init__(self,coro):
        self.coro=coro
        self._done=False
        self._result=None
    
    def run(self):
        print('------------')
        if not self._done:
            try:
                x = self.coro.send(None)
            except StopIteration as e:
                result = e.value
                self._done=True
        else:
            print('task is done')
            
        print('-----------')

        
class Awaitable():
    def __init__(self,obj):
        self.value=obj
    def __await__(self):
        yield self#因为实现了yield的都是生成器函数，则，添加yield后，调用iter得到一个生成器



        
async def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result=await big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
async def big_step():
    ...
    print(f'    begin small_step')
    small_result = await small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

#引入阻塞后，返回阻塞到上级
async def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    await Awaitable((sleep,2))
    assert time.time() - t1 > 2,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return 123

t=Task(one_task())
t.run()
for _ in range(10):#两次run之间的其他时间可以用以其他程序
    sleep(0.2)
    print('1')
t.run()
```

## 引入事件循环

```python
from time import sleep
import collections
import heapq
import random
import itertools

class EventLoop:
    def __init__(self):
        self._ready=collections.deque()  #定义任务队列
        self._scheduled=[]	#定义堆实现定时任务
        self._stopping=False	#标志位是否停止
        
    def call_soon(self,callback,*args):  #加入任务队列
        self._ready.append((callback,args))
    
    def call_later(self,delay,callback,*args):	#加入定时任务
        t=time.time()+delay
        heapq.heappush(self._scheduled,(t,callback,args))
    
    def stop(self):
        self._stopping=True	#停止
        
    def run_forever(self):	#启动事件循环
        while True:
            self.run_once()
            if self._stopping:
                break
    
    def run_once(self):	#运行单个事件
        now=time.time()
        if self._scheduled:
            if self._scheduled[0][0]<now:
                _,cb,args=heapq.heappop(self._scheduled)
                self._ready.append((cb,args))
        
        num=len(self._ready)
        
        for item in range(num):
            cb,args=self._ready.popleft()
            cb(*args)
            
task_id_counter = itertools.count(1)      

class Task:
    def __init__(self,coro):
        self.coro=coro
        self._done=False
        self._result=None
        self._id = f'task-{next(task_id_counter)}'
    
    def run(self):
        print(f'-----{self._id}-------') #对任务进行计数
        if not self._done:
            try:
                x = self.coro.send(None)
            except StopIteration as e:
                result = e.value
                self._done=True
            else:
                loop.call_later(x.value,self.run) #将阻塞任务加入任务队列
        else:
            print('task is done')
            
        print('-----------')

        
class Awaitable():
    def __init__(self,obj):
        self.value=obj
    def __await__(self):
        yield self#因为实现了yield的都是生成器函数，则，添加yield后，调用iter得到一个生成器



        
async def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result=await big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
async def big_step():
    ...
    print(f'    begin small_step')
    small_result = await small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

#引入阻塞后，返回阻塞到上级
async def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    sleep_time=random.random()
    await Awaitable(sleep_time)
    assert time.time() - t1 > sleep_time,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return sleep_time

loop=EventLoop()
t=Task(one_task())
loop.call_soon(t.run)
loop.call_later(2.1,loop.stop)
loop.run_forever()
```

2.启动多任务

```python
from time import sleep
import collections
import heapq
import random
import itertools

class EventLoop:
    def __init__(self):
        self._ready=collections.deque() 
        self._scheduled=[]
        self._stopping=False
        
    def call_soon(self,callback,*args):
        self._ready.append((callback,args))
    
    def call_later(self,delay,callback,*args):
        t=time.time()+delay
        heapq.heappush(self._scheduled,(t,callback,args))
    
    def stop(self):
        self._stopping=True
        
    def run_forever(self):
        while True:
            self.run_once()
            if self._stopping:
                break
    
    def run_once(self):
        now=time.time()
        if self._scheduled:
            if self._scheduled[0][0]<now:
                _,cb,args=heapq.heappop(self._scheduled)
                self._ready.append((cb,args))
        
        num=len(self._ready)
        
        for item in range(num):
            cb,args=self._ready.popleft()
            cb(*args)
            
task_id_counter = itertools.count(1)      

class Task:
    def __init__(self,coro):
        self.coro=coro
        self._done=False
        self._result=None
        self._id = f'task-{next(task_id_counter)}'
    
    def run(self):
        print(f'-----{self._id}-------')
        if not self._done:
            try:
                x = self.coro.send(None)
            except StopIteration as e:
                result = e.value
                self._done=True
            else:
                loop.call_later(x.value,self.run)
        else:
            print('task is done')
            
        print('-----------')

        
class Awaitable():
    def __init__(self,obj):
        self.value=obj
    def __await__(self):
        yield self#因为实现了yield的都是生成器函数，则，添加yield后，调用iter得到一个生成器



        
async def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result=await big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    
async def big_step():
    ...
    print(f'    begin small_step')
    small_result = await small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result * 1000

#引入阻塞后，返回阻塞到上级
async def small_step():
    print('      模拟阻塞')
    t1 = time.time()
    sleep_time=random.random()
    await Awaitable(sleep_time)
    assert time.time() - t1 > sleep_time,'睡眠时间不足' #类试判断时间必须完成
    print('      努力工作中')
    return sleep_time

loop=EventLoop()
for i in range(10):
    t=Task(one_task())
    loop.call_soon(t.run)
loop.call_later(1,loop.stop)
loop.run_forever()
```

## 引入Future对象，捕获事件

函数何时回调是由事件决定，即完成某个操作，获取某个值，而后回调，则定义Future，绑定事件

```python
from time import sleep
import collections
import heapq
import random
import itertools
import threading
import time


class EventLoop:
    def __init__(self):
        self._ready = collections.deque()
        self._scheduled = []
        self._stopping = False

    def call_soon(self, callback, *args):
        self._ready.append((callback, args))

    def call_later(self, delay, callback, *args):
        t = time.time() + delay
        heapq.heappush(self._scheduled, (t, callback, args))

    def stop(self):
        self._stopping = True

    def run_forever(self):
        while True:
            self.run_once()
            if self._stopping:
                break

    def run_once(self):
        now = time.time()
        if self._scheduled:
            if self._scheduled[0][0] < now:
                _, cb, args = heapq.heappop(self._scheduled)
                self._ready.append((cb, args))

        num = len(self._ready)

        for item in range(num):
            cb, args = self._ready.popleft()
            cb(*args)


class Future():
    def __init__(self):
        global loop
        self._loop = loop
        self._result = None
        self._done = False
        self._callbacks = []

    def result(self):
        if self._done:
            return self._result
        else:
            raise RuntimeError('is not done')

    def set_result(self, result):
        if self._done:
            raise RuntimeError('future')
        self._result = result
        self._done = True
        print(self._result)
        for cb in self._callbacks:
            self._loop.call_soon(cb)  # 进行阻塞后的函数回调

    def add_done_callback(self, callback):
        self._callbacks.append(callback)  # 加入等待队列

    def __await__(self):
        yield self  # 因为实现了yield的都是生成器函数，则，添加yield后，调用iter得到一个生成器
        return self.result()


task_id_counter = itertools.count(1)


class Task(Future):
    def __init__(self, coro):
        super().__init__()
        self.coro = coro
        self._loop.call_soon(self.run)
        self._id = f'task-{next(task_id_counter)}'

    def run(self):
        print(f'-----{self._id}-------')
        if not self._done:
            try:
                x = self.coro.send(None)
            except StopIteration as e:
                self.set_result(e.value)  # 获取值，表示阻塞完成，可以运行继续
            else:
                x.add_done_callback(self.run)  # 对于task而言，获得底层的yield值，用于回调判断何时回复运行,即加入future的运行队列
        else:
            print('task is done')

        print('-----------')


async def one_task():
    print(f'begin task')
    ...
    print(f'  begin big_step')
    big_result = await big_step()
    print(f'  end big_step witn {big_result}')
    ...
    print(f'end task')
    return big_result


async def big_step():
    ...
    print(f'    begin small_step')
    small_result = await small_step()
    print(f'    end small_step witn {small_result}')
    ...
    return small_result


# 引入阻塞后，返回阻塞到上级
async def small_step():
    #
    fut = Future()
    fake_io_read(fut)  # 绑定阻塞与future对象
    result = await fut  # 返回给task
    return result


def fake_io_read(future):
    # 模拟IO阻塞，用其他线程运行，对本线程只在乎结果
    def read():
        x = random.random()
        sleep(x)
        future.set_result(x)

    threading.Thread(target=read).start()


def until_all_done(tasks):
    # 用于判断任务是否全部完成
    tasks = [t for t in tasks if not t._done]
    if tasks:
        loop.call_soon(until_all_done, tasks)
    else:
        loop.stop()


loop = EventLoop()
all_tasks = [Task(one_task()) for i in range(1)]
loop.call_later(0.9, until_all_done, all_tasks)
loop.run_forever()

```



