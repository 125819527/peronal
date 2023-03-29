1.使用 apt-get update 令来更新软件源信息

```
 apt-get update 
```

2.默认的官方源速度慢的话，也可以替换为国内 163 sohu 等镜像的源 163 源为 例，在容器内创建 etc/apνsources.list.d/163.list 文件：

```
 vi /etc/apt/sources.list.d/163.list 
```

3.添加如下内容到文件中

```
deb http: //mirrors.163.com/ubuntu / bionic main restricted universe multiverse 
http: //mirrors 163 cαn/ubuntu/ bionic-security main restricted universe multiverse 
deb http://mi ors.163.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb http://mirrors.163.cαn/ubu tu/ bionic proposed main restricted universe multiverse 
deb http: //mi ors.163.com/ubuntu/ bionic -ba orts main restricted universe mul tiverse 
deb-src http://mirrors 163 cαn/ubuntu/ bionic main restricted verse mult verse
deb-src http //mirrors 63 com/ubuntu/ bionic-security mai restricted universe mul tiverse 
deb-src http://mi ors.163.com/ubuntu/ bionic updates main restricted universe multiverse 
deb-src http://mirrors.163.com/ub tu/ bionic-proposed main restricted universe multiverse 
deb-src http //mi ors.163.cαn/ubuntu/ bionic-badφorts main restricted universe multiverse 
```

