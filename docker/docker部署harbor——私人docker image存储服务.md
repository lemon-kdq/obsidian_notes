
安装harbor：
https://blog.csdn.net/m0_37908257/article/details/136568766
note：不用https，这个不好弄

如何将本地image推到harbor：

1. 需要修改/etc/docker/daemon.json 增加
  "insecure-registries" : ["127.0.0.1:5100"]  对应你harbor的ip:port
   
   2.自己到harbor网页创建image 仓，并根据提示给本地仓打上tag然后用docker push指令即可推上去
   

