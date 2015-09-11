# 360_OPS_writeup

360公司OPS实习生试题\_Writeup

## 基础题

### 1.password_cache

1. ssh-keygen生成相应的公私钥对，默认会在~/.ssh下面：`id_rsa` 和`id_rsa.pub`, 将id\_rsa.pub拷到相应的服务器上就不要再服务密码了。
2. 考虑id\_rsa私钥的安全性，先修改权限chmod 600 ~/.ssh/id_rsa,再将id_rsa拷贝到自己知道的目录下。下次连接的时候用ssh -i 来连接，但是要保证sudo不能查看目前应该是没有办法(个人观点)。**个人建议就是将私钥保存在u盘上，用完就拨掉u盘**

