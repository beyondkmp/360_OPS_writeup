# 360_OPS_writeup

360公司OPS实习生试题\_Writeup

## 基础题

### 1.password_cache

1. ssh-keygen生成相应的公私钥对，默认会在~/.ssh下面：`id_rsa` 和`id_rsa.pub`, 将id\_rsa.pub拷到相应的服务器上就不要再服务密码了。
2. 考虑id\_rsa私钥的安全性，先修改权限chmod 600 ~/.ssh/id_rsa,再将id_rsa拷贝到自己知道的目录下。下次连接的时候用ssh -i 来连接，但是要保证sudo不能查看目前应该是没有办法(个人观点)。**个人建议就是将私钥保存在u盘上，用完就拨掉u盘**

### 2. ifconfig_reg

```python
#!/usr/bin/env python
#coding: utf-8
import os

def ifconfig_hash():
        os.system('ifconfig > /tmp/ifconfig_cache')
        ifconfig_cache=open("/tmp/ifconfig_cache")
        save_hash={}
        for l in ifconfig_cache:
                # 判断第一个不为空且不为空行
                if l[0]!=' ' and l[0]!='\n':
                        # 取网卡
                        key=l.split(' ')[0]
                        l=ifconfig_cache.next()
                        #在取网卡的下一行可以直接读取ip
                        value=l.strip().split(' ')[1].split(':')[1]
                        save_hash[key]=value;
        ifconfig_cache.close();
        os.system('rm /tmp/ifconfig_cache')
        return save_hash

if __name__=='__main__':
        print ifconfig_hash()

```

运行结果如下：

![ifconfig_result](image/ifconfig.png)


### 4. log_cutting

```bash
# access.log日志切割
#!/bin/bash
mv access.log access_$(date +"%Y%m%d").log
```

保存上面的脚本程序为access_log.sh,并保存到/usr/bin/access_log.sh,接下来设置crontab作业：

```
0 0 * * * bash /usr/bin/access_log.sh
```

每天的0点0分，那个时候应该都睡觉了，会自动保存access日志，并将其日期作为名字的一部分保存。

**缺陷**

考虑的过于简单，有可能会出现日志丢失的情况。


## 应用题

### 1.socks_proxy:

```bash
#!/bin/bash

while true
do
    if ! nc -z 0.0.0.0 8081 > /dev/null
    then
        ssh -p 22 -N -D 0.0.0.0:8081  beyond@172.16.199.150 &
    fi
    sleep 30
done
```

将上面的程序保存为socks_proxy.sh,每次要运行就直接sh socks_porxy.sh &,也可以写到服务里面。

### 7.agent

### control.py

```python
#!/usr/bin/env python
#coding: utf-8
from socket import *
from time import ctime
import select
import sys
HOST='172.16.199.150'
PORT=21569
BUFSIZ=1024
ADDR=(HOST,PORT)  #服务器的地址与端口

tcpCliSock=socket(AF_INET,SOCK_STREAM) #生成客户端的套接字，并连上服务器
tcpCliSock.connect(ADDR)
input1=[tcpCliSock,sys.stdin]

while True:
    readyInput,readyOutput,readyException=select.select(input1,[],[])
    for indata in readyInput:
        if indata==tcpCliSock:
            data=tcpCliSock.recv(BUFSIZ)
            if not data:
                break
            print data
        else:
            data=raw_input()
            if not data:
                break
            tcpCliSock.send(data) #发送命令

tcpCliSock.close()
```

### server.py

```python
#!/usr/bin/env python
#coding: utf-8

from socket import *
from time import ctime
import select
import sys
import os

HOST=''
PORT=21569
BUFSIZ=1024
ADDR=(HOST,PORT)

tcpSerSock=socket(AF_INET,SOCK_STREAM)
tcpSerSock.bind(ADDR)
tcpSerSock.listen(5)
input=[tcpSerSock,sys.stdin]

while True:
    print 'waiting for connection...'
    tcpCliSock,addr=tcpSerSock.accept()
    print '...connected from:',addr
    input.append(tcpCliSock) #可以有多个控制端同时连接

    while True:
        readyInput,readyOutput,readyException=select.select(input,[],[]) #每次循环都会阻塞在这里，只有当有数据输入时才会执行下面的操作
        for indata in readyInput:
            if indata==tcpCliSock:
                data=tcpCliSock.recv(BUFSIZ)
                if not data:
                    break
                tcpCliSock.send(os.popen(data).read())

tcpCliSock.close()

```
运行结果如下：

![agent](image/agent_result.png)






