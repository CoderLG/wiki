```
进入docker
docker exec -it 9a8b3a04a2ac /bin/bash

查看配置文件
docker ps
docker inspect  9a8b3a04a2ac

映射docker中的端口
docker service ls
docker service update --publish-add 7610:7610 fefa99274c31

删掉映射端口
docker service update --publish-rm 7610:7610 icenter_eurekaserver

查看环境变量
docker exec 2992c1591a68 env

```



**重新打镜像 加入jdk**

```
查看镜像
docker images | grep centos

vi Dockerfile
	from 192.168.4.38:5000/centos-jre-gdal:0.1
    ADD jdk1.8.0_221 /usr/local/jdk1.8.0_221
    ENV JAVA_HOME=/usr/local/jdk1.8.0_211
    ENV JRE_HOME=${JAVA_HOME}/jre
    ENV CLASSPATH=/usr/local/jdk1.8.0_211/lib/rt.jar:/usr/local/jdk1.8.0_211/lib/ext
    ENV PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:${PATH}
    WORKDIR /opt
    
vi build.sh
  	docker build -t centos-jre-gdal:0.1 .
  
./build.sh
  
注意id号
docker images | grep centos


打标签
docker tag centos-jre-gdal:0.1 192.168.4.9:5000/centos-jre-gdal:0.1

注意id号
docker images | grep centos

推给其他
docker push 192.168.4.9:5000/centos-jre-gdal:0.1


------------------
通过tar包上传
	docker save -o centos.tar.gz centos-jre-gdal:0.1
	docke load -i centos.tar.gz
```





