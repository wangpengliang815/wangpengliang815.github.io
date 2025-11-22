# 系统级别

| 描述                              | 示例                   | 备注 |
| --------------------------------- | ---------------------- | ---- |
| 显示机器的处理器架构              | `arch`                 |      |
| 显示机器的处理器架构              | `uname -m`             |      |
| 显示正在使用的内核版本            | `uname -r`             |      |
| 显示硬件系统部件 - (SMBIOS / DMI) | `dmidecode -q`         |      |
| 罗列一个磁盘的架构特性            | `hdparm -i /dev/hda`   |      |
| 在磁盘上执行测试性读取操作        | `hdparm -tT /dev/sda`  |      |
| 显示CPU info的信息                | `cat /proc/cpuinfo`    |      |
| 显示中断                          | `cat /proc/interrupts` |      |
| 校验内存使用                      | `cat /proc/meminfo`    |      |
| 显示哪些swap被使用                | `cat /proc/swaps`      |      |
| 显示内核的版本                    | `cat /proc/version`    |      |
| 显示网络适配器及统计              | `cat /proc/net/dev`    |      |
| 显示已加载的文件系统              | `cat /proc/mounts`     |      |
| 罗列 PCI 设备                     | `lspci -tv`            |      |
| 显示 USB 设备                     | `lsusb -tv`            |      |
| 显示系统日期                      | `date`                 |      |
| 显示2007年的日历表                | `cal 2007`             |      |
| 设置日期和时间 - 月日时分年.秒    | `date 041217002007.00` |      |
| 将时间修改保存到 BIOS             | `clock -w`             |      |
| 关闭系统                          | `shutdown -h now`      |      |
| 关闭系统                          | `telinit 0`            |      |
| 取消按预定时间关闭系统            | `shutdown -c`          |      |
| 重启                              | `shutdown -r now`      |      |
| 重启                              | `reboot`               |      |
| 注销                              | `logout`               |      |

# 目录相关

| 描述                                      | 示例                        | 备注                  |
| ----------------------------------------- | --------------------------- | --------------------- |
| 进入指定目录                              | `cd /home`                  |                       |
| 返回上一级目录                            | `cd ..`                     |                       |
| 返回上两级目录                            | `cd ../..`                  |                       |
| 进入个人主目录                            | `cd`                        |                       |
| 进入上次操作所在目录                      | `cd -`                      |                       |
| 显示当前工作路径                          | `pwd`                       |                       |
| 查看目录中的文件                          | `ls`                        |                       |
| 查看目录中的文件                          | `ls -F`                     |                       |
| 显示文件和目录的详细信息                  | `ls -l`                     |                       |
| 显示隐藏文件                              | `ls -a`                     |                       |
| 显示包含数字的文件名和目录名              | `ls *[0-9]*`                |                       |
| 显示文件和目录由根目录开始的树形结构      | `tree`                      | `yum install tree -y` |
| 创建目录                                  | `mkdir dir`                 |                       |
| 创建多个目录                              | `mkdir dir1 dir2`           |                       |
| 创建目录树                                | `mkdir -p /tmp/dir1/dir2`   |                       |
| 删除文件                                  | `rm -f file1`               |                       |
| 删除目录                                  | `rmdir dir1`                |                       |
| 删除目录并同时删除内容                    | `rm -rf dir1`               |                       |
| 同时删除多个目录及内容                    | `rm -rf dir1 dir2`          |                       |
| 重命名/移动 目录                          | `mv dir1 new_dir`           |                       |
| 复制文件                                  | `cp file1 file2`            |                       |
| 复制目录下的所有文件到当前工作目录        | `cp dir/* .`                |                       |
| 复制目录到当前工作目录                    | `cp -a /tmp/dir1.`          |                       |
| 复制目录                                  | `cp -a dir1 dir2`           |                       |
| 创建一个指向文件或目录的软链接            | `ln -s file1 lnk1`          |                       |
| 创建一个指向文件或目录的物理链接          | `ln file1 lnk1`             |                       |
| 修改一个文件或目录的时间戳 - (YYMMDDhhmm) | `touch -t 0712250000 file1` |                       |

# 网络相关

| 描述                                               | 示例                                           | 备注 |
| -------------------------------------------------- | ---------------------------------------------- | ---- |
| 查看本机IP及网卡信息                               | `ip addr`                                      |      |
| 重启网络                                           | `service network restart`                      |      |
| 防火墙添加端口例外                                 | `firewall-cmd --add-port=8080/tcp --permanent` |      |
| 重新加载防火墙                                     | `firewall-cmd --reload`                        |      |
| 启动防火墙，也可以使用 `service firewalld start`   | `systemctl start firewalld.service`            |      |
| 停止防火墙，也可以使用 `service firewalld stop`    | `systemctl stop firewalld.service`             |      |
| 启用防火墙                                         | `systemctl enable firewalld.service`           |      |
| 重启防火墙                                         | `service firewalld restart`                    |      |
| 查看端口列表，也可以使用 `firewall-cmd --list-all` | `firewall-cmd --permanent --list-port`         |      |
| 查看防火墙状态                                     | `firewall-cmd --state`                         |      |

