**cgroup** 和 **namespace** 是最重要的两种底层技术。cgroup 实现**资源限额**， namespace 实现**资源隔离**。

### cgroup

cgroup 全称 Control Group。Linux 操作系统通过 cgroup 可以设置进程使用 CPU、内存 和 IO 资源的限额。run时设置的参数`--cpu-shares`、`-m`、`--device-write-bps` 实际上就是在配置 cgroup。

cgroup在sys/fs/cgroup/cpu/docker 目录中，Linux 会为每个容器创建一个 cgroup 目录，以容器长ID 命名,
![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFwNdffEsH0228BG3jM87yavaPxdSTHbibYhqJGYicG7UPaicFjcfyAJnZo8TXVKbkZzMdNXOY0aqudQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同样的，/sys/fs/cgroup/memory/docker 和 /sys/fs/cgroup/blkio/docker 中保存的是内存以及 Block IO 的 cgroup 配置。



### namespace

namespace 管理着 host 中全局唯一的资源，并可以让每个容器都觉得只有自己在使用它。换句话说，**namespace 实现了容器间资源的隔离**。

Linux 使用了六种 namespace，分别对应六种资源：**Mount、UTS、IPC、PID、Network 和 User**，下面我们分别讨论。



#### **Mount namespace**

Mount namespace 让容器看上去拥有自己的 `/` 目录，可以执行 `mount` 和 `umount` 命令。当然这些操作只在当前容器中生效，不会影响到 host 和其他容器。



#### **UTS namespace**

简单的说，UTS namespace 让容器有自己的 hostname。 **默认情况下**，容器的 **hostname** 是它的**短ID**，可以通过 `-h` 或 `--hostname` 参数设置。



#### **IPC namespace**

IPC namespace **让容器拥有自己的共享内存和信号量**（semaphore）来实现进程间通信，而**不会与 host 和其他容器的 IPC 混在一起**。



#### **PID namespace**

**容器在 host 中以进程的形式运行**。例如当前 host 中运行了两个容器：通过 `ps axf` 可以查看容器进程：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFwNdffEsH0228BG3jM87yaSUDXGXxrVMCkqrTmwgHo6eDw8Vwk2jF63Je5WmbbGJvByibLOVicEMvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所有容器的进程都挂在 dockerd 进程下，同时也可以看到容器自己的子进程。 如果我们进入到某个容器，`ps` 就只能看到自己的进程了：

![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFwNdffEsH0228BG3jM87yamZibzXdB7ctK9htR17sMk9AiaDNoEUTrKdvnk5OqZscC4PWhjnuY9DfQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

进程的 PID 不同于 host 中对应进程的 PID，容器中 PID=1 的进程当然也不是 host 的 init 进程。也就是说：**容器拥有自己独立的一套 PID**



#### **Network namespace**

Network namespace 让容器拥有自己独立的网卡、IP、路由等资源。我们会在后面网络章节详细讨论。



#### **User namespace**

User namespace 让**容器能够管理自己的用户，host 不能看到容器中创建的用户**。![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFwNdffEsH0228BG3jM87yaxvxGUgwDzr3HOuIgW7fjTQiaBg8vycRBB3PAUcicId7M3uxaT2WCf9zw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
容器中创建了用户 cloudman，但 host 中并不会创建相应的用户。

容器内和host账号不互通,体现容器独立性

下面是容器的常用操作命令：

```
create    创建容器  
run     运行容器  
pause    暂停容器  
unpause   取消暂停继续运行容器  
stop     发送 SIGTERM 停止容器  
kill     发送 SIGKILL 快速停止容器  
start    启动容器  
restart   重启容器  
attach    attach 到容器启动进程的终端  
exec     在容器中启动新进程，通常使用 "-it" 参数  
logs     显示容器启动进程的控制台输出，用 "-f" 持续打印  
rm      从磁盘中删除容器
```

