> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.xn--xsqp6uwvq.com](https://blog.xn--xsqp6uwvq.com/610.html)

> Ubuntu 不能用，不知道为什么，iproute2 配置改了不生效。

Ubuntu 不能用，不知道为什么，iproute2 配置改了不生效。

添加路由表

```
	

vi /etc/iproute2/rt_table


	

252 cucc


	

251 cmcc


	

250 tel

```

```
	

#清空表


	

ip route flush table tel


	

#添加路由表tel中的缺省


	

ip route add default via 58.218.33.1 dev em2 src 58.218.33.177 table tel


	

#设置源进源出


	

ip rule add from 58.218.33.177 table tel


	
	

ip route flush table cmcc


	

ip route add default via 36.149.88.1 dev em3 src 36.149.88.37 table cmcc


	

ip rule add from 36.149.88.37 table cmcc


	
	

ip route flush table cuc


	

ip route add default via 157.0.217.1 dev em4 src 157.0.217.180 table cucc


	

ip rule add from 157.0.217.180 table cucc

```

原理:  
目的 IP 是 58.218.33.177 的传入连接被打标然后走 tel 这个路由表的缺省路由（58.218.33.1）出去。  
传出连接走缺省路由表。