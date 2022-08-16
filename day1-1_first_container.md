day1:在移动硬盘上配置好了虚拟机,并使用

`docker run -d -p 80:80 httpd`

运行了第一个容器(镜像在线上下载)

其过程可以简单的描述为：

1. 从 Docker Hub 下载 httpd 镜像。镜像中已经安装好了 Apache HTTP Server
2. 启动 httpd 容器，并将容器的 80 端口映射到 host 的 80 端口。
3. 在浏览器中输入 http://[your ubuntu host IP],就可以访问容器的 http 服务了，第一个容器运行成功！我们轻轻松松就拥有了一个 WEB 服务器。随着学习的深入，会看到容器技术带给我们更多的价值。