# socket

## 1 定义

socket翻译为套接字，socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用来实现进程在网络中通信

## 2 分类

### 2.1 基于文件的套接字

AF_UNIX，在 unix中一切皆文件，基于文件的套接字调用的就是底层的文件系统来取数据，两个套接字进程运行在同一机器，可以通过访问同一个文件系统间接完成通信。

### 2.2 基于网络类型的套接字家族

python支持很多种地址家族，由于我们只关心网络编程，所以大部分时候我们只使用AF_INET

## 3 socket

### 3.1 主要流程

```
服务端：
​	1 初始化socket对象
​	2 socket对象与服务端的IP和端口绑定（bind）
​	3 对端口进行监听(listen)
客户端：
​	1 初始化socket对象
​	2 连接服务器(connect)
​	3 如果连接成功，客户端与服务端的连接就建立了
连接成功后：
​	1 客户端发送数据请求
​	2 服务端接收请求并处理请求
​	3 讲回应数据发送给客户端，客户端读取数据
​	4 关闭连接，一次交互结束
```

### 3.2 创建对象

```python
# socket_family 可以是 AF_UNIX 或 AF_INET。socket_type 可以是 SOCK_STREAM 或 SOCK_DGRAM。socket_family默认是-1，表示AF_INET，socket_type默认也是-1，表示SOCK_STREAM。可以通过源码查看
socket.socket(socket_family, socket_type, protocal=0)

# 获取tcp/ip套接字
tcp_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_sock = socket.socket()  # 默认就是-1，可以不写

# 获取udp/ip套接字
udpSock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

### 3.3 服务端提供方法

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 获取socket对象
s.bind()   # 绑定主机、端口号到套接字
s.listen()  # 开始TCP监听
s.accept()  # 被动接受TCP客户的连接,(阻塞式)等待连接的到来
```

### 3.4 客户端提供方法

```python
import socket

s = socket.socket()  # socket对象
s.connect()  # 主动初始化连接TCP服务器
s.connect_ex()  # connect()函数的扩展版本,出错时返回出错码,而不是抛出异常
```

###  3.5 公共方法

```python
s.recv()   # 接收TCP数据
s.send()   # 发送TCP数据(send在待发送数据量大于己端缓存区剩余空间时,数据丢失,不会发完)
s.sendall()   # 发送完整的TCP数据(本质就是循环调用send,sendall在待发送数据量大于己端缓存区剩余空间时,数据不丢失,循环调用send直到发完)
s.recvfrom()   # 接收UDP数据
s.sendto()   # 发送UDP数据
s.getpeername()   # 连接到当前套接字的远端的地址
s.getsockname()   # 当前套接字的地址
s.getsockopt()   # 返回指定套接字的参数
s.setsockopt()   # 设置指定套接字的参数
s.close()   # 关闭套接字连接
```

## 4 基于TCP的套接字

### 4.1 服务端文件

```python
# sever.py  服务端文件
# 以手机接打电话为例
import socket

# 1 买手机---获取服务端收发数据的对象
phone = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

# 2 手机插卡---给服务端绑定IP+端口
phone.bind(('127.0.0.1',6666))

# 3 开机---服务端处于监听状态
phone.listen(3)  # 半连接池只能存放三个待确认的连接请求

# 4 等待来电---服务端与客户端建立连接，建立连接后可以拿到TCP连接的通道信息，和客户端的IP和接口
conn,client_addr = phone.accept()
print(conn)
print('客户端IP和接口',client_addr)

# 5 电话接通，两人愉快的聊天---收发数据
data = conn.recv(1024)  # 最大接收数据量为1024bytes
print('客户端消息', data.decode('utf-8'))
conn.send(data.upper())

# 6 挂断电话---断开与客户端的连接
conn.close()

# 7 手机关机---服务端关机
phone.close()
```

### 4.2 客户端文件

```python
# client.py
import socket

phone = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

phone.connect(('127.0.0.1',6666))

phone.send('show 信息'.encode('utf-8'))

data = phone.recv(1024)

print('服务端',data.decode('utf-8'))

phone.close()
```

### 4.3 循环通信时服务端

```python
# sever.py
import socket

# 1 买手机---获得服务端的对象
phone = socket.socket()

# 2 手机插卡---确定服务端的IP和端口
phone.bind(('127.0.0.1',7890))

# 3 手机开机---服务端进入监听状态
phone.listen(3)

# 4 等待接听电话，获取TCP通道和客户端的IP和端口

conn,client_addr = phone.accept()

# 5 通电话---收发数据
while True:
    # 异常处理：当防止客户端突然关闭服务端崩溃
    try:
        data = conn.recv(1024)
        if not data:break
        print('客户端',data.decode('utf-8'))

        conn.send(data.upper())
    except Exception:
        break

# 6 挂断电话---服务端与客户端通道断开
conn.close()

# 7 关机---服务端关机
phone.close()#去掉这段代码则服务端保持在线
```

