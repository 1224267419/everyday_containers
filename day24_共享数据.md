#### docker和host共享数据

对于bind mount,直接将共享目录mount到容器

对于docker managed volume,volume 位于 host 中的目录，是在容器启动时才生成，所以需要将共享数据拷贝到 volume 中。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEXy7icsYEI4Piby9sCkWek1OcpmWvfY67fUvZAs7LCxjHmPAGfhh6oOm1hmoEeP5HB2GtEJUGkT1KQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如图,docker cp将容器中的文件复制到host中,当然直接使用linux自带的cp也可以



#### 容器间共享数据

##### 1.共享数据在bind mount中,然后mount  到多个容器

我们要创建由三个 httpd 容器组成的 web server 集群，它们使用相同的 html 文件，操作如下：

1. 将 $HOME/htdocs mount 到三个 httpd 容器。
   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEXy7icsYEI4Piby9sCkWek1OXbVQOdkicnpHbqQNIDApzL0EnkYsq9NZibC09XmuYkjootXmAjJlFILA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
2. 查看当前主页内容。
   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEXy7icsYEI4Piby9sCkWek1OI2MFyD8zgo5L0WxMoMroZZONcNBdwIBnCmmdkQAtWDs8Mtza5qUwYg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
3. 修改 volume 中的主页文件，再次查看并确认所有容器都使用了新的主页。
   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEXy7icsYEI4Piby9sCkWek1OVqYHQtEfkibKia3wzKFoT6LfAyouQ9An3vEw2QbqCOqicnECeGGXNfJKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所有容器使用同一主页

##### 2.用 volume container 共享数据

volume container 是专**门为其他容器提供 volume 的容器**。它提供的卷可以是 bind mount，也可以是 docker managed volume

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGIqC8nut9TEnPVBSQCBfGeKzIolu2d6r2oL0Kj701hKAKJt2yWCtbp1tjvr94vyrGC4vNYSWOhAw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如图,用docker create创建了`vc_data`容器,vc容器仅用于提供volume,不启动也可以,所以用create
红框第一行用的bind mount,存放静态文件
		第二行用的docker managed volume

其他容器可以通过 `--volumes-from` 使用 `vc_data` 这个 volume container：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGIqC8nut9TEnPVBSQCBfGeAI6alVibLiaPP8pSKTzteibQIZ0eicUSbuafrzuSoda2euIOLODmeiakRMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上图,`docker run --name web1 -d -p  80 --volumes-from vc_data httpd`

web1 容器使用的就是 vc_data 的 volume，而且连 mount point 都是一样的。验证一下数据共享的效果：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqGIqC8nut9TEnPVBSQCBfGeHNmpulnqmrCvibBvhhZ2Zrd6QShuYia6Bjrg01tkXU5jWXxtRXwzJzfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

可见，三个容器已经成功共享了 volume container 中的 volume。

下面我们讨论一下 volume container 的特点：

1. 与 bind mount 相比，不必为每一个容器指定 host path，所有 path 都在 volume container 中定义好了，**容器只需与 volume container 关联，实现了容器与 host 的解耦。**
2. 使用 volume container 的容器其 mount point 是一致的，**有利于配置的规范和标准化**，但也带来一定的局限，使用时需要综合考虑。

##### 3.data-packed volume container

data-packed volume container 将数据完全放到 volume container 中，同时又能与其他容器共享
其原理是将数据打包到镜像中，然后通过 docker managed volume 共享。



我们用下面的 Dockfile 构建镜像：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFyKpRuEpbupibvkZhsODwctn0wsfT7F9OlbAc3cy1suCkb4A8c6vX6F6XpFsapo3w5NkZEx5uQMRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

`ADD` 将静态文件添加到容器目录 /usr/local/apache2/htdocs。
`VOLUME` 的作用与 `-v` 等效，用来创建 docker managed volume，mount point 为 /usr/local/apache2/htdocs，因为这个目录就是 `ADD` 添加的目录，所以会将已有数据拷贝到 volume 中。

build 新镜像 datapacked：(-t为容器重新分配一个伪输入终端，通常与 -i 同时使用；)

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFyKpRuEpbupibvkZhsODwctgVicLkbyQiaUbdRjFuud5kFgTWz5j0xTtpfmN7Nq2ffoHvRInBAGAKgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

用新镜像创建 data-packed volume container：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFyKpRuEpbupibvkZhsODwctibFv0XRouAYiaFVNqAHF5StEHTQMKo9rrd1HvUicXPw0ibTUlgAiahLKHXw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

因为在 Dockerfile 中已经使用了 `VOLUME` 指令，这里就不需要指定 volume 的 mount point 了。启动 httpd 容器并使用 data-packed volume container：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFyKpRuEpbupibvkZhsODwctXztXyf8cSia3aNToOvVLC4wjDLUJoQ57kll0D1LbUQj9ByuwFawib9Gw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

容器能够正确读取 volume 中的数据。**data-packed volume container 是自包含的，不依赖 host 提供数据，具有很强的移植性**，非常适合 **只使用** **静态数据的**场景，比如应用的配置信息、web server 的静态文件等。