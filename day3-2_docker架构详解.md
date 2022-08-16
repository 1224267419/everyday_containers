Docker 的核心组件包括：

1. Docker 客户端 - Client
2. Docker 服务器 - Docker daemon
3. Docker 镜像 - Image
4. Registry
5. Docker 容器 - Container

Docker 架构如下图所示：![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGoHdCKcU42XBesbicBfOav44jzReKyPCXA4zHPLGmZZicFicf8LPiaC1fl4vkKAzl9aicbI1wyIBibxnpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Client/Server 架构。客户端向服务器发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行在同一个 Host 上，客户端也可以通过 socket 或 REST API 与远程的服务器通信



1. 使用**docker命令**,构建运行容器

2. **Docker 服务器**

   **Docker daemo**n 是服务器组件，以 **Linux 后台服务**的方式运行。

   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGoHdCKcU42XBesbicBfOav4cUZ1YxDVtibuCEWJ7qqAeGOm8FoKyRJedh9jwh7YhgdTDfF0icfTia2JQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   Docker daemon **运行在 Docker host** 上，负责**创建、运行、监控容器，构建、存储**镜像。

   **默认**配置下，Docker daemon **只能响应来自本地 Host 的客户端请**求。如果要允许**远程客户端请求**，需要**在配置文件中打开 TCP 监听**，步骤如下：

   1. 编辑配置文件 /etc/systemd/system/multi-user.target.wants/docker.service，在环境变量 `ExecStart` 后面添加 `-H tcp://0.0.0.0`，允许来自任意 IP 的客户端连接。
      ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGoHdCKcU42XBesbicBfOav4V03f6JPyaoPLTWvXrGONyNnryxhGskVLmLXrGmVezQeVPdwQiaHEuEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)如果使用的是其他操作系统，配置文件的位置可能会不一样。
   2. 重启 Docker daemon。
      ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGoHdCKcU42XBesbicBfOav4aEzv6XPtNQjJBXOodftw0icSUpibnSRlb8GNiaSdTxvXicoCDu1S1GVAIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
   3. 服务器 IP 为 192.168.56.102，客户端在命令行里加上 -H 参数，即可与远程服务器通信。
      ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGoHdCKcU42XBesbicBfOav4FIxLwC3UwAHH6s4T0ibib5xedW4I2Ypq9o4PZIOccRFr47DrDvzdjTrw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)`info` 子命令用于查看 Docker 服务器的信息。

##### **Docker 镜像**

可将 Docker 镜像看作只读模板，通过它可以创建 Docker 容器。

我们可以将镜像的内容和创建步骤描述在一个文本文件中，这个文件被称作 Dockerfile，通过执行 `docker build <docker-file>` 命令可以构建出 Docker 镜像，后面我们会讨论。

##### **Docker 容器**

Docker **容器就是** Docker **镜像的运行实例**。

用户可以通过 CLI（docker）或是 API 启动、停止、移动或删除容器。可以这么认为，对于应用软件，镜像是软件生命周期的构建和打包阶段，而容器则是启动和运行阶段。



##### **Registry**

Registry 是存放 Docker 镜像的仓库，Registry 分私有和公有两种。

Docker Hub（https://hub.docker.com/） 是默认的 Registry，由 Docker 公司维护，上面有数以万计的镜像，用户可以自由下载和使用。
也可以创建自己的私有 Registry。后面我们会学习



以下这两比较**重要**

`docker pull` 命令可以从 Registry **下载镜像**。
`docker run` 命令则是**先下载镜像（如果本地没有）**，然后**再启动容器**。