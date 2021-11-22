##                  Rancher 环境搭建







 ~~~ properties
 最好安装为2.x的版本
 镜像地址
 https://hub.docker.com/r/rancher/rancher
 ~~~





### 1.下载docker环境 





​             docker pull rancher/rancher:v2.4.6               





### 2.去dockerhub拉取镜像







​     docker pull rancher/rancher:v2.4.6

​     

​      

### 3.运行镜像 

 

docker run -d  -p 80:80 -p 443:443 rancher/rancher:v2.4.6



### 4.访问

​         http://ip

​        Https://ip

 



