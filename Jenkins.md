#   Jenkins



# 一、安装和配置步骤



## 1、安装docker和docker compose



https://docs.docker.com/engine/install/ubuntu/



现在的官网会推荐直接安装compose。

```bash
root@jenkins:/home/azureuser# docker compose version
Docker Compose version v2.6.0
root@jenkins:/home/azureuser# docker version
Client: Docker Engine - Community
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:02:57 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.17
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.11
  Git commit:       a89b842
  Built:            Mon Jun  6 23:01:03 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.7
  GitCommit:        0197261a30bf81f1ee8e6a4dd2dea0ef95d67ccb
 runc:
  Version:          1.1.3
  GitCommit:        v1.1.3-0-g6724737
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```





安装docker compose：

```bash
# 下载的是可执行文件，修改权限：
root@jenkins:/home/azureuser# chmod +x docker-compose-linux-x86_64 
# 图标变绿
root@jenkins:/home/azureuser# ls
docker-compose-linux-x86_64  jdk8  maven

root@jenkins:/home/azureuser# mv docker-compose-linux-x86_64 docker-compose
root@jenkins:/home/azureuser# ls
docker-compose  jdk8  maven

root@jenkins:/home/azureuser# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

root@jenkins:/home/azureuser# mv docker-compose /usr/bin
root@jenkins:/home/azureuser# ls
jdk8  maven

# 成功
root@jenkins:/home/azureuser# docker-compose version
Docker Compose version v2.9.0
```













## 2、安装JDK1.8和Maven



```bash
wget https://abcdefe.blob.core.windows.net/newcontainer/jdk-8u231-linux-x64.tar.gz

wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz

tar -zxvf jdk-8u231-linux-x64.tar.gz -C /usr/local

tar -zxvf apache-maven-3.8.6-bin.tar.gz -C /usr/local

cd /usr/local
```







```bash
root@jenkins:/usr/local# ls
apache-maven-3.8.6  bin  etc  games  include  jdk1.8.0_231  lib  man  sbin  share  src
root@jenkins:/usr/local# mv jdk1.8.0_231/ jdk/
root@jenkins:/usr/local# mv apache-maven-3.8.6/ maven
root@jenkins:/usr/local# ls
bin  etc  games  include  jdk  lib  man  maven  sbin  share  src
```





```bash
root@jenkins:/usr/local# cd maven/
root@jenkins:/usr/local/maven# cd conf/
root@jenkins:/usr/local/maven/conf# ls
logging  settings.xml  toolchains.xml
```



修改settings.xml。有2处要改。

```bash
root@jenkins:/usr/local/maven/conf# vim settings.xml                                    
root@jenkins:/usr/local/maven/conf#  [dos] 263L, 10660C written

```



（1）配置镜像：

```xml

    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>*</mirrorOf>  
      <name>nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>     
    </mirror>

```



（2）maven配置settings.xml指定默认java8版本

https://blog.csdn.net/abcnull/article/details/110914564



```xml
<profile>
    <id>jdk8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>

```



这个默认开启：

```xml
<activeProfiles>
    <activeProfile>jdk8</activeProfile>
</activeProfiles>
```





## 3、安装Jenkins



https://www.jenkins.io/download/



（1）获取镜像并设置compose yaml

最新LTS版本：**Jenkins 2.346.2 LTS**

**2025年: Jenkins 2.516.1 LTS**

如果用之前版本的jenkins，会导致很多plugins无法安装。



找到默认的lts版本的image：

https://hub.docker.com/r/jenkins/jenkins/tags?page=3

![image-20220810004753402](Jenkins.assets/image-20220810004753402.png)

拉取镜像：

```bash
root@jenkins:/home/azureuser# docker pull jenkins/jenkins:2.516.1-lts
2.346.2-lts: Pulling from jenkins/jenkins
1339eaac5b67: Pull complete
20401c7e91bc: Pull complete
7138cd942003: Pull complete
6d1b42f45e89: Pull complete
98b0e135a912: Pull complete
ed90436583b0: Pull complete
b0b3716848f8: Pull complete
4035b7550508: Pull complete
e9a1c1f127f6: Pull complete
6137d1289fb5: Pull complete
213d8e7e603c: Pull complete
42b46c55d38d: Pull complete
8324f1380818: Pull complete
2201f3ff6253: Pull complete
Digest: sha256:c878e1aac1f5152a6234b33a10542c7f694b7c5c37de27191d1c173800853b93
Status: Downloaded newer image for jenkins/jenkins:2.346.2-lts
docker.io/jenkins/jenkins:2.346.2-lts
```





