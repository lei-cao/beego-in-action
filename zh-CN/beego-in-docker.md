装在 Docker 里面的 Beego
================

[**English Version**](../en-US/beego-in-docker.md)

[Docker](http://docker.io) 是一个快速创建一致的开发和部署环境的工具. 每一个程序员已经习惯了去一遍又一遍的安装和配置开发环境. 我们已经习惯了各种各样的系统库不兼容的 bug. 每次升级系统, 我们的开发环境都可能轻易的崩溃. 我们还经常遇到开发环境和生产环境不一致导致的奇怪的 bug. 幸运的是我们现在有了 Docker. 小伙伴们要是想愉快的开发, Docker 是你必须要尝试的工具.

Docker 最近很火, 已经有很多文章介绍 Docker, 小伙伴们可以自己去看.  这里我会介绍如何结合 Docker 来进行 [Beego](http://beego.me) 的开发.

- [Docke homepage](https://www.docker.com/)
- [Docker: Lightweight Linux Containers for Consistent Development and Deployment](http://www.linuxjournal.com/content/docker-lightweight-linux-containers-consistent-development-and-deployment)
- [Docker: Using Linux Containers to Support Portable Application Deployment](http://www.infoq.com/articles/docker-containers)
- [Deploying Go servers with Docker](https://blog.golang.org/docker)

 在对 Docker 有了基本的了解之后,  让我们开始用 Docker 进行 Beego 开发吧.  我们[最后的程序](http://book.beego.me/)可以在这里看到. 下面是这个部分的目录:

1. [创建服务器](#setup)

2. [安装 Docker](#install)

3. [使用 Golang 的 Docker Image](#golang-docker-image) 

 * [不用安装的 Golang env](#golang-env)

 * [hello world](#hello-world)

4. [创建 Beego 的 Docker Image](#beego-docker-image)

5. [使用 Mysql Docker image](#mysql-docker-image)

6. [使用 Nginx Docker image](#nginx-docker-image)

7. [联合使用这些 Docker](#wire-up)

8. [总结](#conclusion)  



1. <a name="setup"></a>创建服务器
-------------

这个例子是在线上的服务器. 当然小伙伴们也可以在自己的本机来玩.  没有本质的区别.

我的服务器是在 [Linode](https://www.linode.com/) 买的, 安装了 Ubuntu 14.10.  然后讲 `book.beego.me`  指导了这个服务器.


2. <a name="install"></a>安装 Docker
-------------

首先你需要安装 Docker. 不同的系统安装方式可能会有不同.  你可以根据自己的系统, 参考官方文档来安装 [installation](https://docs.docker.com/installation/#installation).

对于我的 Ubuntu 服务器, 只需要这么三行命令:
```bash
$ sudo apt-get update
$ sudo apt-get install docker.io
$ source /etc/bash_completion.d/docker.io
```

3. <a name="golang-docker-image"></a>使用 Golang 的 Docker Image[Golong Docker image](https://registry.hub.docker.com/_/golang/).
-------------

Docker 有一个 [Golang 官方库](https://registry.hub.docker.com/_/golang/). 你可以直接用它最为你的 Go 开发环境. 然后我们会从 Hello World 开始.

 * ### <a name="golang-env"></a>不用安装的 Golang env

 你只要运行下面的命令就可以从 [Golang  官方库](https://registry.hub.docker.com/_/golang/) 创建一个包括 Golang 开发环境的 Docker Container.

`docker run -it golang /bin/bash`

这时, 你已经有了最新的 Golang 开发环境:

`go version`

 你会看到如下图的结果. 我通过在 Golang Image 上运行 `/bin/bash` 来创建了一个 Container. `-it` 选项会让 Docker 创建一个可交互的命令行界面. 之后在这个新建的 Container 内, 我输入 `go version` 来测试我的 Golang 环境.

![](../images/docker.golang.png?raw=true)

 在你第一次运行 `docker run <image name>` 的时候, 如果 Docker 没有在本机找到对应的 Image, 则会自动到 Docker 库中寻找.

通过下面的命令, 你可以查看本地存在的 Docker Image.

`docker images`

通过下面的命令, 你可以查看你创建的 Container.

`docker ps -a`


 * ### <a name="hello-world"></a>Hello World

`docker run -it -v /home/leicao/programming/go/hello-world/:/go/src/hello-world golang /bin/bash`

从 Golang Image 创建了 Container 之后, 你就可以在这个环境上面编译 Go 程序了.

`go build hello.go`
`./hello`

你会看到类似如下的结果:

![](../images/docker.golang.hello-world.png?raw=true)

你可能会纠结这一切是怎么来得. 我来解释一下:

在上面的图片中:

`/home/leicao/programming/go/hello-world/hello.go` 是一个存在我的 Ubuntu 主机里面的一个 Go 的 hello world 源码:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

接着我们运行了下面的命令来创建我们的 Docker Container:

`docker run -it -v /home/leicao/programming/go/hello-world/:/go/src/hello-world golang /bin/bash`

我们使用了选项 `-v /home/leicao/programming/go/hello-world/:/go/src/hello-world`, 它将我的 Ubuntu 主机上的目录 `/home/leicao/programming/go/hello-world/` 挂载到创建的 Docker Container 的 `/go/src/hello-world` 目录.

这样, 在我的 Docker Container 里面, 我便可以得到我的 hello world 源码了. 之后我 `cd src/hello-world/` 并编译它 `go build hello.go` 最后 `./hello` 来运行它.

是不是很爽? 有了 Docker, 我们就不用再自己安装和配置开发环境了, 一切都可以直接拿来用.


4. <a name="beego-docker-image"></a>[创建 Beego 的 Docker Image](https://registry.hub.docker.com/u/leicao/beego/).
-------------------

我们现在有了最基本的 Golang 开发环境, 我们可以在它的基础上进行扩展, 从而支持 Beego 的开发. 我已经创建了一个最基本的 [Beego Image](https://registry.hub.docker.com/u/leicao/beego/), 你可以直接拿来用. 实际上, 你自己 build Docker image 也是非常的容易的. 

首先创建一个 `Dockerfile`:

```dockerfile
# Base image is in https://registry.hub.docker.com/_/golang/
# Refer to https://blog.golang.org/docker for usage
FROM golang:1.3.3
MAINTAINER Lei Cao lexo.charles@gmail.com

# ENV GOPATH /go

# Install beego & bee
RUN go get github.com/astaxie/beego
RUN go get github.com/beego/bee
```

`FROM golang:1.3.3` 这个 Docker image 从官方的 Golang Image 创建,
之后我们运行 `RUN go get github.com/astaxie/beego` 和 `RUN go get github.com/beego/bee` 来安装 Beego 和 Bee 的依赖.

那么我们怎么使用这个 Dockerfile 呢?

-  第一种方式是你自己 build, 在这个 Dockerfile 所在目录下运行下面的命令 

  `docker build -t leicao/beego .`

非常的简单, 接着你就可以在 `docker images` 里面看到它了

-  第二种方式更好更方便, 那便是使用 Docker 官方的 [自动化 build](https://docs.docker.com/docker-hub/builds/#the-dockerfile-and-automated-builds)

根据这个教程, 为你的 Dockerfile 创建 git repo, 我的是 [github.com/lei-cao/dockers](https://github.com/lei-cao/dockers).  之后我就得到了自动 build 的 [beego image](https://registry.hub.docker.com/u/leicao/beego/).

有了 Beego 的 Docker image, 我们就可以和它开心的玩耍了.

我们可以使用 `bee` 工具来创建 beego 应用:

`docker run --rm -v "$(pwd)":/go/src/ -w /go/src leicao/beego bee new beego-docker`


`--rm`  告诉 Docker 自动删除这个 Container 当执行结束后.  `-v "$(pwd)":/go/src/`  会挂载我的当前目录到 Docker Container 里面的 `/go/src` 目录. `-w /go/src` 指定运行命令的目录. `leicao/beego`  是这里所使用的 Docker image, 也就是我们刚刚创建的那个 Beego Image. 最后通过 `bee new beego-docker` 创建了一个 beego 应用 `beego-docker`.

这里最开心的部分就是目录的挂载, 使得我们可以用 Docker Container 来在主机上创建和修改文件, 完成开发, 而不用配置开发环境以及担心开发环境的依赖. 程序员的人生本来就该如此美好.  

接着我们就可以运行我们的 `beego-docker` 项目了

`docker run --rm -v "$(pwd)"/beego-docker:/go/src/beego-docker -w /go/src/beego-docker -p 8081:8080 leicao/beego bee run`

这里使用了一个新的选项 `-p 8081:8080`, 它将 Docker container 内部的 8080 端口映射到主机的 8081 端口. 这时, 我们的 beego 应用已经可以访问了: [http://book.beego.me:8081](http://book.beego.me:8081)

![](../images/docker.bee.run.png?raw=true)


5. <a name="mysql-docker-image"></a>使用 [Mysql Docker image](https://registry.hub.docker.com/_/mysql).
------------------

和我们刚才玩的 Golang Docker Image 类似, 我们可以通过一个命令运行起一个 Mysql server.

`docker run --name db -e MYSQL_ROOT_PASSWORD=root -d mysql`

这个命令会从官方的 mysql Docker image 上运行一个名字叫做 `db` 的 container. 并通过选项 `-e MYSQL_ROOT_PASSWORD=root` 将 root 的密码设置成了 `root`.

要连接到这个 Mysql server:

`docker run -it --link db:mysql mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p'`

 这里又用到了一个新的选项, `--link db:mysql`, 它通过 container 的 name `db` 在两个 container 直接创建了连接, 从而可以使用另外一个 container 的资源. 我们可以在 Mysql 命令行创建我们的数据库 `create database beego;`

![](../images/docker.mysql.png?raw=true)

> [Docker 关于 links 的文档](http://docs.docker.com/userguide/dockerlinks/) 查看更多关于 links 的细节

6. <a name="nginx-docker-image"></a>使用 [Nginx Docker image](https://registry.hub.docker.com/_/nginx).
------------------

And one more thing, Nginx.

先运行我们的 beego 应用:

`docker run --rm --name beego -v "$(pwd)"/beego-docker:/go/src/beego-docker -w /go/src/beego-docker -p 8080:8080 leicao/beego bee run`

下面是 Nginx 的配置文件:

```
server {
  listen              80;
  server_name         book.beego.me;

  client_max_body_size  20m;
  client_header_timeout 1200;
  client_body_timeout   1200;
  send_timeout          1200;
  keepalive_timeout     1200;

  location / {
        proxy_pass http://beego:8080;
   }
}

```

然后启动 Nginx:

`docker run --name nginx --link beego:beego -v "$(pwd)"/nginx:/etc/nginx/conf.d/ -p 80:80 -d nginx`

 你可能会疑惑为什么用 `proxy_pass http://beego:8080;`. 事实上当我们使用 `--link beego:beego` 的时候, Docker 会在 Nginx 的 container 里面的 `/etc/hosts` 文件中设置一个 host, 将 Beego container 的 ip 指向 它的名字 `beego`.

![](../images/docker.link.hosts.png?raw=true)

7. <a name="wire-up"></a>联合使用这些 Docker
-----------------

我们现在有了 Golang, Beego/Bee, Mysql, Nginx. 接下来让我们 参照这个例子 [bee api applicatioin](http://beego.me/blog/beego_api) [Youtube](http://youtu.be/w7RziV_Sn-g), 创建一个 Beego Api 应用

我们来创建一个 beeblog api 应用. [这里是数据库的结构](medias/beeblog.sql)

- 启动一个 Mysql 数据库服务, 并初始化数据库 beeblog:

 `docker run --name db -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=beeblog -d mysql` 
 

- 导入数据库结构:

 `docker run --rm --link db:mysql -v "$(pwd)"/mysql:/home/mysql -it mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p beeblog < /home/mysql/beeblog.sql'`


- 生成 Beego Api 应用:

 `docker run --rm --link db:mysql -v "$(pwd)":/go/src/ -w /go/src leicao/beego bee api beeblog -conn="root:root@tcp(mysql:3306)/beeblog"`


- 运行 Api 应用:

 `docker run --rm --link db:mysql -v "$(pwd)/beeblog":/go/src/beeblog -w /go/src/beeblog -p 8080:8080 --name beego leicao/beego bee run -downdoc=true -gendoc=true`

   或者编译运行:
 `sudo docker run  --link db:mysql -v "$(pwd)/beeblog":/go/src/beeblog -w /go/src/beeblog -p 8080:8080 --name beego -d leicao/beego ./beeblog`


- 启动 Nginx:

 `docker run --name nginx --link beego:beego -v "$(pwd)"/nginx:/etc/nginx/conf.d/ -p 80:80 -d nginx`

在这里便可以访问我们创建的 Api 应用了:

API: [http://book.beego.me/v1/posts](http://book.beego.me/v1/posts)

API 文档: [http://book.beego.me/swagger/swagger-1/](http://book.beego.me/swagger/swagger-1/)

![](../images/docker.swagger.png?raw=true)

8. [总结](#conclusion)
-----------

前面我们介绍了 Docker, Docker image, Docker container 的基本概念以及基本使用方法. 我们不再需要花费大量的时间去配置开发环境了.  一开始你可能会对这些长长的 Docker 命令感到陌生, 但是当你熟悉他们只会, 你就会发现其实他们是很容易使用的.

为了避免过多手动的进行 Docker 的操作, 你可以使用其他一些 Docker 工具来简化操作. [Fig](http://www.fig.sh)  便是其中之一. 后面我还会介绍 Fig 的使用.






