---
layout: default
---
<div style='display: none'>
Docker资源
Docker官方英文资源：

docker官网：http://www.docker.com
Docker windows入门：https://docs.docker.com/windows/
Docker Linux 入门：https://docs.docker.com/linux/
Docker mac 入门：https://docs.docker.com/mac/
Docker 用户指引：https://docs.docker.com/engine/userguide/
Docker 官方博客：http://blog.docker.com/
Docker Hub: https://hub.docker.com/
Docker开源： https://www.docker.com/open-source
Docker中文资源：

Docker中文网站：http://www.docker.org.cn
Docker入门教程: http://www.docker.org.cn/book/docker.html
Docker安装手册：http://www.docker.org.cn/book/install.html
一小时Docker教程 ：https://blog.csphere.cn/archives/22
Docker纸质书：http://www.docker.org.cn/dockershuji.html
DockerPPT：http://www.docker.org.cn/dockerppt.html
</div>

## Overview 
Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.
## Docker architecture
Docker includes three basic concepts:  
Image: Docker images are templates for creating Docker containers  
Container: A container is an application or a group of applications that runs independently and is an entity that mirrors the runtime.  
Repository: Repository is used to save images, which can be understood as a code warehouse in code control  
## Operating Docker 
To creates a new container with ubuntu15.10 image:
```$xslt
$ docker run -i -t ubuntu:15.10 /bin/bashDocker 
//-t:Specify a pseudo terminal or terminal in the new container
//-i:Allows you to interact with the standard input (STDIN) inside the container
```
If you add -d behind docker run, will allow the background process.
By running the 'exit' command to exit the container.
```$xslt
$ docker stop      //stop docker
$ docker ps        //check the status
$ docker           //See all command options
$ docker pull      //pull image
$ docker push      //push image to your accout
$ docker start     //start a container
$ docker restart   //restart
$ docker exec      //enter the background container
$ docker export    //export
$ docker import    //import
$ docker rm        //delete comntainer
$ docker inspect   //View the underlying information of Docker
$ docker images    //list all the images
```
More instructions can be found in [Docker instructions][jekyll-docs3]  
Default website is [Docker Hub][jekyll-docs2], which is the main way to download image.


## Connection between Docker and Container
Input: 'docker run -d -P training/webapp python app.py' to create a container. We use the -P parameter to create a container, and use docker ps to see that container port 5000 is bound to host port 32768, also, we can use -p to Specify the network address bound to the container, like 'docker run -d -p 5000:5000 training/webapp python app.py'  
The 'docker port' command allows us to quickly check the port binding status.  
Port mapping is not the only way to connect docker to another container.  
Docker has a connection system that allows multiple containers to be connected together and share connection information.  
The docker connection creates a parent-child relationship, where the parent container can see the information of the child container.  
```xslt
docker network create -d bridge test-net  
//first create a new Docker network
docker run -itd --name test1 --network test-net ubuntu /bin/bash 
//Run a container and connect to the newly created test-net network

```
## Dockerfile
The Docker file is a file which used to build images, which goes like this:
```xslt
FROM nginx
RUN echo 'this is a local nginx image' > /usr/share/nginx/html/index.html
```
Then input:'docker build -t nginx:test .' to build.  
The dockerfile instructions:
```xslt
//Copy the file or directory from the context directory to the specified path in the container
COPY [--chown=<user>:<group>] <source>...  <destination>
COPY [--chown=<user>:<group>] ["<source>",...  "<destination>"]
//CMD used to run the program
CMD ["<file>","<param1>","<param2>",...]
//ENTRYPOINT,similar with CMD instruction
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
//ENV Set environment variables
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
//VOLUME Define anonymous data volume, data lost
VOLUME ["<path1>", "<path2>"...]
VOLUME <path>
//EXPOSE Declared port
EXPOSE <port1> [<port2>...]
//WORKDIR to Specify working directory
WORKDIR <path>
//Specify the user and user group to execute subsequent commands
USER <User>[:<group>]
//Specify a program or instruction to monitor the running status of the docker container service
HEALTHCHECK [option] CMD <instruction>：setting order which can monitor the status
HEALTHCHECK NONE：Block out its health check instructions
HEALTHCHECK [option] CMD <instruction> : CMD following instructions
//Delay the execution of build commands
ONBUILD <other instructions>
```
CMD is usually used for changing parameters

## Build image
The example format is shown below:
 ![图片pic1]({{ "/assets/images/dockerfile.png" | absolute_url }})
The 'FROM' in the first line specify which image source to use.  
Each instruction creates a new step on the image, and the prefix of each instruction must be capitalized.  
Then input 'docker build' to build.

## Thoughts 
By learning Docker, I realized that docker can solve the problem that the program can only be run in your computer because of surroundings. For example, if you want to run a program which made in other circumstances, in order to make it work, you need to download Java, Tomcat,Mysql in your Linux system to build the operating environment which is too complicated. By docker, the only thing that you need to do is pull an image from the website source.  
So by using docker, A whole set of environment can be packaged and packaged as a mirror image, without repeated configuration of the environment. And Process is isolated between Docker containers, no one will affect anyone.  
What's more, Docker is much lighter than virtual machine.




[jekyll-docs2]: https://hub.docker.com/
[jekyll-docs3]: https://www.runoob.com/docker/docker-command-manual.html

[back](./)