```bash
root@jenkins:/usr/local# mkdir docker
root@jenkins:/usr/local# ls
bin  docker  etc  games  include  jdk  lib  man  maven  sbin  share  src
```



docker compose yaml文件：

```yaml
version: "3.1"
services:
  jenkins:
    image: jenkins/jenkins:2.516.1-lts
    container_name: jenkins
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - ./data/:/var/jenkins_home/
```



（2）启动容器

启动容器：

```bash
root@jenkins:/usr/local/docker/jenkins_docker# docker-compose up -d
[+] Running 2/2
 ⠿ Network jenkins_docker_default  Created                                                                                      0.1s 
 ⠿ Container jenkins               Started
```





发现权限不足：

```bash
root@jenkins:/usr/local/docker/jenkins_docker# docker logs -f jenkins
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```



给data权限：

```bash
root@jenkins:/usr/local/docker/jenkins_docker# chmod -R 777 data
root@jenkins:/usr/local/docker/jenkins_docker# ls
data  docker-compose.yaml
```



restart: 

```bash
root@jenkins:/usr/local/docker/jenkins_docker# docker-compose restart
[+] Running 1/1
 ⠿ Container jenkins  Started 
```



重启之后，再看log，发现错误日志已经没有了：

```bash
root@jenkins:/usr/local/docker/jenkins_docker# docker logs -f jenkins

# 给出了admin密码

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

8d708db7ef8a448badbfc438ca82e0fe
```





（3）登录

稍等大概1分钟：

用上面的password登录：

![image-20220810021552048](Jenkins.assets/image-20220810021552048.png)





安装plugins：选 Select plugins to install

![image-20220810021638145](Jenkins.assets/image-20220810021638145.png)





安装插件：插件可能安装失败，失败的话可以之后再手动安装。我这里全部成功了。

![image-20220810021832210](Jenkins.assets/image-20220810021832210.png)





全部选择root：

![image-20220810022747606](Jenkins.assets/image-20220810022747606.png)



选默认url



成功： 

![image-20220810022852923](Jenkins.assets/image-20220810022852923.png)

## 4、安装plugin

确认几个plugin，然后点击Install without restart

1、Git Parameter

2、Publish Over SSH

![image-20220814231739006](Jenkins.assets/image-20220814231739006.png)



Install成功：

![image-20220814231824388](Jenkins.assets/image-20220814231824388.png)



## 5、配置Jenkins

（1）

```bash
# 把jdk和maven复制到data目录下
root@jenkins:/usr/local/docker/jenkins_docker/data# cp -r /usr/local/jdk ./
root@jenkins:/usr/local/docker/jenkins_docker/data# cp -r /usr/local/maven ./
root@jenkins:/usr/local/docker/jenkins_docker/data#
```



（2）指定 Jenkins 的 jdk 目录

![image-20220814233013269](Jenkins.assets/image-20220814233013269.png)

![image-20220814233031667](Jenkins.assets/image-20220814233031667.png)

最新版要用jdk24

/var/jenkins_home/jdk-24.0.2



![image-20220814233252932](Jenkins.assets/image-20220814233252932.png)



（3）指定 Jenkins 的 maven 目录

和前面jdk一样：

![image-20220814233506792](Jenkins.assets/image-20220814233506792.png)



然后点击save





（4）准备publish



![image-20220814233817432](Jenkins.assets/image-20220814233817432.png)





滑到最下面，可以找到插件：**Publish over SSH**



![image-20220814235240237](Jenkins.assets/image-20220814235240237.png)





> 注意：因为我设置了2222为SSH端口，所以后面端口要选择2222
>
> 另外，test文件夹需要提前创建，不然会显示连接失败。



![image-20220814235329505](Jenkins.assets/image-20220814235329505.png)





测试连接成功：

![image-20220814235353007](Jenkins.assets/image-20220814235353007.png)





Apply then Save：

![image-20220814235445188](Jenkins.assets/image-20220814235445188.png)













# 二、Jenkins pull project



## 1、create new item



![image-20220815001146082](Jenkins.assets/image-20220815001146082.png)









## 2、选择Freestyle

![image-20220815001329725](Jenkins.assets/image-20220815001329725.png)





## 3、找到Source Code Management

填入Respository URL。因为我是Github仓库，所以需要用户名和密码

![image-20220815001549287](Jenkins.assets/image-20220815001549287.png)



