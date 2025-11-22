# Nodejs 安装

## 下载

```bash
 wget https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.gz
 
 tar -xvf node-v16.14.0-linux-x64.tar.gz # 将文件解压到当前目录
```

## 配置环境变量

```bash
ln -s /hexo/node-v16.14.0-linux-x64/bin/node /usr/local/bin/node

ln -s /hexo/node-v16.14.0-linux-x64/bin/npm /usr/local/bin/npm

ln -s /hexo/node-v16.14.0-linux-x64/lib/node_modules/hexo-cli/bin/hexo /usr/local/bin/hexo

ln -s /projects/hexo/node-v16.14.0-linux-x64/bin/cnpm  /usr/local/bin/cnpm # cnpm需要单独安装
```

# Hexo 安装

## 创建工作目录

```bash
mkdir hexo
cd hexo
```
## 初始化 Hexo

```bash
# 安装Git(已安装可跳过)
yum install git-core
# 安装 Hexo
npm install -g hexo-cli
# 初始化 Hexo
hexo init
```

# Pm2

参考：https://www.jianshu.com/p/a256ca175c64

## 环境变量配置

```bash
ln -s /hexo/node-v16.14.0-linux-x64/bin/pm2 /usr/local/bin/pm2
```

## hexo_run.js

```bash
const { exec } = require('child_process')
exec('hexo server',(error, stdout, stderr) => {
        if(error){
                console.log('exec error: ${error}')
                return
        }
        console.log('stdout: ${stdout}');
        console.log('stderr: ${stderr}');
})
```

## 常用命令

```bash
pm2 start hexo_run.js    #启动
pm2 list    #查看pm2管理的所有服务

pm2 stop all    #停止pm2列表的所有服务
pm2 stop 0 #停止进程为0的进程

pm2 reload all #重新载入列表所有进程
pm2 reload 0 #重载列表中进程为0的进程

pm2 restart all    #重启列表中所有的进程
pm2 restart 0    #重启列表中进程为0的进程

pm2 delete 0    #删除列表中进程为0的进程
pm2 delete all    #删除列表中所有的进程
```

