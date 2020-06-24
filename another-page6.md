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
$ docker start     //start a container
$ docker restart   //restart
$ docker exec      //enter the background container
$ docker export    //export
$ docker import    //import
$ docker rm        //delete comntainer
$ docker inspect   //View the underlying information of Docker
$ docker images    //list all the images
```
Default website is [Docker Hub][jekyll-docs2], which is the main way to download image.
## Build image
 First we need to build a Dockerfile. The example format is shown below:
 ![图片pic1]({{ "/assets/images/dockerfile.png" | absolute_url }})
The 'FROM' in the first line specify which image source to use.  
Each instruction creates a new step on the image, and the prefix of each instruction must be capitalized.  
Then input '$ docker build' to build.

## Connection between Docker and Container

## Thoughts 

[jekyll-docs2]: https://hub.docker.com/

[back](./)