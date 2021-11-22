##                       rancher2.搭建





### 1.准备docker环境



### 2.去docker hub拉取镜像



​            **docker pull rancher/rancher:v2.4.6**



### 3.启动容器

   

 docker run -d –restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:v2.4.8（参数错误，不能使用-restart）



docker run -d    -p 80:80 -p 443:443 rancher/rancher:v2.4.6



```
export http_proxy=http://127.0.0.1:1087 export ALL_PROXY=$http_proxy
```

###  