填入我的Github账号和密码：



![image-20220815001908232](Jenkins.assets/image-20220815001908232.png)





最后Apply and Save





## 4、开始build



![image-20220815002053861](Jenkins.assets/image-20220815002053861.png)





查看日志：

![image-20220815002131930](Jenkins.assets/image-20220815002131930.png)



发现build成功：

![image-20220815002236232](Jenkins.assets/image-20220815002236232.png)





## 5、查看project位置



```bash
root@jenkins:/home/azureuser# docker exec -it jenkins bash
jenkins@86216a6c0943:/$ cd ~
jenkins@86216a6c0943:~$ pwd
/var/jenkins_home
# ~ 就是/var/jenkins_home目录

jenkins@86216a6c0943:~$ ls
# 这个目录下有之前通过volume同步过来的jdk和maven。
# 还有一个目录叫：workspace。所有的工程都会放到workspace里
```



![image-20220815002818955](Jenkins.assets/image-20220815002818955.png)





```bash
jenkins@86216a6c0943:~$ cd workspace/
jenkins@86216a6c0943:~/workspace$ ls
jenkinstest  jenkinstest@tmp
jenkins@86216a6c0943:~/workspace$ cd jenkinstest
jenkins@86216a6c0943:~/workspace/jenkinstest$ ls
pom.xml  src
```











# 三、Jenkins构建jar



## 1、进入Configure



![image-20220815003302733](Jenkins.assets/image-20220815003302733.png)





![image-20220815003430218](Jenkins.assets/image-20220815003430218.png)







选择之前的maven，然后在Goal里加入一个全局命令clean package -DskipTests

![image-20220815003644684](Jenkins.assets/image-20220815003644684.png)





## 2、build again

这次build耗时比较久，因为第一次使用maven的时候，需要初始化大量本地仓库

![image-20220815003744870](Jenkins.assets/image-20220815003744870.png)



查看Output：

除了有我们之前设置的全局命令，后面就是下载相关镜像。（从maven.aliyun.com，见前面第一章里的安装maven）

![image-20220815003958754](Jenkins.assets/image-20220815003958754.png)



Finish：

![image-20220815005659192](Jenkins.assets/image-20220815005659192.png)







此时发现多了一个target。里面就有我们打好的jar包：mydevops-0.0.1-SNAPSHOT.jar

```bash
jenkins@86216a6c0943:~/workspace/jenkinstest$ ls
pom.xml  src  target
jenkins@86216a6c0943:~/workspace/jenkinstest$ cd target
jenkins@86216a6c0943:~/workspace/jenkinstest/target$ ls
classes  generated-sources  generated-test-sources  maven-archiver  maven-status  mydevops-0.0.1-SNAPSHOT.jar  mydevops-0.0.1-SNAPSHOT.jar.original  test-classes
```











# 四、Jenkins推送jar包到目标服务器



## 1、Post build

![image-20220815010134077](Jenkins.assets/image-20220815010134077.png)



![image-20220815010217542](Jenkins.assets/image-20220815010217542.png)





![image-20220815010238347](Jenkins.assets/image-20220815010238347.png)





## 2、选择要传到目标服务器上的文件

![image-20220815010455876](Jenkins.assets/image-20220815010455876.png)



然后直接Apply and Save





3、build again



![image-20220815010649740](Jenkins.assets/image-20220815010649740.png)



这里出现错误：

```
Build step 'Send build artifacts over SSH' changed build result to UNSTABLE
```

![image-20220815010926815](Jenkins.assets/image-20220815010926815.png)



（1）开启verbose：

![image-20220815011313941](Jenkins.assets/image-20220815011313941.png)





（2）开启verbose之后，看到下面报错：

```bash
SSH: Connecting from host [86216a6c0943]
SSH: Connecting with configuration [jenkins] ...
SSH: Creating session: username [azureuser], hostname [20.127.132.51], port [2,222]
SSH: Connecting session ...
SSH: Connected
SSH: Opening SFTP channel ...
SSH: SFTP channel open
SSH: Connecting SFTP channel ...
SSH: Connected
SSH: cd [/home/azureuser/test]
SSH: OK
SSH: cd [/home/azureuser/test]
SSH: OK
SSH: mkdir [target]
SSH: FAILED: Message [Permission denied]
SSH: mkdir [target]
SSH: FAILED: Message [Permission denied]
SSH: Disconnecting configuration [jenkins] ...
ERROR: Exception when publishing, exception message [Could not create or change to directory. Directory [target]]
Build step 'Send build artifacts over SSH' changed build result to UNSTABLE
Finished: UNSTABLE
```



