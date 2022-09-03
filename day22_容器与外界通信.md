#### 容器访问外部世界

在我们当前的实验环境下，docker host 是可以访问外网的。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9XRg5aDq2HFCBAaCxGdNLe4f4kYU80Cre2MLqmoaia6xyRw7cvtO864w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

而且**容器默认就能访问外网**

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9gmBmSXXQUJqRXTh0nAOB7fia2ENfSf6O6HEPPT7SuvnlCrCxWOh5Jkg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





在上面的例子中，busybox 位于 `docker0` 这个私有 bridge 网络中（172.17.0.0/16），当 busybox 从容器向外 ping 时，数据包是怎样到达 bing.com 的关键就是 NAT。我们查看一下 docker host 上的 iptables 规则：

`iptables -t nat -S`![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9S03VRYPcyTSnY6Mqg6ciakAdOg3cicnOIpiaoabT9EsoakmwEUk6gyLEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其含义是：如果网桥 `docker0` 收到来自 172.17.0.0/16 网段的外出包，把它交给 MASQUERADE 处理。而 MASQUERADE 的处理方式是将包的源地址替换成 host 的地址发送出去，**即做了一次网络地址转换（NAT）**。

下面我们通过 tcpdump 查看地址是如何转换的。先查看 docker host 的路由表：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9lUbJiazj3sNnuwEC60LFmEdF5ushic6WE70zhZ7yT9D36jSLfrkwdajg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**默认路由通过 enp0s3 发出去**，所以我们要同时监控 enp0s3 和 docker0 上的 icmp（ping）数据包。

当 busybox ping bing.com 时，tcpdump 输出如下：![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9b4jwouGTxeutluM7nKcicAwPKCvLtMDiaA5RqmJicCL7z4UbPMnqhDhJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

docker0 收到 **busybox 的 ping 包，源地址为容器 IP 172.17.0.2**，这没问题，交给 MASQUERADE 处理。这时，在 enp0s3 上我们看到了变化：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9jra8R7ibkKHs5J5tz28cmffuuX6icibNugVONMyS0gzBrfRAYbM7LyH6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**ping 包的源地址变成了 enp0s3 的 IP 10.0.2.15**

这就是 iptable NAT 规则处理的结果，从而保证数据包能够到达外网。下面用一张图来说明这个过程![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGqiaPibldibtMBypjCia3YIQB9h0683TFMichVWCmrjD9qUVdhiaOuyjRib7L4jR4x9GcxR55GYWgdlXKFw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1. busybox 发送 ping 包：172.17.0.2 > www.bing.com。
2. docker0 收到包，发现是发送到外网的，交给 NAT 处理。
3. NAT 将源地址换成 enp0s3 的 IP：10.0.2.15 > www.bing.com。
4. ping 包从 enp0s3 发送出去，到达 www.bing.com。

通过 NAT，docker 实现了容器对外网的访问。



#### 外部世界如何访问容器

docker 可将**容器对外提供服务的端口映射到 host 的某个端口**，外网通过该端口访问容器。容器启动时通过**`-p`参数映射端口**：(端口映射)

`docker run -d -p 80 httpd`    将 80 端口映射到 host 的端口：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqH4HibS9e3cHlCJUGfmYRglLCX6XIw5OibSNv7NjLHH3UyD43fqZTPTmwffehZ4aAjaxZLBeicDkMF5A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



如图,容器启动后，可通过 `docker ps` 或者 `docker port` 查看到 host 映射的端口。在上面的例子中，httpd 容器的 80 端口被映射到 host 32773 上，这样就可以通过 `<host ip>:<32773>` 访问容器的 web 服务了。![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqH4HibS9e3cHlCJUGfmYRglLAHgSxKUQYYekbnjGcs1yzRDGrwMAt7pzYvU5vnLygHicE7CDiaUF6PzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



除了映射动态端口，也可在 `-p` 中指定映射到 host 某个特定端口，例如可将 80 端口映射到 host 的 8080 端口：

`docker run -d -p 8080:80 httpd` 将 httpd的80端口映射到host的8080端口：

每一个映射的端口，host 都会启动一个 `docker-proxy` 进程来处理访问容器的流量：

<img src="http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqH4HibS9e3cHlCJUGfmYRglLfbRpDlpNa0FgKmHKrRD5BlXicOxO9neIOicDZZjYus3yj9ROfvufwSzA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片"  />

以 0.0.0.0:32773->80/tcp 为例分析整个过程：![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqH4HibS9e3cHlCJUGfmYRglLrLuzSEQHwnDpWzmicEcuZ6JAm9ticcPIzjIDcXdKzqavSeuXHd0DbnmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1. docker-proxy 监听 host 的 32773 端口。
2. 当 curl 访问 10.0.2.15:32773 时，docker-proxy 转发给容器 172.17.0.2:80。
3. httpd 容器响应请求并返回结果。