## volume 生命周期管理



本节将讨论如何备份、恢复、迁移和销毁 volume。



#### **备份**

因为 volume 实际上是 host 文件系统中的目录和文件，所以 **volume 的备份实际上是对文件系统的备份**。

所有的本地镜像都存在 host 的 /myregistry 目录中，我们要做的就是定期备份这个目录。



#### **恢复**

volume 的恢复也很简单，如果数据损坏了，直接用之前备份的数据拷贝到 /myregistry 就可以了。



#### 迁移

如果我们想使用更新版本的 Registry，这就涉及到数据迁移，方法是：

1. `docker stop` 当前 Registry 容器。

2. 启动新版本容器并 mount 原有 volume。

   

   `docker run -d -p 5000:5000 -v /myregistry:/var/lib/registry registry:latest`

当然，在启用新容器前要确保新版本的默认数据路径是否发生变化。



#### **销毁**

可以删除不再需要的 volume，但一定要确保知道自己正在做什么，volume 删除后数据是找不回来的。

对于bind mount 删除只能由host负责
对于docker managed volume,在执行 `docker rm` 删除容器时可以**带上 `-v` 参数，docker 会将容器使用到的 volume 一并删除，但前提是没有其他容器 mount 该 volume**，目的是保护数据，非常合理。



但如果删除时没有`-v`参数,就会导致volume独立存在,浪费存储空间,因此 docker 提供了 volume 子命令可以对 docker managed volume 进行维护。
![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEtn1GAYztccOIyLOSXkTFgKDzIhFwVwHHj8cUpBVJIyTIicPSF69kqqAiaXbSCqKnhkz5HNJibvYjSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



容器 bbox 使用的 docker managed volume 可以通过 `docker volume ls` 查看到。

删除 bbox：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEtn1GAYztccOIyLOSXkTFgJQ1o2vFpxoe6YXMy57BRTwHgor2MgCqS6ONYUWHQUVl6AiahH0icQTibQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

因为没有使用 `-v`，volume 遗留了下来。对于这样的孤儿 volume，**可以用 `docker volume rm` 删除**：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEtn1GAYztccOIyLOSXkTFgfib4mUaT85QFpEyY9MV8dFicfPicUF4rpElhA2QoCbW9euPSaWdJnicvxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



`docker volume rm $(docker volume ls -q)` 
删除所有孤儿volume

### **小结**

本章我们学习了以下内容：

1. docker 为容器提供了两种存储资源：数据层和 Data Volume。
2. 数据层包括镜像层和容器层，由 storage driver 管理。
3. Data Volume 有两种类型：bind mount 和 docker managed volume。
4. bind mount 可实现容器与 host 之间，容器与容器之间共享数据。
5. volume container 是一种具有更好移植性的容器间数据共享方案，特别是 data-packed volume container。
6. 