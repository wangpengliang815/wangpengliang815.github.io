# 安装 jdk/git

```bash
yum -y install java-1.8.0-openjdk* #安装jdk
yum install git #安装git
```

# 安装 ca-certificates

```bash
yum install epel-release
yum install -y ca-certificates
```

# 导入公钥

```bash
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

# 安装 Jenkins

```bash
wget https://mirrors.bfsu.edu.cn/jenkins/redhat-stable/jenkins-2.319.3-1.1.noarch.rpm
rpm -ivh jenkins-2.319.3-1.1.noarch.rpm
```

# 配置修改

主要修改两个地方：

**账户**：因为 `Jenkins` 默认账户是 `jenkins`，这个账户我们没有，而且为了不因为权限出现各种问题，这里直接使用 `root`，也可以创建一个名叫 `jenkins` 的账户

**端口**：Jenkins的默认端口是 `8080`，为了避免端口冲突，可以修改为其他端口

配置文件地址：`vi /etc/sysconfig/jenkins`

# 防火墙添加端口

查看开放端口：

```bash
firewall-cmd --zone=public --list-ports
```

添加 `Jenkins` 端口

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent 
```

防火墙配置立即生效

```bash
firewall-cmd --reload
```

# 目录权限修改

```bash
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

# 启动 Jenkins

```bash
systemctl start jenkins
```


如果报错：

```bash
Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.
```

解决方案：

1. 查看当前Java的环境变量  `echo $JAVA_HOME`
2. 复制Java的环境变量地址, 编辑 `/etc/init.d/jenkins` 文件, 指定位置添加该地址, 后缀附上 `/bin/java`

```bash
vim /etc/init.d/jenkins

candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
/usr/jdk/jdk-18.0.1.1/bin/java # 新增jdk地址
"
```

修改完重新加载一下配置文件，使其生效：

```bash
systemctl daemon-reload
```

然后启动，将刚才修改的端口放开或者直接关闭防火墙，这里直接关闭防火墙：

```bash
systemctl stop firewalld
```

重启 Jenkins

```bash
systemctl restart jenkins
```

# Jenkins 插件安装

**步骤**：添加插件 => 系统管理 => 插件管理 

需要添加的插件：

- `Gitlab Hook`
- `Build Authorization Token Root`
- `Publish Over SSH`
- `Gitlab Authentication`
- `Gitlab`
- `Git Parameter`

在centos7中 `ssh` 服务默认是已经被安装了的。通过命令 `rpm -qa | grep openssh` 查看是否安装了ssh服务

~~~~bash
rpm -qa | grep openssh
~~~~

