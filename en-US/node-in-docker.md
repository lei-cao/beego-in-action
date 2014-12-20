Node in Docker
================

[**简体中文版**](../zh-CN/node-in-docker.md)

We put the Beego development environment into the Docker [here](beego-in-docker.md).
In this section let's keep working with Docker and Node.
You might be thinking we are talking about Beego, why do we need to work with node?
Well, in fact the reason is while you are developing the web application,
you need many useful tools to work together and many of them rely on Node.


Here is our [`Dockerfile`](https://github.com/lei-cao/dockers/blob/master/node.js/Dockerfile) for our node tools:
Based on the [official node.js image](https://registry.hub.docker.com/_/node/) it installs the basic global libraries we need,
 you can add any other libraries you need for your application.

```
FROM node:latest

RUN npm install -g express
RUN npm install -g gulp
RUN npm install -g grunt
RUN npm install -g yo
RUN npm install -g bower
RUN npm install -g nodemon
```

You can start the Node project with npm from scratch:

`sudo docker run -it --rm -v "$(pwd)":/tmp/myapp -w /tmp/myapp leicao/node npm init`

`sudo docker run --rm -it -v "$(pwd)":/tmp/myapp -w /tmp/myapp leicao/node npm install`

`sudo docker run --rm -it -v "$(pwd)":/tmp/myapp -w /tmp/myapp leicao/node bower --allow-root install`
