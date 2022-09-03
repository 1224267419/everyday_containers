容器之间可通过 IP，Docker DNS Server 或 joined 容器三种方式通信。

#### **IP 通信**

从上一节的例子可以得出这样一个结论：两个容器要能通信，必须要有属于同一个网络的网卡。

满足这个条件后，容器就可以通过 IP 交互了。具体做法是在**容器创建时通过** `--network` **指定相应的网络**，或者通过 `docker network connect` **将现有容器加入到指定网络**。可参考[上一节 httpd 和 busybox 的例子](http://mp.weixin.qq.com/s?__biz=MzIwMTM5MjUwMg==&mid=2653587690&idx=1&sn=770e7bbd74377f84bd5e8ee0f42ca194&chksm=8d3080f3ba4709e5f3a8fe9b666a5db5b246746271952828eed2d6d34cb19076d5cde2aeb6a7&scene=21#wechat_redirect)，这里不再赘述。



#### **Docker DNS Server**

通过 IP 访问容器虽然满足了通信的需求，但还是不够灵活。因为我们在部署应用之前可能无法确定 IP，部署之后再指定要访问的 IP 会比较麻烦。对于这个问题，可以通过 docker 自带的 DNS 服务解决。
docker daemon 实现了一个内嵌的 DNS server，使容器可以直接通过“容器名”通信。方法很简单，只要在启动时用 `--name` 为容器命名就可以了。

下面启动两个容器 bbox1 和 bbox2：

`docker run -it --network=my_net2 --name=bbox1 busybox`

`docker run -it --network=my_net2 --name=bbox2 busybox`
<img src="http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGz1VIvtVQibic3vAV2QYnhetYZ9HXZxjrByWyBGaUgX6p2W8CaK0OeeZr6Tia9lENDrexWojibWIyKag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:67%;" />

然后就能在bbox2里面直接ping到bbox1,且无需指定ip

使用 docker DNS 有个限制：**只能在 user-defined 网络中使用**。也就是说，**默认的 bridge 网络是无法使用 DNS 的**。



#### **joined 容器**

joined 容器是另一种实现容器间通信的方式。

joined 容器非常特别，它可以使两个或多个容器共享一个网络栈，共享网卡和配置信息，joined 容器之间可以通过 127.0.0.1 直接通信。请看下面的例子：



先创建一个 httpd 容器，名字为 web1。

`docker run -d -it --name=web1 httpd`

然后创建 busybox 容器并通过 `--network=container:web1` 指定 jointed 容器为 web1：

`docker run -it --network=container:web1 busybox`



![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGz1VIvtVQibic3vAV2QYnhetwwOvecBsFhsMnRCictbicicqXLqPrykpib8z7dpGAIP6ZUSjTAfpzanlqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以下是web1 容器中的网络配置信息

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGz1VIvtVQibic3vAV2QYnhetIiaKql85HEIpVaiaJFJn2ElyxSMyaac90HjfIPfzCalqaS1I0XeEpC5A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对比两个容器的网络, 他们的的网卡 mac 地址与 IP 完全一样，它们共享了相同的网络栈。busybox 可以直接用 127.0.0.1 访问 web1 的 http 服务。



##### joined网络总结:

joined 容器非常适合以下场景：

1. 不同容器中的程序希望通过 loopback 高效快速地通信，比如 web server 与 app server。
2. 希望监控其他容器的网络流量，比如运行在独立容器中的**网络监控程序**。