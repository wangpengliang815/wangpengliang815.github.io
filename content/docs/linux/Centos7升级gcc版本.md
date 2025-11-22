Centos 7 默认 `gcc` 版本为4.8，有时需要更高版本的，这里以升级至8.3.1版本为例，分别执行下面三条命令即可，无需手动[下载源码](http://ftp.gnu.org/gnu/gcc/)编译。


1、安装centos-release-scl
```bash
sudo yum install centos-release-scl
```


2、安装devtoolset，注意：如果想安装7.* 版本的，就改成devtoolset-7-gcc，以此类推
```bash
sudo yum install devtoolset-8-gcc*
```


3、激活对应的devtoolset，可以一次安装多个版本的devtoolset，需要的时候用下面这条命令切换到对应的版本
```bash
scl enable devtoolset-8 bash
```


搞定，查看一下gcc版本
```bash
gcc -v
```
```bash
[root@wangpengliang ~]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/opt/rh/devtoolset-8/root/usr/libexec/gcc/x86_64-redhat-linux/8/lto-wrapp
erTarget: x86_64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefi
x=/opt/rh/devtoolset-8/root/usr --mandir=/opt/rh/devtoolset-8/root/usr/share/man --infodir=/opt/rh/devtoolset-8/root/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --with-default-libstdcxx-abi=gcc4-compatible --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-8.3.1-20190311/obj-x86_64-redhat-linux/isl-install --disable-libmpx --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linuxThread model: posix
gcc version 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC) 
[root@wangpengliang ~]#
```


显示为 gcc version 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC)


补充：这条激活命令只对本次会话有效，重启会话后还是会变回原来的4.8.5版本，要想随意切换可按如下操作。


首先，安装的 `devtoolset` 是在 `/opt/rh`  目录下的
```
[root@wangpengliang rh]# ls
devtoolset-8  devtoolset-9
[root@wangpengliang rh]#
```


每个版本的目录下面都有个 `enable` 文件，如果需要启用某个版本，只需要执行
```bash
source ./enable
```


所以要想切换到某个版本，只需要执行
```bash
source /opt/rh/devtoolset-8/enable
```


可以将对应版本的切换命令写个shell文件放在配了环境变量的目录下，需要时随时切换，或者开机自启


4、直接替换旧的gcc
旧的gcc是运行的 `/usr/bin/gcc` ，所以将该目录下的gcc/g++替换为刚安装的新版本gcc软连接，免得每次 `enable` 


```bash
mv /usr/bin/gcc /usr/bin/gcc-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc

mv /usr/bin/g++ /usr/bin/g++-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/g++ /usr/bin/g++

gcc --version
g++ --version
```
