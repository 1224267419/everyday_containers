RUN、CMD 和 ENTRYPOINT运行指令的格式



Shell 格式

shell 格式底层会调用 /bin/sh -c <command>。

<instruction> <command>


例如：

```
RUN apt-get install python3  

CMD echo "Hello world"  

ENTRYPOINT echo "Hello world" 
```



Exec 格式

会直接调用 <command>，不会被 shell 解析。

<instruction> ["executable", "param1", "param2", ...]

例如：

```
RUN ["apt-get", "install", "python3"]  

CMD ["/bin/echo", "Hello world"]  

ENTRYPOINT ["/bin/echo", "Hello world"]
```



**CMD 和 ENTRYPOINT 推荐使用 Exec 格式**，因为指令可读性更强，更容易理解。RUN 则两种格式都可以。



### RUN

RUN 指令通常用于安装应用和软件包。
RUN 在当前镜像的顶部执行命令，并通过创建新的镜像层。

下面是使用 RUN 安装多个包的例子：

```
RUN apt-get update && apt-get install -y \  

 bzr \

 cvs \

 git \

 mercurial \

 subversion
```

**apt-get update 和 apt-get install 被放在一个 RUN 指令中执行**，这样能够保证每次安装的是最新的包。如果 apt-get install 在单独的 RUN 中执行，则会使用 apt-get update 创建的镜像层，而这一层可能是很久以前缓存的。(缓存机制的影响)



#### **CMD**

CMD 指令允许用户指定容器的**默认执行的命令**。

1. 如果 docker run 指定了其他命令，CMD 指定的默认命令将被忽略。
2. 如果 Dockerfile 中有多个 CMD 指令，只有**最后一个 CMD 有效**。

推荐Exec格式:CMD ["executable","param1","param2"]



CMD ["param1","param2"] 为 ENTRYPOINT 提供额外的参数，此时 ENTRYPOINT 必须使用 Exec 格式。



Dockerfile 片段如下：

`CMD echo "Hello world"`



运行容器 docker run -it [image] 将输出：

`Hello world`

比如 docker run -it [image] /bin/bash，CMD 会被忽略掉，命令 bash 将被执行：

`root@10a32dc7d3d3:/#`,即CMD默认命令被忽略



#### **ENTRYPOINT**

和CMD很像,区别在于**不会被忽略,一定会执行**

推荐使用Exec模式

ENTRYPOINT 中的参数始终会被使用，而 CMD 的额外参数可以在容器启动时动态替换掉。
**ENTRYPOINT 的 Shell 格式会忽略任何 CMD 或 docker run 提供的参数。**