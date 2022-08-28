`docker images` 可以查看已经下载到本地的image镜像

`docker ps` 或者 `docker container ls` 显示正在运行的容器。

如果你是一个运维人员，想研究负载均衡软件 HAProxy，只需要执行`docker run haproxy`，无需繁琐的手工安装和配置既可以直接进入实战。

如果你是一个开放人员，想学习怎么用 django 开发 Python Web 应用，执行 `docker run django`，在容器里随便折腾吧，不用担心会搞乱 Host 的环境。

即环境和配置全部由container解决,一次配置即可多次多地使用



### Dockerfile

**Dockerfile** 是镜像的**描述文件(readme)**，定义了如何构建 Docker 镜像。

hello-world 的 Dockerfile:![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFSIBib6RbMHHFjGNfMy30LWBibZSNKzJtasySoVIAzzDbFHicWFicqGyicZOlR0wiaHl7GicKufCCfAWM5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

只有短短三条指令。

1. FROM scratch
   从 0 开始构建。
2. COPY hello /
   将文件“hello”复制到镜像的根目录。
3. CMD ["/hello"]
   容器启动时，执行 /hello

可执行文件hello就是文件系统的全部内容，连最基本的 /bin，/usr, /lib, /dev 都没有。

# base 镜像

base 镜像有两层含义：

1. 不依赖其他镜像，**从 scratch 构建**。
2. 其他镜像可以之为**基础**进行扩展。

所以，能称作 base 镜像的通常都是各种 Linux 发行版的 Docker 镜像，比如 Ubuntu, Debian, CentOS 等。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqHgLTBaYYiaZT02d8FqmkE11j5wPPEk028ere1yavXzrr1l8eXqHEn98vclEIzHGU5W1rWhIFIoNSA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Linux 操作系统由**内核空间**和**用户空间**组成,**用户空间基于内核空间**

**rootfs**

内核空间是 kernel，Linux 刚启动时会加载 bootfs 文件系统，之后 bootfs 会被卸载掉。

用户空间的文件系统是 rootfs，包含我们熟悉的 /dev, /proc, /bin 等目录。

对于 base 镜像来说，底层直接用 Host 的 kernel，自己只需要提供 rootfs 就行了。

而对于一个精简的 OS，rootfs 可以很小，只需要包括最基本的命令、工具和程序库就可以了。

**base 镜像提供的是最小安装的 Linux 发行版**。

不同 Linux 发行版的区别主要就是 rootfs。

所以 Docker 可以同时支持多种 Linux 镜像，模拟出多种操作系统环境。![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqHgLTBaYYiaZT02d8FqmkE11VmKI6eD1LyFrfejFpBwCYDdmREGJdPvNRLKFAB6mWxffrYANm1NL0Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图 Debian 和 BusyBox（一种嵌入式 Linux）上层提供各自的 rootfs，底层共用 Docker Host 的 kernel。(一个host同时安装两个linux系统)



base 镜像只是在用户空间与发行版一致，kernel 版本与发型版是不同的。

1. 例如 CentOS 7 使用 3.x.x 的 kernel，如果 Docker Host 是 Ubuntu 16.04（比如我们的实验环境），那么在 CentOS 容器中使用的实际是是 Host 4.x.x 的 kernel

2. 容器只能使用 Host 的 kernel，并且不能修改。
   所有容器都共用 host 的 kernel，在容器中没办法对 kernel 升级。如果容器对 kernel 版本有要求（比如应用只能在某个 kernel 版本下运行），则不建议用容器，这种场景虚拟机可能更合适。