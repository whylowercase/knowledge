# 什么是Nginx

Nginx 事更加轻量级的服务器，内存占用少，启动快，高并发能力强，在互联网项目中广泛应用。可以作为反向代理服务器使用

# Nginx的特点

1. 内存占用少
2. 高并发：单机支持10万以上的并发连接
3. 跨平台：可以在Linux,windows,macos上
4. 扩展性好：支持第三方插件
5. 安装使用简单
6. 稳定性
7. 免费

# Nginx可以用来干什么

## 静态资源服务器

可以作为http服务器，将静态文件通过http协议展现给客户端

## 反向代理

客户端将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器，获取数据后再返回给客户端。对外暴露的是反向代理服务器地址，隐藏了真实服务器IP地址

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fgitee.com%2FSnailClimb%2Fblog-images%2Fraw%2Fmaster%2Fnginx%2F%2Freverse-proxy.png&sign=286269bbff8cc1fe87969db28b7a004509c1396407e36ac5150c22f9879d4f74)

客户端请求直接经过代理服务器，由代理服务器将请求转发到内网服务器并最终决定哪一台服务器处理客户端的请求。

## 正向代理

客户端通过正向代理服务器访问目标服务器，目标服务器不知道客户端是谁，就是说客户端对目标服务器的访问是透明的。

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fgitee.com%2FSnailClimb%2Fblog-images%2Fraw%2Fmaster%2Fnginx%2F%2Fforward-proxy.png&sign=9969dc2bbd628eb70a5b027d4a12a209044785532ca6d6e63b7b0035931aec28)

我们可以借助VPN来访问无法直接访问的网站，VPN会把访问外网服务器的客户端请求代理到一个可以直接访问外网的代理服务器上去。代理服务器会把外网服务器返回的内容在转发给客户端。

## 负载均衡

如果一台服务器处理用户请求处理不过来的话，简单的办法就是增加多台服务器（服务器集群）部署相同的服务来处理用户请求。Nginx可以将接收到的客户端请求以一定的规则（负载均衡策略）均匀地分配到集群中的所有服务器上，这个就是`负载均衡`。nginx还带有健康检查（服务器心跳检查）的功能，会定期轮询向集群中的所有服务器发送健康检查请求，来检查集群中是否有服务器处于异常状态。

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fgitee.com%2FSnailClimb%2Fblog-images%2Fraw%2Fmaster%2Fnginx%2F%2Fnginx-load-balance.png&sign=bfe3a3f2c0613966e67fb3f8be2b919214b128567a875bfe4361113f7c6c44d5)



## 动静分离

将动态请求和静态请求分开，可以理解为Nginx处理静态页面，Tomcat处理动态页面。减轻服务器的压力，提高网站响应速度。



# 负载均衡策略

1. 轮询

每个请求按时间顺序逐一分配到不同的后端服务器

```nginx
upstream backserver {
  server 172.27.26.174:8099;
  server 172.27.26.175:8099;
  server 172.27.26.176:8099;
}
```

2. 加权随机

通过加入weight参数，调整服务器被访问的概率

```nginx
upstream backserver {
  server 172.27.26.174:8099 weight=6;
  server 172.27.26.175:8099 weight=2;
  server 172.27.26.176:8099 weight=3;
}
```

3. IP哈希

根据发出请求的和护短ip的hash值来分配服务器，可以保证同一IP发出的请求映射到同一服务器，或者具有相同hash值的不同IP映射到同一服务器。

```nginx
upstream backserver {
  ip_hash;
  server 172.27.26.174:8099;
  server 172.27.26.175:8099;
  server 172.27.26.176:8099;
}
```

4. 最小连接数

当有新的请求出现时，遍历服务器节点列表并选择其中连接数最小的一台服务器来响应当前的请求。

```nginx
upstream backserver {
  least_conn;
  server 172.27.26.174:8099;
  server 172.27.26.175:8099;
  server 172.27.26.176:8099;
}
```



# Nginx性能优化

* 设置Nginx运行工作进程的个数：一般为CPU核心数或者核心数X2
* 开启gzip压缩：
* 设置单个worker进程允许客户端最大连接数：一般设置为65535
* 连接超时时间设置：避免在建立无用连接上消耗太多资源
* 设置缓存

