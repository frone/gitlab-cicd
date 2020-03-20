![img](https://upload-images.jianshu.io/upload_images/1044756-28cdc48f58802c20.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Dockerfile语法由两部分构成，注释和命令+参数



```bash
# Line blocks used for commenting
command argument argument ..
```

一个简单的例子



```csharp
#第一行必须指令基于的基础镜像
FROM ubutu

#维护者信息
MAINTAINER docker_user  docker_user@mail.com

#镜像的操作指令
RUN apt-get update && apt-get install -y ngnix 
RUN echo "\ndaemon off;">>/etc/ngnix/nignix.conf

#容器启动时执行指令
CMD /usr/sbin/ngnix
```

Dockerfile 命令

### `FROM`

基础镜像可以为任意镜像。如果基础镜像没有被发现，Docker将试图从Docker image index来查找该镜像。FROM命令必须是Dockerfile的首个命令。如果同一个DockerFile创建多个镜像时，可使用多个FROM指令（每个镜像一次）



```bash
# Usage: FROM [image name]
FROM ubuntu 
```

### `MAINTAINER`

指定维护者的信息，并应该放在FROM的后面。



```bash
# Usage: MAINTAINER [name]
MAINTAINER authors_name 
```

### `RUN`

RUN命令是Dockerfile执行命令的核心部分。它接受命令作为参数并用于创建镜像。不像CMD命令，RUN命令用于创建镜像（在之前commit的层之上形成新的层）。
 格式为Run 或者Run [“executable” ,”Param1”, “param2”]
 前者在shell终端上运行，即/bin/sh -C，后者使用exec运行。例如：RUN [“/bin/bash”, “-c”,”echo hello”]
 每条run指令在当前基础镜像执行，并且提交新镜像。当命令比较长时，可以使用“/”换行。



```csharp
# Usage: RUN [command]
RUN apt-get update 
```

### `USER`

格式为 USER daemon 。
 指定运行容器时的用户名或UID，后续的 RUN 也会使用指定用户。
 当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如： RUN groupadd -r postgres && useradd -r -g postgres postgres 。要临时获取管理员权限可以使用 gosu ，而不推荐 sudo 。



```bash
# Usage: USER [UID]
USER 751
```

### `VOLUME`

VOLUME命令用于让你的容器访问宿主机上的目录。
 格式为 VOLUME [“/data”] 。
 创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放[数据库](https://link.jianshu.com?t=http://lib.csdn.net/base/mysql)和需要保持的数据等。



```objectivec
# Usage: VOLUME ["/dir_1", "/dir_2" ..]
VOLUME ["/my_files", "/app_files"]
```

### `WORKDIR`

WORKDIR命令用于设置CMD指明的命令的运行目录。
 格式为 WORKDIR /path/to/workdir 。
 为后续的 RUN 、 CMD 、 ENTRYPOINT 指令配置工作目录。
 可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如:



```bash
WORKDIR /a 
WORKDIR b 
WORKDIR c 
RUN pwd 
# 最终路径为 /a/b/c 。
```

### `CMD`

支持三种格式：
 CMD [“executable” ,”Param1”, “param2”]使用exec执行，推荐
 CMD command param1 param2，在/bin/sh上执行
 CMD [“Param1”, “param2”] 提供给ENTRYPOINT做默认参数。
 每个容器只能执行一条CMD命令，多个CMD命令时，只最后一条被执行。



```objectivec
# Usage 1: CMD application "argument", "argument", ..
CMD "echo" "Hello docker!"
```

### `ENV`

格式为 ENV 。 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。



```bash
ENV TZ "Asia/Shanghai"
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
```

### `ADD`

ADD命令有两个参数，源和目标。它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统。如果源是一个URL，那该URL的内容将被下载并复制到容器中。如果文件是可识别的压缩格式，则docker会帮忙解压缩。



```bash
# Usage: ADD [source directory or URL] [destination directory]
ADD /my_app_folder /my_app_folder 
```

###  `COPY`（基本于ADD没有区别）

COPY 将文件从路径 <src复制添加到容器内部路径 <dest>。



```xml
COPY <src> <dest>
```

### `EXPOSE`

指定在docker允许时指定的端口进行转发



```xml
EXPOSE <port>[<port>...]
```

### `ENTRYPOINT`

两种格式：
 ENTRYPOINT ["executable", "param1", "param2"]
 ENTRYPOINT command param1 param2（shell中执行）。
 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。
 每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

### `ONBUILD`

ONBUILD 指定的命令在构建镜像时并不执行，而是在它的子镜像中执行