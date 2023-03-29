# PYTHON魔法函数

## 1.魔法函数定义

魔法方法是Python的内置函数，一般以双下划线开头，每个魔法方法对应的一个内置函数或者运算符，例如对自定义类进行大小比较

```python
class People(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
        return

    def __str__(self):
        return self.name + ":" + str(self.age)

    def __lt__(self, other):
        return self.name < other.name if self.name != other.name else self.age < other.age


if __name__=="__main__":

    print("\t".join([str(item) for item in sorted([People("abc", 18),
        People("abe", 19), People("abe", 12), People("abc", 17)])]))

```

上个例子中的`__lt__`函数即`less than`函数，即当比较两个People实例时自动调用

## 2.非数学运算

### 2.1 字符串表示

```python
class Cat:
    """定义一个猫类"""
 
    def __init__(self, new_name= "汤姆", new_age= 20):
        """在创建完对象之后 会自动调用, 它完成对象的初始化的功能"""
        self.name = new_name
        self.age = new_age  # 它是一个对象中的属性,在对象中存储,即只要这个对象还存在,那么这个变量就可以使用
        # num = 100  # 它是一个局部变量,当这个函数执行完之后,这个变量的空间就没有了,因此其他方法不能使用这个变量
 
    def __str__(self):
        """返回一个对象的描述信息"""
        # print(num)
        return "st名字是:%s , st年龄是:%d" % (self.name, self.age)
    
    def __repr__(self):
        return "re名字是:%s , re年龄是:%d" % (self.name, self.age)
    

# 创建了一个对象
tom = Cat("汤姆", 30)
#包含__repr__,与__str__，使用print(tom)调用__str__,使用tom时使用__repr__，在不存在str,但是具有repr时，print也会调用repr
```

### 2.2 集合序列相关

```python
#在对对象使用len，会调用__len__方法
class Students():
    def __init__(self, *args):
        self.names = args
    def __len__(self):
        return len(self.names)

ss = Students('Bob', 'Alice', 'Tim')
print(len(ss))


```

```python
#__getitem_() 主要作用是可以让对象实现迭代功能,返回与指定键相关联的值,也使对象可已被以a['']的形式使用,列表、元组等序列之所以可以索引取值、分片取值，是因为它们实现了__getitem__方法
class Tag:
    def __init__(self):
        self.change={'python':'This is python'}
 
    def __getitem__(self, item):
        print('这个方法被调用')
        return self.change[item]
 
a=Tag()
print(a['python'])

#实现迭代，寻找iter与next都不存在寻找__getitem__
class Animal:
    def __init__(self, animal_list):
        self.animals_name = animal_list

    def __getitem__(self, index):
        return self.animals_name[index]

animals = Animal(["dog","cat","fish"])
for animal in animals:
    print(animal)
```

```python
#__setitem__,在设置类实例属性时自动调用的
# -*- coding:utf-8 -*-
 
class A:
    def __init__(self):
        self['B']='BB'
        self['D']='DD'
        
    def __setitem__(self,name,value):
 
        print "__setitem__:Set %s Value %s" %(name,value)
        
        
if __name__=='__main__':
    X=A()

__setitem__:Set B Value BB
__setitem__:Set D Value DD

```

```python
#__delitem__(),在对对象的组成部分使用__del__语句的时候被调用，应删除与key相关联的值
class Tag:
    def __init__(self):
        self.change={'python':'This is python',
                     'php':'PHP is a good language'}
 
    def __getitem__(self, item):
        print('调用getitem')
        return self.change[item]
 
    def __setitem__(self, key, value):
        print('调用setitem')
        self.change[key]=value
 
    def __delitem__(self, key):
        print('调用delitem')
        del self.change[key]
 
a=Tag()
print(a['php'])
del a['php']
print(a.change)

#输出：
调用getitem
PHP is a good language
调用delitem
{'python': 'This is python'}

```

```python
#__contains__,使对象实现in的方法
class Graph():
    def __init__(self):
        self.items = {'a':1,'b':2,'c':3}
    def __contains__(self,x): # 判断一个定点是否包含在里面
        return x in self.items

a = Graph()
print('a' in a) # 返回True
print('d' in a) # 返回False

>> True
>> False

```

### 2.3 迭代相关

只有同时满足iter与next才满足可迭代协议，才算迭代器，iter需要返回迭代器，如果本身实现next，也可返回自身

```python
#对象具有__iter__实现可迭代，__next__实现迭代器，在不存在iter时，不会重置
class test():
    def __init__(self,data=1):
        self.data = data

    def __next__(self):
        if self.data > 5:
            raise StopIteration
        else:
            self.data+=1
            return self.data

t = test(3)   
for i in range(1):
    print(t.__next__())
#结果为3
for i in range(1):
    print(t.__next__())   
#结果为4
    
```

```python
#iter返回对象本身，表示为可迭代，next获取下一个遍历数据，此时使用for会从头开始遍历
class test():
    def __init__(self,data=1):
        self.data = data

    def __iter__(self):
        return self
    def __next__(self):
        if self.data > 5:
            raise StopIteration
        else:
            self.data+=1
            return self.data

for item in test(3):
    print(item)
```



### 2.4 可调用

该方法的功能类似于在类中重载 () 运算符，使得类实例对象可以像调用普通函数那样，以“`对象名()`”的形式使用。作用：为了将类的实例对象变为可调用对象

```python
class CLanguage:
    # 定义__call__方法
    def __call__(self,name,add):
        print("调用__call__()方法",name,add)

clangs = CLanguage()
clangs("")

```

或者使用函数本身进行调用

```python
class Fun:
    def __init__(self):
        pass

print(callable(Fun))

```

通过增加__call__()函数，将类实例化对象变为可调用

```pyth
class Fun:
    def __init__(self):
        pass
    
    def __call__(self, *args, **kwargs):
        pass

a = Fun()
print(callable(a))

```



