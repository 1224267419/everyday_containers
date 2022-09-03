![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEwPSYbFVIEZopYzvV5rO5adZXdX3DJ9jAuMyX2KChaozsEVq886sQ6ELIs33sGG3CkvukKewCFCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如图,两个 busybox 容器都挂在 my_net2 上，应该能够互通(之前建立过)![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEwPSYbFVIEZopYzvV5rO5aDTrnP5eC34BqI9GbHGf6RBuQrRJjDrGrrwWjKP4za8eeZmfd9ibX7qg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如图,ping 172.22.16.8和172.22.16.1都能ping通,证明了同一网络中的容器、网关之间都是可以通信的



但`my_net2` 与默认 bridge 网络不能通信,由拓扑图可知,两个网络属于不同的网桥，不能通信



 让busybox 与 httpd 通信：为 httpd 容器添加一块 net_my2 的网卡。这个可以通过`docker network connect` 命令实现。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEwPSYbFVIEZopYzvV5rO5a3hUVDrT1QiapqEOVvZRcmNibU9rAs0ia3XqyDQdI8SJksfs1ibHg2qPHxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
我们在 httpd 容器中查看一下网络配置：AA
![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEwPSYbFVIEZopYzvV5rO5aQiaDuwibt4K8xiby1P5kuT1k8Aic9SxDnic1ic3o9o1ZdUrfMCWyWDfPhzGA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

容器中增加了一个网卡 eth1，分配了 my_net2 的 IP 172.22.16.3。现在 busybox 应该能够访问 httpd 了，验证一下：<img src="http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEwPSYbFVIEZopYzvV5rO5aiclBbH6mete7bf3ibcZgPsyCU2jOeyR2Dpf43ZAFUnxK0RRlvfdSxSNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

busybox 能够 ping 到 httpd，并且可以访问 httpd 的 web 服务。当前网络结构如图所示：![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEwPSYbFVIEZopYzvV5rO5ac9FR5kFasQHoHvfgLLWmeBucU2fQO3xwvZM2bpJBsA1aNQDwHbJiaFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)