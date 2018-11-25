# 阿里云服务器基础操作

### 更新系统
```shell
yum upgrade -y
```

### 修改计算机名
```shell
hostnamectl set-hostname aleserver

//使用下面的命令，重启使之生效

systemctl restart systemd-hostnamed
```
### 修改ssh默认端口

* 修改ssh端口号为想要的
```shell
vi /etc/ssh/sshd_config

// 搜索 Port，去掉#号，把22改成喜欢的端口，如22022
:wq
```
* 设置semanage
```shell
yum -y install policycoreutils-python
semanage port -a -t ssh_port_t -p tcp 22022

//检测是否成功
semanage port -l | grep ssh
```

* 设置防火墙
```shell
systemctl start  firewalld
```
* 添加端口（这里同时添加了80和443，不需要的同学后面两句不用）
```shell
firewall-cmd  --permanent --zone=public --add-port=22022/tcp
firewall-cmd  --permanent --zone=public --add-port=80/tcp
firewall-cmd  --permanent --zone=public --add-port=443/tcp
```
* 修改防火墙端口
```shell
cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/
vi /etc/firewalld/services/ssh.xml

// 修改端口为22022
:wq
```
* 重新载入，使之生效
```shell
systemctl restart sshd.service
firewall-cmd --reload  
systemctl restart  firewalld
```
此时 22端口已经不能登录


### 使用秘钥登录
* 添加用户
```shell
useradd Jason
```
* 设置密码
```shell
passwd Jason
```
* 加入wheel组
```shell
usermod -G wheel Jason  
```
* 修改配置文件
```shell
vi /etc/pam.d/su

//找到

#auth required pam_wheel.so use_uid  // 找到此行，去掉行首的“#”
:wq
```

* 修改 /etc/ssh/sshd_config 

```shell
vim /etc/ssh/sshd_config

// 找到 #StrictModes yes 改成 StrictModes no （去掉注释后改成 no） 
// 找到 #PubkeyAuthentication yes 去掉注释 
// 找到 #AuthorizedKeysFile .ssh/authorized_keys 去掉注释
:wq
``` 
* 使用Jason登录
```shell
ssh-keygen -t RSA -C "yuomail@gmail.com"
// 第一次提示存储位置，直接回车
// 第二次提示是否需要密码，空密码直接回车
// 第三次重复密码

cd /home/Jason/.ssh
cp id_rsa.pub authorized_keys
chmod 600 authorized_keys 
```
* 下载私钥文件保存到本地

* 配置xshell或SecureCRT等

* 使用秘钥登录
```shell
su //登录成功后，换root权限
```
* 修改ssh登录方式
```shell
vim /etc/ssh/sshd_config

#PermitRootLogin yes 　← 找到这一行，将行首的“#”去掉，并将yes改为no
　↓
PermitRootLogin no 　← 修改后变为此状态，不允许用root进行登录

#PasswordAuthentication yes　← 找到这一行，将yes改为no
　↓
PasswordAuthentication no　← 修改后变为此状态，不允许密码方式的登录

#PermitEmptyPasswords no　 ← 找到此行将行头的“#”删除，不允许空密码登录
　↓
PermitEmptyPasswords no　 ← 修改后变为此状态，禁止空密码进行登录
wq:
```
* 重新载入，使之生效
```shell
systemctl reload sshd 
systemctl restart sshd.service
```
此时只能用Jason账号登录。