### 4.4 循环通信客户端

```python
# client.py
import socket

phone = socket.socket()
phone.connect(('127.0.0.1', 7890))
while True:
    info = input('>>').strip()
    # 当发送的数据长度为0时，服务端和客户端都会进入等待收数据的阻塞阶段，所以进行判断，判断用户输入的信息被strip处理后，长度是否为0
    if len(info) == 0:continue
    if info == 'q': break
    phone.send(info.encode('utf-8'))
    data = phone.recv(1024)
    print('服务端', data.decode('utf-8'))

phone.close()
```

## 5 粘包问题

### 5.1 问题原因

接收端不知道发送端将要传送的字节流的长度，所以解决粘包的方法就是围绕，如何让发送端在发送数据前，把自己将要发送的字节流总大小让接收端知晓，然后接收端来一个循环接收完所有数据

###  5.2 解决方法

发送端发数据前，先将待发的数据长度告知接收端。将数据长度放在一个固定长度的字节中发给接收端；接收端先接收这个固定长度的数据头，从这个数据头中获悉待接收数据的长度，做好循环接收的准备

### 5.3 使用工具

struct模块，该模块可以将任意数据类型转换成固定长度的`bytes`，因此借助该模块就可以将发送方发送的数据总大小通过struct模块打包成固定长度的`bytes`，接收端接收后获取发送方发送的数据总大小之后，使用循环接收即可

### 5.4  具体步骤

1.把报头做成字典，字典里包含将要发送的真实数据的详细信息，然后json序列化，然后用struck将序列化后的数据长度打包成4个字节

2.发送时： 先发报头长度，再编码报头内容然后发送，最后发真实内容

3.接收时： 先接收报头长度，用struct取出来，根据取出的长度收取报头内容，然后解码反序列化，从反序列化的结果中取出待取数据的详细信息，然后去取真实的数据内容

### 5.5 具体实现

服务端：

```python
import subprocess
import struct
import json
import socket

server=socket(AF_INET,SOCK_STREAM)
server.bind(('127.0.0.1',8083))
server.listen(5)

#  服务端应该做两件事
# 第一件事：循环地从板连接池中取出链接请求与其建立双向链接，拿到链接对象
while True:
    conn,client_addr=server.accept()

    # 第二件事：拿到链接对象，与其进行通信循环
    while True:
        try:
            cmd=conn.recv(1024)
            if len(cmd) == 0:break
            obj=subprocess.Popen(cmd.decode('utf-8'),
                             shell=True,
                             stdout=subprocess.PIPE,
                             stderr=subprocess.PIPE
                             )

            stdout_res=obj.stdout.read()
            stderr_res=obj.stderr.read()
            total_size=len(stdout_res)+len(stderr_res)

            # 1、制作头
            header_dic={
                "filename":"a.txt",
                "total_size":total_size,
                "md5":"1111111111"
            }

            json_str = json.dumps(header_dic)
            json_str_bytes = json_str.encode('utf-8')


            # 2、先把头的长度发过去
            x=struct.pack('i',len(json_str_bytes))
            conn.send(x)

            # 3、发头信息
            conn.send(json_str_bytes)
            # 4、再发真实的数据
            conn.send(stdout_res)
            conn.send(stderr_res)

        except Exception:
            break
    conn.close()
```

客户端

```python
import struct
import json
from socket import *

client=socket(AF_INET,SOCK_STREAM)
client.connect(('127.0.0.1',8083))

while True:
    cmd=input('cmd>>：').strip()
    if len(cmd) == 0: continue
    client.send(cmd.encode('utf-8'))

    # 接收端
    # 1、先收4个字节，从中提取接下来要收的头的长度
    x = client.recv(4)
    header_len=struct.unpack('i',x)[0]

    # 2、接收头，并解析
    json_str_bytes=client.recv(header_len)
    json_str=json_str_bytes.decode('utf-8')
    header_dic=json.loads(json_str)
    print(header_dic)
    total_size=header_dic["total_size"]

    # 3、接收真实的数据
    recv_size = 0
    while recv_size < total_size:
        recv_data=client.recv(1024)
        recv_size+=len(recv_data)
        print(recv_data.decode('utf-8'),end='')
    else:
        print()
```

