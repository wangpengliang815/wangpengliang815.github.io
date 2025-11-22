
# 网络模式

- `VMnet0`（桥接模式）
- `VMnet1`（仅主机模式）
- `VMnet8`（NAT模式）


## NAT-VMnet8
NAT(地址转换)-VMnet8：虚拟机要联网得先通过宿主机才能和外面进行通信。

`NAT` 模式下的虚拟系统的 `TCP/IP` 配置信息是由 `VMnet8(NAT)` 虚拟网络的 `DHCP` 服务器提供的，无法进行手工修改，因此虚拟系统也就无法和本局域网中的其他真实主机进行通讯。使得虚拟局域网内的虚拟机在对外访问时，使用的则是宿主机的IP地址，这样从外部网络来看只能看到宿主机，完全看不到新建的虚拟局域网。就是虚拟系统会通过宿主机的网络来访问外网，而这里的宿主机相当于有两个网卡，一个是真实网卡，一个是虚拟网卡，真实网卡相当于链接了现实世界的真实路由器，而宿主机的虚拟网卡，相当于连接了一个可以认为是虚拟交换机。

![](/images/2021-09-03-15-22-27.png)

> 虚拟机可以上网可以ping通主机，但是主机ping不通虚拟机

## Bridged-VMnet0
Bridged(桥接)-VMnet0：虚拟机和宿主机在网络上就是平级的关系，相当于连接在同一交换机上。


需要手工为虚拟系统配置`IP地址`、`子网掩码`，而且还要和宿主机器处于同一网段，这样虚拟机才能和宿主机器进行通信，实现通过局域网的网关或路由器访问互联网。使用 Bridged 模式的虚拟系统和宿主机器的关系，就像连接在同一个Hub上的两台电脑。相当于在一个局域网内创立了一个单独的主机，他可以访问这个局域网内的所有的主机，但是需要手动配置IP地址，子网掩码，并且需要和真实主机在同一网段（nat是两个网段）。

![](/images/2021-09-03-15-23-07.png)

> 这个模式里，虚拟机和宿主机可以互相ping通

# 查看IP及网卡信息
```shell
ip addr
```

![](/images/2021-09-03-15-23-43.png)

# 编辑网卡信息
```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
```bash
TYPE=Ethernet
BOOTPROTO=static #修改成static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777736
UUID=bf5337ab-c044-4af7-9143-12da0d493b89
DEVICE=eno16777736
ONBOOT=yes #修改成yes
PEERDNS=yes
PEERROUTES=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPADDR=192.168.31.32  # 自定义虚拟机的ip地址（主机是192.168.31.31），必须与主机在同一网段
NETMASK=255.255.255.0 # 设置子网掩码，跟宿主一样
GATEWAY=192.168.31.1  # 默认网关，跟宿主一样
DNS1=192.168.31.1 # DNS，跟宿主一样
```


# 重启 network
```shell
service network restart
```
