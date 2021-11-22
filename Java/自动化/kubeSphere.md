##                               kubeSphere环境搭建

##  

### 1.下载源码（安装脚本）

​     

​           curl -L  https://kubesphere.io/download/stable/v2.1.1 > install.tar.gz



### 2.解压并进入config目录，修改host.ini文件

​           

![image-20210315133856053](/Users/zhang/Library/Application Support/typora-user-images/image-20210315133856053.png) 



### 3.修改common.yaml配置文件

​      配置阿里云加速镜像（推荐阿里云）  

   **https://gij5bfoo.mirror.aliyuncs.com** 

![image-20210315134511764](/Users/zhang/Library/Application Support/typora-user-images/image-20210315134511764.png)





    **如果想要相应的功能，可以直接修改相应的配置文件，个性下图的false为true**

![image-20210315134803557](/Users/zhang/Library/Application Support/typora-user-images/image-20210315134803557.png)







### 4. 进入scripts目录下，运行install.sh脚本

 





