Docker 会缓存已有镜像的镜像层，构建新镜像时，如果某镜像层已经存在,无需重新创建。

在前面的 Dockerfile 中添加一点新内容，往镜像中复制一个文件：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFpR0GJE0I5fjibozEQaYcYibVicXs3oJKV1ycKpDpf8ia6npmoXPJ7Zu1NptAr2Oic9cA8EXx6Cibct3fQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



如果我们希望在构建镜像时不使用缓存，可以在 `docker build` 命令中加上 `--no-cache` 参数Dockerfile 中**每一个指令都会创建一个镜像层，上层是依赖于下层的**。无论什么时候，只要**某一层发生变化**，其**上面所有层的缓存都会失效**。



如果交换前面 RUN 和 COPY 的顺序：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFpR0GJE0I5fjibozEQaYcYibgr2vHSjCqdvJM4ATO8ZRwsNqYef6Fq0Vr9LZ967SRyEDC9qEiafNGRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

虽然在逻辑上这种改动对镜像的内容没有影响，但由于分层的结构特性，**Docker 必须重建受影响的镜像层。**

```
root@ubuntu:~# docker build -t ubuntu-with-vi-dockerfile-3 .

Sending build context to Docker daemon 37.89 kB

Step 1 : FROM ubuntu

 ---> f753707788c5

Step 2 : COPY testfile /

 ---> bc87c9710f40

Removing intermediate container 04ff324d6af5

Step 3 : RUN apt-get update && apt-get install -y vim

 ---> Running in 7f0fcb5ee373

Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]

......
```

从上面的输出可以看到**生成了新的镜像层** bc87c9710f40，**缓存失效**。

除了构建时使用缓存，Docker 在下载镜像时也会使用。