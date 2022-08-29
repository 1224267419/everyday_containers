控制容器使用资源多少是十分重要的,能避免太多资源占用影响性能

#### **内存限额**

与操作系统类似，容器可使用的内存包括两部分：物理内存和 swap。 Docker 通过下面两组参数来控制容器内存的使用量。

1. `-m` 或 `--memory`：设置内存的使用限额，例如 100M, 2G。
2. `--memory-swap`：设置 **内存+swap** 的使用限额。

for example:

`docker run -m 200M --memory-swap=300M ubuntu`

其含义是允许该容器**最多使用 200M 的内存和 100M 的 swap**。默认情况下，上面两组参数为 -1，即对容器内存和 swap 的使用没有限制。

下面我们将使用 progrium/stress 镜像来学习如何为容器分配内存。该镜像可用于对容器执行压力测试。执行如下命令：

`docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 280M`

`--vm 1`：启动 1 个内存工作线程。

`--vm-bytes 280M`：每个线程分配 280M 内存。

![image-20220829195322241](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220829195322241.png)

如图,重复分配内存,释放内存的操作

注意,内存线程*每个线程分配的内存应<`--memory-swap=`分配的总内存

![image-20220829195749587](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220829195749587.png)

超过了结果如上图,分配的内存超过限额，stress 线程报错，容器退出。

如果在启动容器时只指定 `-m` 而不指定 `--memory-swap`，那么 `--memory-swap` 默认为 `-m` 的两倍，比如：

`docker run -it -m 200M ubuntu`

容器最多使用 200M 物理内存和 200M swap。(共计400M)



#### CPU资源限制

Docker 可以通过 `-c` 或 `--cpu-shares` 设置容器使用 CPU 的权重。如果不指定，**默认值为 1024**。

通过 `-c` 设置的 cpu share 是一个相对的权重值。某个容器最终能分配到的 CPU 资源取决于它的 cpu share 占所有容器 cpu share 总和的比例,即**通过 cpu share 可以设置容器使用 CPU 的优先级**。

`docker run --name "container_A" -c 1024 ubuntu`
`docker run --name "container_B" -c 512 ubuntu`

如上两个容器,当两个容器都需要 CPU 资源时，container_A 可以得到的 CPU 是 container_B 的两倍。
如果 container_A 处于空闲状态，这时，为了充分利用 CPU 资源，container_B 也可以分配到全部可用的 CPU。

`docker run --name container_A -it -c 1024 stress --cpu 1`
`docker run --name container_B -it -c 512 stress --cpu 1
--cpu` 用来设置工作线程的数量,这里假设只有一颗cpu,所以1个工作线程就能将 CPU 压满。如果 host 有多颗 CPU，则需要相应增加 `--cpu` 的数量。

在 host 中执行 `top`，查看容器对 CPU 的使用情况： 
![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqE8BWUXVcR7QxuXDHmpGd8omMwuuOlYLANNJjLVQfSYO9XLSRTiaHgiaU0DPOXTWUue1xHFzkGp6kHQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

用`docker pause container_A`暂停container_A,则

`top` 显示 container_B 在 container_A 空闲的情况下能够用满整颗 CPU： 
![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqE8BWUXVcR7QxuXDHmpGd8oHPIJ31oWbN0LpHsXWrMFo0BWs1aWiaXFn41oRFdj5WMjHC7qT8oYlRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





#### 限制Block IO

Block IO 指的是磁盘的读写，docker 可通过设置权重、限制 bps 和 iops 的方式控制容器读写磁盘的带宽

默认情况下所有容器能平等地读写磁盘，可以通过设置 `--blkio-weight` 参数来改变容器 block IO 的优先级。

`--blkio-weight` 与 `--cpu-shares` 类似，设置的是相对权重值，**默认为 500**。



```
docker run -it --name container_A --blkio-weight 600 ubuntu

docker run -it --name container_B --blkio-weight 300 ubuntu
```

如上container_A**读写磁盘的带宽**是 container_B 的**两倍**

bps 是 byte per second，每秒读写的数据量。
iops 是 io per second，每秒 IO 的次数。

可通过以下参数控制容器的 bps 和 iops：
`--device-read-bps`，限制读某个设备的 bps。
`--device-write-bps`，限制写某个设备的 bps。
`--device-read-iops`，限制读某个设备的 iops。
`--device-write-iops`，限制写某个设备的 iops。

`docker run -it --device-write-bps /dev/sda:30MB ubuntu` 限制容器写 /dev/sda 的速率为 30 MB/s
因为容器的文件系统是在 host /dev/sda 上的，**在容器中写文件相当于对** host /dev/sda **进行写操作**。另外，`oflag=direct` 指定用 direct IO 方式写文件，这样 `--device-write-bps` 才能生效。