（3）尝试给test一个全权限：

```bash
root@jenkins:/home/azureuser/test# chmod 777 .
root@jenkins:/home/azureuser/test# ls
root@jenkins:/home/azureuser/test# cd ..
root@jenkins:/home/azureuser# ls
jdk8  maven  test
```





（4）build成功：

![image-20220815011601731](Jenkins.assets/image-20220815011601731.png)





```bash
SSH: Connecting from host [86216a6c0943]
SSH: Connecting with configuration [jenkins] ...
SSH: Creating session: username [azureuser], hostname [20.127.132.51], port [2,222]
SSH: Connecting session ...
SSH: Connected
SSH: Opening SFTP channel ...
SSH: SFTP channel open
SSH: Connecting SFTP channel ...
SSH: Connected
SSH: cd [/home/azureuser/test]
SSH: OK
SSH: cd [/home/azureuser/test]
SSH: OK
SSH: mkdir [target]
SSH: OK
SSH: cd [target]
SSH: OK
SSH: put [mydevops-0.0.1-SNAPSHOT.jar]
SSH: OK
SSH: Disconnecting configuration [jenkins] ...
SSH: Transferred 1 file(s)
Finished: SUCCESS
```



```bash
root@jenkins:/home/azureuser# cd test/
root@jenkins:/home/azureuser/test# ls
target
root@jenkins:/home/azureuser/test# cd target/
root@jenkins:/home/azureuser/test/target# ls
mydevops-0.0.1-SNAPSHOT.jar
```









# 五、在服务器用docker运行jar包



## 1、设置jar包名称

![image-20220815012939020](Jenkins.assets/image-20220815012939020.png)







## 2、create DockerFile

```dockerfile
FROM daocloud.io/library/java:8u40-jdk
COPY mydevops.jar /usr/local/
WORKDIR /usr/local
CMD java -jar mydevops.jar
```



![image-20220815013320056](Jenkins.assets/image-20220815013320056.png)







## 3、创建docker-compose.yml



```yaml
version: '3.1'
services:
  mydevops:
    build:
      context: ./
      dockerfile: DockerFile
    image: mydevops:v1.0
    container_name: mydevops
    ports:
      - 8081:8080
```





## 4、Commit and Push



![image-20220815014117895](Jenkins.assets/image-20220815014117895.png)





Push：

![image-20220815014324090](Jenkins.assets/image-20220815014324090.png)





![image-20220815014407846](Jenkins.assets/image-20220815014407846.png)

## 5、build again



![image-20220815014540681](Jenkins.assets/image-20220815014540681.png)





```bash
# 多了一个mydevops.jar
root@jenkins:/home/azureuser/test/target# ls
mydevops-0.0.1-SNAPSHOT.jar  mydevops.jar
```



```bash
root@jenkins:/home/azureuser/test/target# docker exec -it jenkins bash
jenkins@86216a6c0943:/$ cd ~
jenkins@86216a6c0943:~$ cd workspace/
jenkins@86216a6c0943:~/workspace$ ls
jenkinstest  jenkinstest@tmp
jenkins@86216a6c0943:~/workspace$ cd jenkinstest
jenkins@86216a6c0943:~/workspace/jenkinstest$ ls
Docker  pom.xml  src  target
jenkins@86216a6c0943:~/workspace/jenkinstest$ cd Docker/
jenkins@86216a6c0943:~/workspace/jenkinstest/Docker$ ls
DockerFile  docker-compose.yml
```







## 6、设置Post-build命令



![image-20220815015054943](Jenkins.assets/image-20220815015054943.png)



除了jar，也要上传Docker文件夹里的文件。然后执行命令。

```bash
cd /home/azureuser/test/Docker
mv ../target/*jar ./
sudo docker build -t wenyis/demo3 .
sudo docker run -d -p 3355:8080 --name idea wenyis/demo3
```

```bash
target/*.jar Docker/*
```





![image-20220815023040767](Jenkins.assets/image-20220815023040767.png)



Exec command: 

```bash
# 第一次cd最好用绝对路径。
# 这里用 ./test/Docker/ 也行。貌似是会到/home/azureuser目录下

cd /home/azureuser/test/Docker
mv ../target/*jar ./
sudo docker-compose down
sudo docker-compose up -d --build
sudo docker image prune -f # 删除多余镜像文件，注意：image没有s
```







