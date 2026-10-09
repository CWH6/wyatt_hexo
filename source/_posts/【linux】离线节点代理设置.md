# 场景

将node1（不离线），node3（离线），内网互联，可以将node3的网络代理到node1，比如内网jenkins需要代理到node1



## node1

安装代理工具

```shell
# 下载 squid
sudo apt update && sudo apt install squid -y
```



修改配置

```shell
# 备份配置文件
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.bak

# 编辑配置文件
vim /etc/squid/squid.conf

http_port 172.16.0.136:3128     # 只监听内网网卡，不监听公网
acl node3 src 172.16.0.134/32
http_access allow node3
http_access deny all

```



调整配置顺序&验证

```shell
sudo sed -i '/^http_access deny all/i acl node3 src 172.16.0.134/32\nhttp_access allow node3' /etc/squid/squid.conf

# 确认插进去了
sudo grep -nE 'acl node3|http_access allow node3|http_access deny all' /etc/squid/squid.conf
```

输出如下正确

```shell
1628:acl node3 src 172.16.0.134/32
1629:http_access allow node3
1630:http_access deny all
```



重启服务

```shell
# 查看最近日志
sudo tail -20 /var/log/squid/access.log
 
# 启动  squid
sudo systemctl restart squid
```



## node3

测试代理

```shell
curl -x http://172.16.0.136:3128 -I https://www.baidu.com
```



响应结果如下：

```shell
HTTP/1.1 200 OK
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Content-Length: 0
Content-Type: text/html
Pragma: no-cache
Server: bfe
Date: Thu, 08 Oct 2026 06:06:05 GMT
```











