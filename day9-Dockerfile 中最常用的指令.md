Dockerfile 中最常用的指令

**FROM**
指定 base 镜像。

**MAINTAINER**
作者

**COPY**
将文件从 build context 复制到镜像。
COPY 支持两种形式：

1. COPY src dest
2. COPY ["src", "dest"]

注意：src 只能指定 build context 中的文件或目录。

**ADD**

与 COPY 类似,如果 src 是归档文件（tar, zip, tgz, xz 等），文件会被自动解压到 dest。

**EXPOSE**
指定容器中的进程会监听某个端口，Docker 可以将该端口暴露出来。

**RUN**
在容器中运行指定的命令。

**CMD**
容器启动时运行指定的命令。
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换。