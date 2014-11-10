Dockerized Beego
================

[**简体中文版**](../zh-CN/beego-in-docker.md)

[Docker](http://docker.io) is a amazing tool to create the consistent environment for development and deployment. We developers have already gotten used to spend hours to set up the development environment to just start writing code. We've spent so many times in the dependency hell. Our development easily get crashed while we upgrade our system. We encounter wired bugs because the development environment is different from the production server environment. Thankfully we have Docker now. It's the must-try tool to make your development easier and happier.

There are already many articles introduce you what is Docker, how to install Docker and get started with it. I will directly talk about how to set up the development environment for developing [beego](http://beego.me) applications. If you are new to Docker, it's better to check out the links below first.


- [Docker homepage](https://www.docker.com/)
- [Docker: Lightweight Linux Containers for Consistent Development and Deployment](http://www.linuxjournal.com/content/docker-lightweight-linux-containers-consistent-development-and-deployment)
- [Docker: Using Linux Containers to Support Portable Application Deployment](http://www.infoq.com/articles/docker-containers)
- [Deploying Go servers with Docker](https://blog.golang.org/docker)

After you getting the basic concept of Docker, let's start to dockerize our production ready Beego application. Here is our [final application](http://book.beego.me/), nothing much, just the beego application default page with reading the data from the mysql database dynamically. Here is our outlines:

1. [Set up the server](#setup)

2. [Install the Docker on the server.](#install)

3. [Get along with the Golang Docker image](#golang-docker-image)

 * [The ready-to-use Golang env](#golang-env)

 * [The hello world](#hello-world)

4. [Build the beego Docker image](#beego-docker-image)

5. [Get along with the Mysql Docker image](#mysql-docker-image)

6. [Get along with the Nginx Docker image](#nginx-docker-image)

7. [Wire dockers up together](#wire-up)

8. [Conclusion](#conclusion)  



1. <a name="setup"></a>Set up the server
-------------

I made this demo on the live server. If you guys want to play on your local machine, you can skip this section. It's perfectly ok, no much difference.

I bought the virtual host from [Linode](https://www.linode.com/) and install the Ubuntu 14.10. I pointed the subdomain `book.beego.me` to this Ubuntu server. This step is very flexible, Nothing much.


2. <a name="install"></a>Install the Docker on the server.
-------------

First of all you need to install the Docker. For different systems it might be different, you can just follow this guide   [installation](https://docs.docker.com/installation/#installation) to get Docker installed on your machine no matter what system you are using. 

Specifically for my Ubuntu server:

```bash
$ sudo apt-get update
$ sudo apt-get install docker.io
$ source /etc/bash_completion.d/docker.io
```

3. <a name="golang-docker-image"></a>Get along with the [Golang Docker image](https://registry.hub.docker.com/_/golang/).
-------------

Docker has made a [Golang official repo](https://registry.hub.docker.com/_/golang/), you can start using it as your base Go environment. Let's create a hello world web application with our Golang Docker image.

 * ### <a name="golang-env"></a>The ready-to-use Golang env

Just run this command to start a new Docker Container from the [Golang official repo](https://registry.hub.docker.com/_/golang/)

`docker run -it golang /bin/bash`

After the container running, you got the Golang environment:

`go version`

You should see something like below. It created a new container from the Golang image and ran command `/bin/bash` with options `-it` on that container which will give you a terminal to interactive with your container. Then I typed `go version` to check my Golang environment.

![](../images/docker.golang.png?raw=true)

The first time when you run `docker run <image name>`, if docker can't find it on your local machine, it will automatically try to pull the Docker image from the Docker repository.

 You can check your local Docker images by command:

`docker images`

You can check the containers you created from the images by command:

`docker ps -a`


 * ### <a name="hello-world"></a>The hello world

`docker run -it -v /home/leicao/programming/go/hello-world/:/go/src/hello-world golang /bin/bash`

After the container running, you can build your Go code with the container's Go environment:

`go build hello.go`
`./hello`

Below is the result you should see:

![](../images/docker.golang.hello-world.png?raw=true)

Now you might get confused how had this been done, let me explain it detailed:

In the picture above:

`/home/leicao/programming/go/hello-world/hello.go` is the hello world file on my host machine which is my Ubuntu running on Linode, it just contains the simple Go hello world code:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

Then we run our Docker container with command:

`docker run -it -v /home/leicao/programming/go/hello-world/:/go/src/hello-world golang /bin/bash`

We added option `-v /home/leicao/programming/go/hello-world/:/go/src/hello-world`, which will mount the folder `/home/leicao/programming/go/hello-world/` onto my Docker container as the volume `/go/src/hello-world`.

After this, inside my Docker container, I got the mounted folder `/go/src/hello-world`. Then I can `cd src/hello-world/` to it and `go build hello.go` and finally `./hello`!!!

Isn't it amazing? With Docker we don't need to try hard to deal with the go environment anymore. Everything is ready to use.


4. <a name="beego-docker-image"></a>Build the [beego Docker image](https://registry.hub.docker.com/u/leicao/beego/).
-------------------

Now we have a Golang environment, we can extend it with Beego support. I made a Beego Docker image from the official Golang image, you can get it [here](https://registry.hub.docker.com/u/leicao/beego/) and use it directly. In fact, it's very easy to make your own Docker images. It can be done in 5 minutes.

We start this with a `Dockerfile`:

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

It says we started this image from the official Golang base image `FROM golang:1.3.3`, and then we just go get our dependencies `RUN go get github.com/astaxie/beego` and `RUN go get github.com/beego/bee`. It's quite intuitive.

How to use this Dockerfile?

- The first option is build it by your own, run this command inside the folder with your Dockerfile

  `docker build -t leicao/beego .`

 It's simple, right? If it's successfully built, you can see it by `docker images`
 

- The second option is even better which is the Docker's [automated builds](https://docs.docker.com/docker-hub/builds/#the-dockerfile-and-automated-builds)

 It's also very simple, just following the guide, creating your git repo with the Dockerfile, in my case it's [github.com/lei-cao/dockers](https://github.com/lei-cao/dockers). And then I got this [build](https://registry.hub.docker.com/u/leicao/beego/) built by Docker automated builds.

Now we have the Beego Docker image, let's play with it.

We can create a brand new beego application with the `bee` tool:

`docker run --rm -v "$(pwd)":/go/src/ -w /go/src leicao/beego bee new beego-docker`

`--rm` tells Docker remove this container immediately after executing.  `-v "$(pwd)":/go/src/` will mount my current folder on my Ubuntu host to the Docker container `/go/src/. `-w /go/src` tells Docker the working dir. `leicao/beego` is the Docker image we are using to crate the Docker container. And at last, our bee command `bee new beego-docker` to create our beego project `beego-docker`.

The amazing part here is since we are using mounted volume, so I created the beego project on my host server which is my Ubuntu server by using the Docker image `leicao/beego` we created before. We can working with Golang and Beego without setting up the development environment at all. All we need is our Dockers.

Then let's run our `beego-docker` project:

`docker run --rm -v "$(pwd)"/beego-docker:/go/src/beego-docker -w /go/src/beego-docker -p 8081:8080 leicao/beego bee run`

The new option I am using here is `-p 8081:8080` which will expose the port 8080 inside the Docker container to the host's 8081 port. And guess what, see it on live!! [http://book.beego.me:8081](http://book.beego.me:8081).

![](../images/docker.bee.run.png?raw=true)


5. <a name="mysql-docker-image"></a>Get along with the [Mysql Docker image](https://registry.hub.docker.com/_/mysql).
------------------

Similar as the Golang Docker image we play with, we can get the Mysql running in a single command:

`docker run --name db -e MYSQL_ROOT_PASSWORD=root -d mysql`

This command will pull the official mysql Docker image from the repo if needed and start a mysql service container which is named `db`. I set this mysql server's root password to `root` by the option `-e MYSQL_ROOT_PASSWORD=root`.

To connect to this server from command line:

`docker run -it --link db:mysql mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p'`

This command will connect to the mysql server we ran before by using `--link db:mysql` which links by the container name `db`. And we can create our database here `create database beego;`

![](../images/docker.mysql.png?raw=true)

> You can check the [Docker document about links](http://docs.docker.com/userguide/dockerlinks/) for more information. 

6. <a name="nginx-docker-image"></a>Get along with the [Nginx Docker image](https://registry.hub.docker.com/_/nginx).
------------------

And one more thing, Nginx.

With our Beego application running by:

`docker run --rm --name beego -v "$(pwd)"/beego-docker:/go/src/beego-docker -w /go/src/beego-docker -p 8080:8080 leicao/beego bee run`

And with this nginx proxy config:

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

We can start our nginx by:

`docker run --name nginx --link beego:beego -v "$(pwd)"/nginx:/etc/nginx/conf.d/ -p 80:80 -d nginx`

You might be wondering why it's `proxy_pass http://beego:8080;` here. It happens magically by using `--link beego:beego`, Docker will set a host which pointing container beego's ip to the container's name, in the nginx's `/etc/hosts` file,

![](../images/docker.link.hosts.png?raw=true)

7. <a name="wire-up"></a>Wire dockers up together
-----------------

We are having Golang, Beego/Bee, Mysql, Nginx now. Let's start a new api project similar as the tutorial here [bee api application](http://beego.me/blog/beego_api) Video: [Youtube](http://youtu.be/w7RziV_Sn-g).

We will create a beeblog api and [Here is our database schema](medias/beeblog.sql)

- Start a Mysql server with db name beeblog:

 `docker run --name db -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=beeblog -d mysql` 
 

- Import our database schema:

 `docker run --rm --link db:mysql -v "$(pwd)"/mysql:/home/mysql -it mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p beeblog < /home/mysql/beeblog.sql'`


- Generate the api application:

 `docker run --rm --link db:mysql -v "$(pwd)":/go/src/ -w /go/src leicao/beego bee api beeblog -conn="root:root@tcp(mysql:3306)/beeblog"`


- Run our api application:

 `docker run --rm --link db:mysql -v "$(pwd)/beeblog":/go/src/beeblog -w /go/src/beeblog -p 8080:8080 --name beego leicao/beego bee run -downdoc=true -gendoc=true`

  Or run it directly
 `sudo docker run  --link db:mysql -v "$(pwd)/beeblog":/go/src/beeblog -w /go/src/beeblog -p 8080:8080 --name beego -d leicao/beego ./beeblog`

- Start the Nginx:

 `docker run --name nginx --link beego:beego -v "$(pwd)"/nginx:/etc/nginx/conf.d/ -p 80:80 -d nginx`

Now you can see our beeblog api on live:

The API: [http://book.beego.me/v1/posts](http://book.beego.me/v1/posts)

And the document: [http://book.beego.me/swagger/swagger-1/](http://book.beego.me/swagger/swagger-1/)

![](../images/docker.swagger.png?raw=true)

8. [Conclusion](#conclusion)
-----------

This section gives you a basic version of Docker, Docker images, Docker containers and how to work with them. We don't need to spend so much time to set up the development environment and suffer the dependency hell anymore. At the beginning you might be afraid of all those long docker commands you need to type, just go check the Docker document and you will get familiar with them soon.

Also there are some convenient tools to help you out from these commands. [Fig](http://www.fig.sh) is one of them, it gives you fast and isolated development environments using Docker. I will talk about Fig in the future.





