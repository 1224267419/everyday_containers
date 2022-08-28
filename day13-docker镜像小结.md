### 镜像常用子命令

docker+下列命令

```
images   显示镜像列表
history  显示镜像构建历史
commit   从容器创建新镜像
build   从 Dockerfile 构建镜像
tag    给镜像打 tag
pull    从 registry 下载镜像
push    将 镜像 上传到 registry
rmi    删除 Docker host 中的镜像
search   搜索 Docker Hub 中的镜像
```


除了 rmi 和 search，其他命令都已经用过了。

**rmi**

rmi 只能删除 host 上的镜像，不会删除 registry 的镜像

如果一个镜像对应了多个 tag，只有**当最后一个 tag 被删除时，镜像才被真正删除**。例如 host 中 debian 镜像有两个 tag：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEDhn6EE9JvZoD1ZwQLQOpA7kVA1iaMzrFffibGOcNz7vNaQwyCG2HPRnyoIzLbmkIpnIdDIibINQFfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

删除其中 debian:latest 只是删除了 latest tag，镜像本身没有删除。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEDhn6EE9JvZoD1ZwQLQOpAvhoDpoU1ZiaoUyZWqlq19Hf0A31jVcueLykwrWibNxib9eLHiarIYbwAhQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

只有当 debian:jessie 也被删除时，整个镜像才会被删除。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEDhn6EE9JvZoD1ZwQLQOpAZ1JuxxGIIPhKAtXLVoLj4K1OzDbXHNCM5Rfj7yorVlnDWpCoSvETcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)**search**

search 让我们无需打开浏览器，在命令行中就可以搜索 Docker Hub 中的镜像。
效果如图

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEDhn6EE9JvZoD1ZwQLQOpAAF2bP8JoYJ7ZgF9f2W6gibIwQdU8bzVZPlY6ibz1kSyRPb2q7w1jtqxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

至于镜像的tag还得去看Docker Hub网站