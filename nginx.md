# nginx

### 1.基本概念

![image-20191225160017906](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225160017906.png)

### 2.反向代理

![image-20191225160418054](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225160418054.png)

### 3.负载均衡

![image-20191225160656543](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225160656543.png)

### 4.动静分离

![image-20191225160752940](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225160752940.png)

### 5.nginx配置文件

![image-20191225163851949](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225163851949.png)

![image-20191225164123157](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225164123157.png)

### 配置实例

##### 反向代理配置：

![image-20191225222637976](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225222637976.png)

![image-20191225170820671](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225170820671.png)



![image-20191225170900410](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225170900410.png)



![image-20191225223023121](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225223023121.png)

##### 负载均衡配置：

![image-20191225223206752](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225223206752.png)



![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225171624623.png)

分配策略：

**轮询（默认）**

**权重策略**

![image-20191225172023823](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225172023823.png)

**ip_hash**

![image-20191225172121317](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225172121317.png)

**fail:**

![image-20191225172211187](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225172211187.png)

##### 动静分离：

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225173316935.png)

##### 高可用集群

https://www.jianshu.com/p/5403818b1b34

[补充] 

```
vi /etc/hosts
```

添加主机名LVS_DEVEL

### 6.nginx原理

![image-20191225182754811](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225182754811.png)

worker工作方式

![image-20191225182929221](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225182929221.png)

好处：

![image-20191225183933941](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225183933941.png)

![image-20191225183334041](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225183334041.png)

![image-20191225183501109](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225183501109.png)

![image-20191225183602380](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191225183602380.png)