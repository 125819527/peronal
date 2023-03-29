# python Web框架

软件开发的架构有两种，分别是C/S架构与B/S架构，本质上都是借助socket实现网络通信，因此Django作为一个web框架本质上也是一个socket服务端，浏览器则是客户端

## 1.使用socket创建最基础web服务端

```python
import socket

server = socket.socket()
server.bind(('127.0.0.1',8080))
server.listen(5)

while True:
    conn, addr = server.accept()
    data = conn.recv(1024)
    print(data)
    conn.send(b'web frame')
    conn.close()

```

## 2.使数据满足HTTP协议

此时访问提示发送无效数据，socket服务端向客户端发送的数据格式不符合HTTP协议的要求，如果想让客户端浏览器收到服务端发送的数据，就要遵循HTTP协议，因此服务端在向客户端发送数据的时候必须按照HTTP协议的规则加上相应状态行

```python
import socket

server = socket.socket()
server.bind(('127.0.0.1', 8080))
server.listen(5)

while True:
    conn, addr = server.accept()
    data = conn.recv(1024)
    print(data)
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')
    conn.send(b'web frame')
    conn.close()

```

## 3.增加模拟路由功能

在使用浏览器上网的时候，访问不同的网址就会返回不同的页面或者是内容，根据已有的代码就可以通过if判断实现客户端访问不同的url返回不同的数据或者页面，在请求头增加请求路径

```python
'''
b'GET /index HTTP/1.1\r\n   # \r\n表示换行
Host: 127.0.0.1:8080\r\n
Connection: keep-alive\r\n
Cache-Control: max-age=0\r\n
Upgrade-Insecure-Requests: 1\r\n
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36\r\n
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n
Sec-Fetch-Site: cross-site\r\n
Sec-Fetch-Mode: navigate\r\n
Sec-Fetch-User: ?1\r\n
Sec-Fetch-Dest: document\r\n
Accept-Encoding: gzip, deflate, br\r\n
Accept-Language: zh-CN,zh;q=0.9\r\n\r\n'
'''
# if判断实现根据不同的url返回不同的数据或页面
import socket


server = socket.socket()
server.bind(('127.0.0.1',8080))
server.listen(5)

while True:
    conn,addr = server.accept()
    data = conn.recv(1024)
    print(data)
    data_list = data.decode('utf-8').split(' ')  # 将数据以空格切分
    print(data_list)
    current_path = data_list[1]  # 列表中的第二个元素就是路由
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')
    if current_path == '/index':
        conn.send(b'index')
    elif current_path == '/login':
        conn.send(b'login')
    else:
        conn.send(b'web frame')
    conn.close()
```

## 4.采用函数与列表优化路由

```python
import socket


def index(url):
    s = 'this is {}'.format(url)
    return bytes(s, encoding='utf-8')


def login(url):
    s = 'this is {}'.format(url)
    return bytes(s, encoding='utf-8')

# 定义一个url与函数功能的对应关系的列表
url_list = [
    ('/index', index),
    ('/login', login)
]

server = socket.socket()
server.bind(('127.0.0.1', 8080))
server.listen(5)

while True:
    conn, addr = server.accept()
    data = conn.recv(1024)
    print(data)
    data_list = data.decode('utf-8').split(' ')
    print(data_list)
    current_path = data_list[1]
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')

    # 定义一个变量名用以保存函数的内存地址
    func = None
    for i in url_list:
        if i[0] == current_path:
            func = i[1]
            break
    if func:
        res = func(current_path)
    else:
        res = b'404 not found'
    conn.send(res)
    conn.close()
```

## 5.加入html模板

```python
import socket
import datetime

server = socket.socket()
server.bind(('127.0.0.1',8080))
server.listen(5)

def index(url):
    current_time = datetime.datetime.now().strftime('%Y-%m-%d %X')
    with open('time.time.html','r',encoding='utf-8') as f:

        data = f.read()
        
    # 在网页上定义好特殊符号，用字符串方法替换,类比渲染功能
    s = data.replace('sfsd',current_time)
    return bytes(s, encoding='utf-8')

def login(url):
    with open('myhtml.html','r',encoding='utf-8') as f:
        res = f.read()
    return bytes(res,encoding='utf-8')

# 定义一个url与函数功能的对应关系的列表
url_list = [
    # (路由，视图函数)
    ('/index',index),
    ('/login',login)
]
while True:
    conn,addr = server.accept()
    data = conn.recv(1024)
    print(data)
    data_list = data.decode('utf-8').split(' ')
    print(data_list)
    current_path = data_list[1]
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')

    # 定义一个变量名用以保存函数的内存地址
    func = None
    for i in url_list:
        if i[0] == current_path:
            func = i[1]
            break
    if func:
        res = func(current_path)
    else:
        res = b'404 not found'
    conn.send(res)
    conn.close()
```

## 6.wsgiref模块

可以借助wsgiref模块对套接字原始框架进行处理

```python
from wsgiref.simple_server import make_server


def run(env,response):
    '''

    :param env: 请求相关的所有数据
    :param response: 响应相关的所有数据
    :return: 返回给浏览器的数据,return [b'']
    '''
    # 响应首行 响应头
    response('200 OK', [])

    # env就是字典格式的HTTP数据
    print(env)
    current_path = env.get('PATH_INFO')  # 获取客户端浏览器请求的路由
    # wsgiref模块帮你处理号HTTP格式数据，封装成成字典
    if current_path == '/index':
        return [b'index']
    elif current_path == '/login':
        return [b'welcome']
    else:
        return [b'404 error']


if __name__ == '__main__':
    server = make_server('127.0.0.1',8080,run)
    # 实时监听127.0.0.1：8080地址，只要有客户端来了都会交给run函数处理，即触发run函数的运行
    server.serve_forever()  # 启动服务端
```

## 7.将不同的功能放在不同的py文件

结构目录

urls.py				路由与视图函数对应关系

```python
# urls.py
import views

urls = [
    ('/index', views.index),
    ('/loging', views.longin)
]


```

views.py			视图函数的逻辑（后端业务逻辑）

```python
# views.py
import datetime


def index(env):
    with open(r'template/myhtml.html', 'r', encoding='utf-8') as f:
        res = f.read()
        return res


def longin(env):
    current_time = datetime.datetime.now().strftime('%Y-%m-%d %X')
    with open(r'template/time.time.html', 'r', encoding='utf-8') as f:
        res = f.read()
    res = res.replace('sfsd', current_time)
    return res
```



templates文件夹	  专门存储html文件
sever.py			服务端文件

```python
from wsgiref.simple_server import make_server
from urls import  urls
from views import *



def run(env,response):
    '''

    :param env: 请求相关的所有数据
    :param response: 响应相关的所有数据
    :return: 返回给浏览器的数据,return [b'']
    '''
    # 响应首行 响应头
    response('200 OK', [])

    # env就是字典格式的HTTP数据
    print(env)
    current_path = env.get('PATH_INFO')
    # wsgiref模块帮你处理号HTTP格式数据，封装成成字典
    func = None
    for i in urls:
        if current_path == i[0]:
            func = i[1]
            break
    if func:
        res = func(env)
    else:
        res = '404 not found'
    return [res.encode()]

if __name__ == '__main__':
    server = make_server('127.0.0.1',8080,run)
    # 实时监听127.0.0.1：8080地址，只要有客户端来了都会交给run函数处理，即触发run函数的运行
    server.serve_forever()  # 启动服务端


```

