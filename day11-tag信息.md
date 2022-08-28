如何在各个host上使用镜像

1. dockerfile构建
2. 上传到公共 Registry,再下载使用
3. 搭建私人Registry

当我们执行 `docker build` 命令时已经为镜像取了个名字，例如前面：

`docker build -t ubuntu-with-vi`,  ubuntu-with-vi就是名字

`docker build -t ubuntu-with-vi:latest`和上面是一样的
只是显式地写出了tag信息(默认值是latest),tag可以用于区分小版本





假设我们现在发布了一个镜像 myimage，版本为 v1.9.1。那么我们可以给镜像打上四个 tag：1.9.1、1.9、1 和 latest。

![img](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqHd3gqdnC7x54x2TbI3TTPohksSPv5UHUfOZ4uUBTibFrFBYNUVvgX70uRkkAyv6vR29ZjeJ4B3LRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以通过 docker tag 命令方便地给镜像打 tag。

```
docker tag myimage-v1.9.1 myimage:1
docker tag myimage-v1.9.1 myimage:1.9
docker tag myimage-v1.9.1 myimage:1.9.1
docker tag myimage-v1.9.1 myimage:latest
```

过了一段时间，我们发布了 v1.9.2。这时可以打上 1.9.2 的 tag，并将 1.9、1 和 latest 从 v1.9.1 移到 v1.9.2。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqHd3gqdnC7x54x2TbI3TTPoDnMAPhdZNvZUMWMz4SZlascSdaMicnJGpjzsrl1ficvHicMsXwMUG5ibMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

命令为：

```
docker tag myimage-v1.9.2 myimage:1
docker tag myimage-v1.9.2 myimage:1.9
docker tag myimage-v1.9.2 myimage:1.9.2
docker tag myimage-v1.9.2 myimage:latest
```

这种 tag 方案使镜像的版本很直观，用户在选择非常灵活：

1. myimage:1 始终指向 1 这个分支中最新的镜像。
2. myimage:1.9 始终指向 1.9.x 中最新的镜像。
3. myimage:latest 始终指向所有版本中最新的镜像。
4. 如果想使用特定版本，可以选择 myimage:1.9.1、myimage:1.9.2 或 myimage:2.0.0。

**Docker Hub 上很多 repository 都采用这种方案，所以大家一定要熟悉。**
