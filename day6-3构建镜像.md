使用现成镜像构建自己的镜像显然是更好的选择,节省了构建基础镜像的时间

特别是使用那些官方镜像，因为 Docker 的工程师知道如何更好的在容器中运行软件。

当然,**某些情况下我们也不得不自己构建镜像，**

1. **找不到现成的镜像**，比如自己开发的应用程序。

2. 需要在镜像中**加入特定的功能**，比如**官方镜像几乎都不提供 ssh**。

#### docker commit法创建镜像

docker commit 命令是创建新镜像最直观的方法，其过程包含三个步骤：

1. 运行容器

2. 修改容器

3. 将容器保存为新的镜像

举个例子：在 ubuntu base 镜像中安装 vim 并保存为新镜像。

1. 第一步， 运行容器 
   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEBniciaDNNUTvHlKjxibK4VSIoUOVrYMxKk1YPezdhIbvOx4MWibEoEeLCNibFRoo9qKbORiaiadwA0Q7ibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
   `-it` 参数的作用是以交互模式进入容器，并打开终端。`412b30588f4a` 是容器的内部 ID。
2. 安装vim
   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEBniciaDNNUTvHlKjxibK4VSIzo60zInHxNmPxcgRhqiaSNuaIozR7Hra3SUffnufzDt4fD1OYibyYMQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3. 保存为新镜像
   在新窗口中查看当前运行的容器。
   ![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqEBniciaDNNUTvHlKjxibK4VSI9PvicWKubAElMsu2ZQzqIRLiburdX1HQQqCl2OrSFH1sU0hw6HqibCYyQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)`silly_goldberg` 是 Docker 为我们的容器随机分配的名字。
   执行 docker commit 命令将容器保存为镜像, 新镜像命名为 `ubuntu-with-vi`。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqG0UQD6CWkqyyvq61f2ibLDHOwZIwD4e6ckeJpFy43ibPpa7UWAVFrlkicFq3a7hyNkUibSKXucqLxcTg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



#### Dockerfile构建镜像

**第一个 Dockerfile**

用 Dockerfile 创建上节的 ubuntu-with-vi，其内容则为：

```
root@ubuntu:~# pwd         ①  

/root  

root@ubuntu:~# ls          ②   

Dockerfile   

root@ubuntu:~# docker build -t ubuntu-with-vi-dockerfile .        ③   

Sending build context to Docker daemon 32.26 kB           ④   

Step 1 : FROM ubuntu           ⑤   

 ---> f753707788c5   

Step 2 : RUN apt-get update && apt-get install -y vim           ⑥   

 ---> Running in 9f4d4166f7e3             ⑦   



......   



Setting up vim (2:7.4.1689-3ubuntu1.1) ...   

 ---> 35ca89798937           ⑧    

Removing intermediate container 9f4d4166f7e3          ⑨   

Successfully built 35ca89798937           ⑩   

root@ubuntu:~#   
```

① 当前目录为 /root。

② Dockerfile 准备就绪。

③ 运行 docker build 命令，`-t` 将新镜像命名为 `ubuntu-with-vi-dockerfile`，命令末尾的 `.` 指明 build context 为当前目录。Docker 默认会从 build context 中查找 Dockerfile 文件，我们也可以通过 `-f` 参数指定 Dockerfile 的位置。

④ 从这步开始就是镜像真正的构建过程。 首先 Docker 将 build context 中的所有文件发送给 Docker daemon。build context 为镜像构建提供所需要的文件或目录。
Dockerfile 中的 ADD、COPY 等命令可以将 build context 中的文件添加到镜像。此例中，build context 为当前目录 `/root`，该目录下的所有文件和子目录都会被发送给 Docker daemon。

所以，使用 build context 就得小心了，不要将多余文件放到 build context，特别不要把 `/`、`/usr` 作为 build context，否则构建过程会相当缓慢甚至失败。

⑤ Step 1：执行 `FROM`，将 ubuntu 作为 base 镜像。
ubuntu 镜像 ID 为 f753707788c5。

⑥ Step 2：执行 `RUN`，安装 vim，具体步骤为 ⑦、⑧、⑨。

⑦ 启动 ID 为 9f4d4166f7e3 的临时容器，在容器中通过 apt-get 安装 vim。

⑧ 安装成功后，将容器保存为镜像，其 ID 为 35ca89798937。
**这一步底层使用的是类似 docker commit 的命令**。

⑨ **删除临时容器** 9f4d4166f7e3。

⑩ 镜像构建成功。 ID:35ca89798937
通过 `docker images `查看镜像信息。 

在上面的构建过程中，我们要特别注意指令 RUN 的执行过程 ⑦、⑧、⑨。Docker 会在启动的临时容器中执行操作，并通过 commit 保存为新的镜像。



**查看镜像分层结构**

ubuntu-with-vi-dockerfile 是**通过在 base 镜像的顶部添加一个新的镜像层**而得到的。

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqH2VtvHUgNIGJTnpxeFbHRC624WceCStKuuMmiaHWrNTyedNicIkbic2nA84jF7ykpia5LxBZ4Vmj92cQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个**新镜像层的内容**由 `RUN apt-get update && apt-get install -y vim` 生成。这一点我们可以通过 `docker history` **命令验证**。

`docker history` 会显示镜像的构建历史，也就是 Dockerfile 的执行过程。
`docker history `也向我们展示了镜像的分层结构，每一层由上至下排列。

表示无法获取 IMAGE ID，通常从 Docker Hub 下载的镜像会有这个问题。
