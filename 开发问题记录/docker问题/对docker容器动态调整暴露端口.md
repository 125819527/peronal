# 对docker容器动态调整暴露端口

1. docker的端口映射并不是在docker技术中实现的，而是通过宿主机的iptables来实现。通过控制网桥来做端口映射，类似路由器中设置路由端口映射

2. 可以通过iptables查看当前端口映射

   ```
   sudo iptables -t nat -vnL
   ```

   ```
   Chain DOCKER
   target     prot opt source               destination
   RETURN     all  --  0.0.0.0/0            0.0.0.0/0
   DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.17.0.3:80
   ```

   我们可以看到docker创建了一个名为DOKCER的自定义的链条Chain。而我开放80端口的容器的ip是172.17.0.3

3. 也可以通过inspect命令查看容器ip:

   ```
   docker inspect containerId |grep IPAddress
   ```

4. 我们想再增加一个端口映射，比如`8081->81`，就在这个链条是再加一条规则

   ```
   sudo iptables -t nat -A  DOCKER -p tcp --dport 8081 -j DNAT --to-destination 172.17.0.3:81
   ```

5. 如果加错了或者想修改：

   先显示行号查看

   ```
   sudo iptables -t nat -vnL DOCKER --line-number
   ```

   删除规则3

   ```text
   sudo iptables -t nat -D DOCKER 3
   ```