Apply and Save







## 7、build 发布



![image-20220815015838187](Jenkins.assets/image-20220815015838187.png)





遇到错误：

```bash
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1&filters=%7B%22label%22%3A%7B%22com.docker.compose.project%3Ddocker%22%3Atrue%7D%7D&limit=0": dial unix /var/run/docker.sock: connect: permission denied
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1&filters=%7B%22label%22%3A%7B%22com.docker.compose.project%3Ddocker%22%3Atrue%7D%7D&limit=0": dial unix /var/run/docker.sock: connect: permission denied
SSH: EXEC: completed after 201 ms
```



![image-20220815020958203](Jenkins.assets/image-20220815020958203.png)





<font color="red">错误原因：应该是权限问题：只能用sudo运行docker-compose。</font>

解决方法1：

```bash
cd /home/azureuser/test/Docker
mv ../target/*jar ./
sudo docker-compose down
sudo docker-compose up -d --build
sudo docker image prune -f
```





解决方法2：让azureuser不用sudo也能运行docker命令

https://www.digitalocean.com/community/questions/how-to-fix-docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket





build成功：



![image-20220815022133438](Jenkins.assets/image-20220815022133438.png)







```bash
root@jenkins:/home/azureuser/test/Docker# docker images
REPOSITORY        TAG           IMAGE ID       CREATED         SIZE
mydevops          v1.0          117aa5ac1e73   2 minutes ago   833MB
jenkins/jenkins   2.346.2-lts   89b279054578   4 weeks ago     461MB
```



```bash
root@jenkins:/home/azureuser/test/Docker# docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS                                                                                      NAMES
68f1343e551e   mydevops:v1.0                 "/bin/sh -c 'java -j…"   2 minutes ago   Up 2 minutes   0.0.0.0:8081->8080/tcp, :::8081->8080/tcp                                                  mydevops
86216a6c0943   jenkins/jenkins:2.346.2-lts   "/usr/bin/tini -- /u…"   5 days ago      Up 4 hours     0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   jenkins
```



```bash
root@jenkins:/home/azureuser/test/Docker# curl http://localhost:8081/hello
Hello Jenkins!
```



![image-20220815022559636](Jenkins.assets/image-20220815022559636.png)











# 六、实现CD



## 1、配置Configure

在General里可以找到之前下载的插件 Git Parameter

![image-20220815225243071](Jenkins.assets/image-20220815225243071.png)









设置tag为parameter的名字：

![image-20220815225526196](Jenkins.assets/image-20220815225526196.png)









## 2、添加Shell命令





![image-20220815225557132](Jenkins.assets/image-20220815225557132.png)









## 3、Move Shell command before Maven

```bash
# 选择相应的版本
git checkout $tag
```



![image-20220815225853534](Jenkins.assets/image-20220815225853534.png)



Apply and Save





## 4、给Github的project打上标签



![image-20220815230009854](Jenkins.assets/image-20220815230009854.png)





Github要打tag首先需要有release。创建一个release，并打上tag v1.0

![image-20220815230220873](Jenkins.assets/image-20220815230220873.png)







## 5、去IDEA里修改代码



![image-20220815230734677](Jenkins.assets/image-20220815230734677.png)





![image-20220815230758659](Jenkins.assets/image-20220815230758659.png)









## 6、创建一个新的release



![image-20220815231237468](Jenkins.assets/image-20220815231237468.png)





## 7、回到Jenkins再次构建



这次首先发现之前的Build变成了Build with Parameter

![image-20220815231323762](Jenkins.assets/image-20220815231323762.png)





点进去之后，发现里面有我们的release，我可以选择相应的release来build

![image-20220815231421019](Jenkins.assets/image-20220815231421019.png)







## 8、测试



（1）首先，如果我build v1.0：

![image-20220815231631381](Jenkins.assets/image-20220815231631381.png)

（2）如果我build v1.1：

```bash
# v1.1的镜像会被下载
root@jenkins:/usr/local/docker/jenkins_docker# docker images
REPOSITORY        TAG           IMAGE ID       CREATED              SIZE 
mydevops          v1.1          5f3059951a29   3 seconds ago        833MB
mydevops          v1.0          7767693e4c08   About a minute ago   833MB
jenkins/jenkins   2.346.2-lts   89b279054578   4 weeks ago          461MB
```





![image-20220815231809628](Jenkins.assets/image-20220815231809628.png)
