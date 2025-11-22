# ✨ Hello

使用Hugo构建的Blog，旨在将学习过程中的知识点记录下来以便查阅.  [☘️ Github](https://github.com/wangpengliang815)

# 🏷️ Technology Stack

* `.NETCore/Python`
* `Docker/Harbor`
* `RabbitMq/Kafaka`
* `Jenkins/GitLab`
* `Redis`
* `InfluxDB`
* `Microservice`
* `ELK/Prometheus`


# ⌨️ Daily Command

## git 代理

```bash
git config --global http.proxy 127.0.0.1:21882 #设置代理
git config --global --get http.proxy #查看代理
git config --global --unset http.proxy #取消代理
```

## 设置主机名称

```bash
hostnamectl set-hostname centos-master
```

## 自动补全

```bash
yum install bash-completion -y
```

## vim 安装

```bash
yum install vim -y
```

使用 `vi` or `vim` 进入文本后，按`i`编辑文本

`ESC` 键：

- `:q!`  不保存文件，强制退出
- `:w`   保存文件，不退出
- `:wq`  保存文件，退出

## ssh-key 生成

检查 `ssh-key` 是否已存在

```bash
ls -al ~/.ssh
```

如不存在按照如下命令生成 `sshkey`

```bash
ssh-keygen -t rsa -C "15101587969@163.com"
```

按照提示完成三次回车即可生成。通过查看 `~/.ssh/id_rsa.pub` 文件内容，获取到 `public key`

```bash
cat ~/.ssh/id_rsa.pub
```

## netstat

centos7 默认没有 netstat 命令，需要安装 net-tools 工具

```
yum install -y net-tools
```



```bash
查看监听端口
netstat -lnpt

检查端口被哪个进程占用
netstat -lnpt |grep 5672

查看进程的详细信息
ps 6832

中止进程
kill -9 6832
```

## 软件安装

Centos 版本 Linux 提供了多种软件安装方式，包括：

- 使用源码编译安装
- rpm 包安装
- yum命令安装

### rpm 查看软件信息

查询已安装的包版本

```bash
[root@localhost /]# rpm -q nginx
nginx-1.12.2-3.el7.x86_64
```

查询已安装的所有软件包

```bash
[root@localhost /]rpm -qa
[root@localhost /]rpm -qa|less # 分页列出安装包信息
```

查询某个软件的安装集合

```bash
[root@localhost /]# rpm -qa|grep nginx
```

查询某个应用程序的安装位置与安装文件

```bash
[root@localhost /]# rpm -qla|grep nginx
```

### yum 查看软件信息

列出所有已安装的软件包

```bash
yum list installed 
```

查找软件包 

```bash
yum search 
```

列出所有可安装的软件包 

```bash
yum list 
```

列出所有可更新的软件包 

```bash
yum list updates 
```


列出所有已安装但不在 Yum Repository 内的软件包 

```bash
yum list extras 
```

获取软件包信息 

```bash
yum info bash
```


列出所有可更新的软件包信息 

````bash
yum info updates 
````


列出所有已安装的软件包信息 

```bash
yum info installed 
```

列出所有已安装但不在 Yum Repository 内的软件包信息 

```bash
yum info extras 
```

列出软件包提供哪些文件 

```bash
yum provides
```



