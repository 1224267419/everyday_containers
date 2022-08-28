Docker 支持通过扩展现有镜像，创建新的镜像。![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFEMKBODWC1dUhel7ZP5umPaqkbT8QpCjmE8yW4nicr7Tc9x9iaUY2ggVYEcPL4Lfmao69PgKb3XOTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如此图

① 新镜像不再是从 scratch 开始，而是直接在 Debian base 镜像上构建。
② 安装 emacs 编辑器。
③ 安装 apache2。
④ 容器启动时运行 bash。![图片](http://mmbiz.qpic.cn/mmbiz_png/Hia4HVYXRicqFEMKBODWC1dUhel7ZP5umPmIx4x6MY4SAnKoLOibiahqQpKuEISvrGhKJLmZ84qQgUqwJLlZutLFsg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

使用分层的结构能共享资源,有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上**保存一份 base 镜像**；同时内**存中也只需加载一份** base 镜像，就可以为**所有容器服务**了。而且镜像的**每一层都可以被共享**

#### 容器层记录对镜像的修改，所有镜像层都是只读的，不会被容器修改，

#### 容器 Copy-on-Write 特性

多个容器**共享一份基础镜像**，**所有对容器的改动** - 无论添加、删除、还是修改文件**都只会发生在容器层中。**

不同层中有一个相同路径的文件，比如 /a，上层的 /a 会覆盖下层的 /a，也就是说用户只能访问到上层中的文件 /a。在容器层中，用户看到的是一个叠加之后的文件系统。

1. **添加文件**
   在容器中创建文件时，新文件被添加到**容器层**中。
2. **读取文件**
   在容器中读取某个文件时，Docker 会**从上往下**依次在各镜像层中查找此文件。

3. **修改文件**
   在容器中修改已存在的文件时，Docker 会**从上往下依次在各镜像层中查找**此文件。一旦找到，立即**将其复制到容器层，然后修改之**。
4. **删除文件**
   在容器中删除文件时，Docker 也是**从上往下依次在镜像层中查找此文件**。找到后，会**在容器层中记录下此删除操作。**

只有当需要修改时才复制一份数据，这种特性被称作 Copy-on-Write。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。

所以**镜像可以被多个容器共享**。