``` bash
/sbin/route del default
/sbin/ifconfig eth0 down
/sbin/ifconfig eth0 10.0.2.15 netmask 255.255.255.0 broadcast 10.0.2.255 up
/sbin/route add default gw 10.0.2.2 eth0
echo "nameserver 10.0.2.3" > /etc/resolv.conf
```
  如果 route del default 报错，可以忽略。
  然后确认：
``` bash
/sbin/ifconfig eth0
/sbin/route -n
```
  正确结果应该是：
``` bash
eth0 inet addr:10.0.2.15
```
  路由表应该有：
```
10.0.2.0    0.0.0.0     255.255.255.0   U    eth0
0.0.0.0     10.0.2.2    0.0.0.0         UG   eth0
```
  然后再测：
``` bash
ping -c 4 10.0.2.2
telnet 110.242.69.21 80
```

连上后输入：
``` bash
  GET / HTTP/1.0
  Host: www.baidu.com
  # 有空行
```
  注意最后要多按一次回车，形成空行。
  如果这个能返回 HTTP/HTML，说明外网 TCP 已经通了
  
  # ping
``` bash
/sbin/ifconfig eth0 10.0.2.15 netmask 255.255.255.0 broadcast 10.0.2.255 up
/sbin/route add default gw 10.0.2.2 eth0
echo "nameserver 10.0.2.3" > /etc/resolv.conf
```
  如果已有错误默认路由，先：
``` bash
/sbin/route del default
```
  然后验证：
``` bash
ping -c 4 10.0.2.2
ping -c 4 8.8.8.8
ping -c 4 110.242.69.21
ping -c 4 www.baidu.com
```
