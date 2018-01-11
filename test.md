## 1 环境
* 阿里云美国ecs服务器
* centos 7
## 2 安装
参考[此文章](https://www.textarea.com/ExpectoPatronum/shiyong-shadowsocks-kexue-shangwang-265/)，选择对应的操作系统，按步骤来即可。注意使用root安装，否则会出现权限错误。

## 3 遇到的问题

前面的安装应该很顺利就可以完成。当使用ssserver启动时，报了这个错误
```
Traceback (most recent call last):
  File "/bin/ssserver", line 11, in <module>
    load_entry_point('shadowsocks==2.8.2', 'console_scripts', 'ssserver')()
  File "/usr/lib/python2.7/site-packages/shadowsocks/server.py", line 68, in main
    tcp_servers.append(tcprelay.TCPRelay(a_config, dns_resolver, False))
  File "/usr/lib/python2.7/site-packages/shadowsocks/tcprelay.py", line 582, in __init__
    server_socket.bind(sa)
  File "/usr/lib64/python2.7/socket.py", line 224, in meth
    return getattr(self._sock,name)(*args)
socket.error: [Errno 99] Cannot assign requested address
```
github上的解决方法是将/etc/shadowsocks.json配置中的server置为0.0.0.0即可。修改后，发现仍然telnet对应端口，仍然不通。

因为阿里云由防火墙规则，通过设置安全组白名单，方可访问ecs服务器的某些端口。而且规则会按协议进行区分。

在阿里云实例的安全组规则中配置“自定义TCP”规则。表单填写如下：
```
端口范围: 8080/8080
授权对象: 0.0.0.0/0
优先级：1
```
上面配置的含义是开放0.0.0.0的8080端口访问，优先级为1（最高）

完成上面修改后，问题解决

## 4 多端口和多密码配置

shadowsocks支持不通端口使用不同密码，这样就可以避免多人持有同一密码的情况。具体在/etc/shadowsocks.json配置添加如下内容：
```
   "port_password": {
      "8080": "pwFor8080",
      "8082": "pwFor8082",
      "8081": "pwFor8081"
   }
```

